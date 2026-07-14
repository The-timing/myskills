---
name: upload-to-oss
description: 将文件上传方式从本地存储切换为阿里云 OSS。包含后端依赖添加、配置修改、工具类创建以及前端上传组件和头像上传逻辑的修改。
---

<skill_instructions>
当用户请求将文件上传方式更改为阿里云 OSS 时，请按照以下步骤操作：

## 1. 后端依赖添加 (lanyan-common)
在 `lanyan-common/pom.xml` 中确认是否有添加阿里云 OSS 依赖，没有则添加：
```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.10.2</version>
</dependency>
```

## 2. 后端配置添加 (lanyan-admin)
在 `lanyan-admin/src/main/resources/application.yml` 中添加 OSS 配置：
```yaml
aliyunoss:
  # 地域节点
  endpoint: xxxxxxx
  # AccessKey
  accessKeyId: xxxxxxx
  # AccessKey 秘钥
  accessKeySecret: xxxxxxx
  # bucket名称
  bucketName: xxxxxx
  # bucket下文件夹的路径
  filehost: img
  # 访问域名
  url: https://xxxxxxxxxxxxxxx
```

## 3. 创建 OSS 工具类 (lanyan-common)
创建文件 `lanyan-common/src/main/java/com/lanyan/common/utils/oss/AliyunOssUploadUtils.java`：
```java
package com.lanyan.common.utils.oss;

import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import com.lanyan.common.config.AliyunOssConfig;
import com.lanyan.common.utils.file.FileUploadUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;
import java.io.IOException;

@Component
public class AliyunOssUploadUtils {

    private static AliyunOssConfig aliyunOssConfig;

    @Autowired
    public AliyunOssUploadUtils(AliyunOssConfig aliyunOssConfig) {
        AliyunOssUploadUtils.aliyunOssConfig = aliyunOssConfig;
    }

    public static String uploadFile(MultipartFile file) throws Exception {
        OSS ossClient = new OSSClientBuilder().build(aliyunOssConfig.getEndpoint(), aliyunOssConfig.getAccessKeyId(), aliyunOssConfig.getAccessKeySecret());
        String filePathName = FileUploadUtils.extractFilename(file);
        filePathName = aliyunOssConfig.getFilehost() + "/" + filePathName;
        try {
            ossClient.putObject(aliyunOssConfig.getBucketName(), filePathName, file.getInputStream());
        } catch (IOException e) {
            e.printStackTrace();
            throw e;
        } finally {
            if (ossClient != null) {
                ossClient.shutdown();
            }
        }
        return aliyunOssConfig.getUrl() + "/" + filePathName;
    }
}
```

## 4. 创建 OSS 配置类 (lanyan-common)
创建文件 `lanyan-common/src/main/java/com/lanyan/common/config/AliyunOssConfig.java`：
```java
package com.lanyan.common.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Data
@Component
@ConfigurationProperties(prefix = "aliyunoss")
public class AliyunOssConfig {
    private String endpoint;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;
    private String filehost;
    private String url;
}
```

## 5. 修改 CommonController (lanyan-admin)
修改 `lanyan-admin/src/main/java/com/lanyan/web/controller/common/CommonController.java`，更新 `upload` 和 `uploadFiles` 方法以使用 `AliyunOssUploadUtils.uploadFile(file)`。
注意：
1. 引入 `com.lanyan.common.utils.oss.AliyunOssUploadUtils`。
2. `upload` 方法返回的 `url` 应为 OSS 完整路径。
3. `uploadFiles` 方法返回的 `urls` 列表应为 OSS 完整路径列表。

## 6. 前端 ImageUpload 修改 (lanyan-ui)
修改 `lanyan-ui/src/components/ImageUpload/index.vue`：
1. `uploadImgUrl` 修改为 `process.env.VUE_APP_BASE_API + "/common/upload"` (如果之前不是)。
2. 在 `handleUploadSuccess` 或类似回调中，确保 `item` 对象的 `url` 直接使用返回的 OSS URL，而不是拼接 `baseUrl`。
   ```javascript
   // 示例修改
   item = { name: item, url: item }; // 之前可能是 this.baseUrl + item
   ```

## 7. 前端 FileUpload 修改 (lanyan-ui)
修改 `lanyan-ui/src/components/FileUpload/index.vue`：
1. `uploadFileUrl` 确认指向 `/common/upload`。
2. `handleUploadSuccess` 中，直接使用 `res.url`，不再拼接 baseUrl。this.uploadList.push({ name: res.fileName, url: res.url });

## 8. 修改 SysProfileController (lanyan-admin)
修改 `lanyan-admin/src/main/java/com/lanyan/web/controller/system/SysProfileController.java` 中的 `avatar` 方法，使用 `AliyunOssUploadUtils.uploadFile(file)` 并返回完整 URL。

## 9. 前端 UserAvatar 修改 (lanyan-ui)
修改 `lanyan-ui/src/views/system/user/profile/userAvatar.vue`：
1. `uploadImg` 方法中，成功回调直接使用 `response.imgUrl` (或 `url`)，不再拼接 `process.env.VUE_APP_BASE_API`。
2. 添加 `setAvatarBase64` 和 `transBase64FromImage` 方法以支持跨域图片裁剪预览。
3. `editCropper` 方法调用 `setAvatarBase64`。

## 10. 修改 Store (lanyan-ui)
修改 `lanyan-ui/src/store/modules/user.js`：
在获取 `avatar` 的逻辑中，如果 `user.avatar` 是完整 URL (OSS)，则不再拼接 `process.env.VUE_APP_BASE_API`。

## 11. 提示用户
提醒用户登录阿里云 OSS 控制台，在权限管理下的跨域设置中配置允许跨域。
</skill_instructions>
