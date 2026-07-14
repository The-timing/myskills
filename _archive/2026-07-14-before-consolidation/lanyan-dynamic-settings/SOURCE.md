---
name: "lanyan-dynamic-settings"
description: "生成基于 LanYan 规范的动态分组系统配置代码。当用户需要实现带有动态表单（多种数据类型）、分组Tab展示以及客户端读取接口的通用配置管理功能时调用。"
---

# LanYan 动态系统配置规范 (Dynamic Settings)

该 Skill 用于在 LanYan 体系项目中快速构建和整改**动态系统配置管理**模块。

## 核心逻辑架构

该模块将配置分为**开发者模式（配置定义）**与**运营者模式（值维护）**，并提供**客户端便捷读取**的接口。

### 1. 数据库设计要素 (`bus_setting`)
表结构必须包含以下核心字段：
- **分组标识**：`group_key`, `group_name`，用于大类归属（如：系统配置、支付配置、UI配置）。
- **键值定义**：`value_key`, `value_name`, `value`。
- **展示控制**：`group_sort`, `value_sort`（排序用）。
- **组件类型**：`type`（0:输入框, 1:数字, 2:文件, 3:图片, 5:长文本, 6:富文本, 7:时间, 8:开关等）。
- **扩展约束**：`hint` (提示词), `selectValue` (可选值), `maxValue`, `minValue`, `stepValue`。

### 2. 前端管理端渲染逻辑 (Vue)
采用同一页面，通过权限或系统参数（如 `system.edit`）控制两种视图：
- **开发视图 (isShow=true)**：标准的列表 CRUD，供开发人员录入配置项、类型、约束条件等。
- **运营视图 (isShow=false)**：
  - 基于 `getAll` 接口获取数据。
  - 使用 `el-tabs` 按照 `groupName` 渲染分组标签页。
  - 内部遍历 `settingList`，通过 `v-if="item.type == X"` 动态渲染 `el-input`, `image-upload`, `editor`, `el-switch` 等不同形态的表单组件。
  - 底部提供统一的 `submitAll` 按钮进行批量保存。

### 3. 后端管理端接口 (`BusSettingController`)
除了基础的 CRUD（用于开发视图）外，必须提供：
- `getGroup()`: 聚合查询所有分组名及排序。
- `getAll()`: 获取所有配置，并按 `groupSort` 及 `groupKey` 进行 Map 分组，组装成 `{groupKey, groupName, settingList}` 格式的集合。
- `submitAll(@RequestBody List<SettingItemDto>)`: 接收分组数据，遍历 `settingList` 批量更新 `value` 字段。

### 4. 客户端读取接口 (`PublicController`)
向 APP/小程序等外部端提供 `/getSetting` 接口，公共读取能力优先使用统一封装的 `R<T>` 返回，根据传参不同返回不同粒度数据：
- **无参**：返回全部有效配置。
- **仅传 groupKey**：查询该组下的所有配置，并转换为 `Map<valueKey, value>` 格式返回，方便前端直接通过 key 访问。
- **传 groupKey + valueKey**：精确获取单一配置项的值。

## 触发场景
当用户提出以下需求时：
1. "为项目添加系统设置功能"
2. "实现一个动态表单配置页，支持图片、富文本和开关等多种类型"
3. "参考上一项目的系统配置重构当前配置模块"

## 代码模版 (Code Templates)

### 1. 数据库表结构 (`bus_setting.sql`)
```sql
CREATE TABLE `bus_setting` (
  `setting_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `group_key` varchar(100) DEFAULT NULL COMMENT '组属性',
  `group_name` varchar(100) DEFAULT NULL COMMENT '设置组',
  `value_key` varchar(100) DEFAULT NULL COMMENT '名属性',
  `value_name` varchar(100) DEFAULT NULL COMMENT '设置名',
  `value` text COMMENT '设置值',
  `group_sort` int(11) DEFAULT '0' COMMENT '组排序',
  `value_sort` int(11) DEFAULT '0' COMMENT '值排序',
  `type` char(1) DEFAULT '0' COMMENT '类型(0:输入框, 1:数字, 2:文件, 3:图片, 5:长文本, 6:富文本, 7:时间, 8:开关)',
  `hint` varchar(255) DEFAULT NULL COMMENT '提示词',
  `select_value` varchar(500) DEFAULT NULL COMMENT '可选值',
  `max_value` decimal(10,2) DEFAULT NULL COMMENT '最大值',
  `min_value` decimal(10,2) DEFAULT NULL COMMENT '最小值',
  `step_value` decimal(10,2) DEFAULT NULL COMMENT '步长',
  `del_flag` char(1) DEFAULT '0' COMMENT '删除标志',
  `create_by` varchar(64) DEFAULT '' COMMENT '创建者',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(64) DEFAULT '' COMMENT '更新者',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`setting_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统设置表';
```

### 2. 后端实体与 DTO

**实体类 (`BusSetting.java`)**
```java
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import java.math.BigDecimal;
import java.util.Date;

@Data
@TableName("bus_setting")
public class BusSetting {
    @TableId(type = IdType.AUTO)
    private Long settingId;
    private String groupKey;
    private String groupName;
    private String valueKey;
    private String valueName;
    private String value;
    private Integer groupSort;
    private Integer valueSort;
    private String type;
    private String hint;
    private String selectValue;
    private BigDecimal maxValue;
    private BigDecimal minValue;
    private BigDecimal stepValue;
    private String delFlag;
    private String createBy;
    private Date createTime;
    private String updateBy;
    private Date updateTime;
}
```

**数据传输对象 (`SettingItemDto.java`)**
```java
import lombok.Data;
import java.util.List;

@Data
public class SettingItemDto {
    private String groupKey;
    private String groupName;
    private List<BusSetting> settingList;
}
```

### 3. 后端管理接口核心逻辑 (`BusSettingController.java`)
用于支持运营视图的动态表单获取和批量更新。
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import java.util.*;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/system/busSetting")
public class BusSettingController extends BaseController {

    @Autowired
    private IBusSettingService busSettingService;

    // 聚合查询所有分组及排序，供开发视图下拉选择使用
    @GetMapping("getGroup")
    public AjaxResult getGroup() {
        QueryWrapper<BusSetting> wrapper = new QueryWrapper<>();
        wrapper.select("group_key", "group_name", "group_sort", "MAX(value_sort) + 1 AS value_sort")
                .eq("del_flag", 0)
                .groupBy("group_key", "group_name", "group_sort")
                .orderByAsc("MIN(group_sort)");
        return success(busSettingService.list(wrapper));
    }

    // 获取所有配置，并按 groupKey 分组，供运营视图动态 Tab 渲染使用
    @GetMapping("getAll")
    public AjaxResult getAll() {
        List<BusSetting> list = busSettingService.lambdaQuery()
                .eq(BusSetting::getDelFlag, "0")
                .orderByAsc(BusSetting::getGroupSort, BusSetting::getValueSort)
                .list();
        
        // 将 list 根据 groupKey 分组并按 groupSort 排序
        Map<String, List<BusSetting>> map = list.stream()
                .sorted(Comparator.comparing(BusSetting::getGroupSort))
                .collect(Collectors.groupingBy(
                        BusSetting::getGroupKey,
                        LinkedHashMap::new,
                        Collectors.toList()
                ));
                
        List<Map<String, Object>> res = new ArrayList<>();
        for (String key : map.keySet()) {
            Map<String, Object> item = new HashMap<>();
            List<BusSetting> settingList = map.get(key);
            item.put("groupKey", settingList.get(0).getGroupKey());
            item.put("groupName", settingList.get(0).getGroupName());
            item.put("settingList", settingList);
            res.add(item);
        }
        return success(res);
    }

    // 批量提交所有设置项的值
    @PostMapping("submitAll")
    public AjaxResult submitAll(@RequestBody List<SettingItemDto> list) {
        for (SettingItemDto itemDto : list) {
            List<BusSetting> settingList = itemDto.getSettingList();
            for (BusSetting item : settingList) {
                busSettingService.lambdaUpdate()
                        .eq(BusSetting::getSettingId, item.getSettingId())
                        .set(BusSetting::getValue, item.getValue())
                        .update();
            }
        }
        return success();
    }
}
```

### 4. 前端运营视图核心渲染 (`index.vue`)
包含了完整的动态 Tab 渲染逻辑及接口调用方式。
```vue
<template>
  <div class="app-container">
    <!-- 开发视图（isShow=true）: 顶部按钮 -->
    <el-row :gutter="10" class="mb8" v-if="isShow">
      <el-col :span="1.5">
        <el-button type="primary" plain icon="el-icon-plus" size="mini" @click="handleAdd">新增</el-button>
      </el-col>
    </el-row>

    <!-- 运营视图（isShow=false）: 动态Tab表单 -->
    <el-form ref="form" :model="form" v-if="!isShow" label-width="150px">
      <el-tabs v-model="activeName">
        <el-tab-pane v-for="(item,index) in allData" :key="index" :label="item.groupName" :name="item.groupKey">
          <el-form-item v-for="(item2,index2) in item.settingList" :key="item2.settingId" :label="item2.valueName">
            <el-tooltip effect="dark" :content="item2.hint" :disabled="!item2.hint" placement="top-start">
              <el-input v-if="item2.type == 0" v-model="item2.value" :placeholder="'请输入'+item2.valueName"/>
              <el-input-number v-else-if="item2.type == 1" v-model="item2.value" :max="item2.maxValue" :min="item2.minValue" :step="item2.stepValue"/>
              <file-upload v-else-if="item2.type == 2" v-model="item2.value" :limit="item2.maxValue"/>
              <image-upload v-else-if="item2.type == 3" v-model="item2.value" :limit="item2.maxValue"/>
              <el-input v-else-if="item2.type == 5" v-model="item2.value" type="textarea" autosize/>
              <editor v-else-if="item2.type == 6" v-model="item2.value" :min-height="192"/>
              <el-date-picker v-else-if="item2.type == 7" v-model="item2.value" type="datetime"/>
              <el-switch v-else-if="item2.type == 8" v-model="item2.value" active-value="1" inactive-value="0"/>
            </el-tooltip>
          </el-form-item>
        </el-tab-pane>
      </el-tabs>
    </el-form>
    
    <div v-if="!isShow" slot="footer" class="dialog-footer" style="margin-top: 20px;">
      <el-button type="primary" @click="submitAll">提 交</el-button>
    </div>

    <!-- 开发视图（isShow=true）: 标准列表 -->
    <el-table v-if="isShow" v-loading="loading" :data="settingList" border>
      <el-table-column label="ID" align="center" prop="settingId"/>
      <el-table-column label="设置组" align="center" prop="groupName" />
      <el-table-column label="设置名" align="center" prop="valueName" />
      <el-table-column label="设置值" align="center" prop="value" :show-overflow-tooltip="true"/>
      <el-table-column label="类型" align="center" prop="type">
        <template slot-scope="scope">
          {{ scope.row.type == '0' ? '输入框' : scope.row.type == '1' ? '数字输入框' : scope.row.type == '3' ? '图片上传' : scope.row.type == '8' ? '开关' : '--' }}
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)">修改</el-button>
          <el-button size="mini" type="text" icon="el-icon-delete" @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 开发视图（isShow=true）: 新增/修改弹窗 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px" v-if="isShow">
        <el-form-item label="组属性" prop="groupKey">
          <el-select
            v-model="form.groupKey"
            filterable
            allow-create
            default-first-option
            @change="checkGroup"
            placeholder="请选择文章标签">
            <el-option
              v-for="item in GroupList"
              :key="item.groupKey"
              :label="item.groupKey"
              :value="item.groupKey">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="设置组" prop="groupName" v-if="isShow">
          <el-input v-model="form.groupName" placeholder="请输入设置组"/>
        </el-form-item>
        <el-form-item label="名属性" prop="valueKey" v-if="isShow">
          <el-input v-model="form.valueKey" placeholder="请输入名属性"/>
        </el-form-item>
        <el-form-item label="设置名" prop="valueName" v-if="isShow">
          <el-input v-model="form.valueName" placeholder="请输入设置名"/>
        </el-form-item>
        <el-form-item label="设置值" prop="value">
          <el-input v-model="form.value" type="textarea" placeholder="请输入内容"/>
        </el-form-item>
        <el-form-item label="组排序" prop="groupSort" v-if="isShow">
          <el-input-number v-model="form.groupSort" :min="0" placeholder="请输入组排序" :step="1"/>
        </el-form-item>
        <el-form-item label="值排序" prop="valueSort" v-if="isShow">
          <el-input-number v-model="form.valueSort" :min="0" placeholder="请输入值排序" :step="1"/>
        </el-form-item>
        <el-form-item label="类型" prop="type" v-if="isShow">
          <el-select v-model="form.type" placeholder="请选择类型">
            <el-option key="0" label="输入框" value="0"/>
            <el-option key="1" label="数字输入框" value="1"/>
            <el-option key="2" label="文件上传" value="2"/>
            <el-option key="3" label="图片上传" value="3"/>
            <el-option key="4" label="选择器" value="4"/>
            <el-option key="5" label="长文本" value="5"/>
            <el-option key="6" label="富文本" value="6"/>
            <el-option key="7" label="时间选择器" value="7"/>
            <el-option key="8" label="开关" value="8"/>
          </el-select>
        </el-form-item>
        <el-form-item label="提示词" prop="hint" v-if="isShow">
          <el-input v-model="form.hint" type="textarea" placeholder="请输入内容"/>
        </el-form-item>
        <el-form-item label="可选值" prop="selectValue" v-if="isShow">
          <el-input v-model="form.selectValue" type="textarea" placeholder="请输入内容"/>
        </el-form-item>
        <el-form-item label="最大值" prop="maxValue" v-if="isShow&&(form.type == 1 || form.type == 2 || form.type == 3)">
          <el-input-number v-model="form.maxValue" placeholder="请输入最大值"/>
        </el-form-item>
        <el-form-item label="最小值" prop="minValue" v-if="isShow&&form.type == 1">
          <el-input-number v-model="form.minValue" placeholder="请输入最大值"/>
        </el-form-item>
        <el-form-item label="步数" prop="stepValue" v-if="isShow&&form.type == 1">
          <el-input-number v-model="form.stepValue" placeholder="请输入最大值"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { listBusSetting, getBusSetting, delBusSetting, addBusSetting, updateBusSetting, getGroup, getAll, submitAll } from "@/api/system/busSetting";

export default {
  name: "BusSetting",
  data() {
    return {
      loading: true,
      isShow: false, // 控制是否显示开发视图
      settingList: [], // 开发视图表格数据
      GroupList: [], // 下拉分组数据
      allData: [],   // 运营视图动态数据
      activeName: '',// 当前激活的 Tab groupKey
      open: false,
      title: "",
      form: {},
      queryParams: { pageNum: 1, pageSize: 10 }
    };
  },
  created() {
    // 假设通过系统参数判断是否开启开发模式（也可通过角色权限判断）
    this.getConfigKey("system.edit").then(res => {
      this.isShow = (res.msg == 'true');
      if (this.isShow) {
        this.getList(); // 加载开发视图的标准表格数据
      } else {
        this.getAllData(); // 加载运营视图动态数据
      }
    });
  },
  methods: {
    // --- 运营视图方法 ---
    getAllData() {
      getAll().then(res => {
        this.allData = res.data;
        if (res.data.length != 0) {
          this.activeName = res.data[0].groupKey; // 默认激活第一个Tab
        }
      });
    },
    submitAll() {
      submitAll(this.allData).then(res => {
        this.$modal.msgSuccess("修改成功");
      });
    },

    // --- 开发视图方法 ---
    getList() {
      this.loading = true;
      listBusSetting(this.queryParams).then(response => {
        this.settingList = response.rows;
        this.loading = false;
      });
    },
    getGroupList() {
      getGroup().then(res => {
        this.GroupList = res.data;
      });
    },
    checkGroup(e) {
      this.GroupList.forEach(item => {
        if (item.groupKey == e) {
          this.form.groupName = item.groupName;
          this.form.groupSort = item.groupSort;
          if (!this.form.settingId) this.form.valueSort = item.valueSort;
        }
      });
    },
    handleAdd() {
      this.form = { type: '0' };
      this.getGroupList();
      this.open = true;
      this.title = "添加设置";
    },
    handleUpdate(row) {
      this.getGroupList();
      getBusSetting(row.settingId).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改设置";
      });
    },
    submitForm() {
      if (this.form.settingId != null) {
        updateBusSetting(this.form).then(response => {
          this.$modal.msgSuccess("修改成功");
          this.open = false;
          this.getList();
        });
      } else {
        addBusSetting(this.form).then(response => {
          this.$modal.msgSuccess("新增成功");
          this.open = false;
          this.getList();
        });
      }
    },
    handleDelete(row) {
      this.$modal.confirm('是否确认删除设置编号为"' + row.settingId + '"的数据项？').then(() => {
        return delBusSetting(row.settingId);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      });
    }
  }
};
</script>
```

### 5. 客户端读取接口 (`PublicController.java`)
```java
@GetMapping("/getSetting")
@ApiOperation(value = "获取设置")
public R<Object> getSetting(String groupKey, String valueKey) {
    if (StringUtils.isEmpty(groupKey)) {
        return R.ok(settingService.lambdaQuery().eq(BusSetting::getDelFlag, "0").list());
    } else if (StringUtils.isEmpty(valueKey)) {
        List<BusSetting> list = settingService.lambdaQuery()
                .eq(BusSetting::getDelFlag, "0")
                .eq(BusSetting::getGroupKey, groupKey)
                .orderByAsc(BusSetting::getValueSort)
                .list();
        Map<String, String> res = new HashMap<>();
        for (BusSetting setting : list) {
            res.put(setting.getValueKey(), setting.getValue());
        }
        return R.ok(res);
    } else {
        BusSetting setting = settingService.lambdaQuery()
                .eq(BusSetting::getDelFlag, "0")
                .eq(BusSetting::getGroupKey, groupKey)
                .eq(BusSetting::getValueKey, valueKey)
                .one();
        return R.ok(setting != null ? setting.getValue() : null, "获取成功");
    }
}
```
