<template>
  <view class="plate-field">
    <view class="plate-input-wrap" @click="showPlateKeyboard()">
      <block v-for="(item, index) in plateSlots" :key="index">
        <text v-if="index === 1" class="plate-dot"></text>
        <view
          class="plate-box"
          :class="{ active: index === currentPlateIndex && plateKeyboardVisible }"
          @click.stop="showPlateKeyboard(index)"
        >
          <text>{{ item }}</text>
        </view>
      </block>
    </view>

    <view v-if="plateKeyboardVisible" class="plate-keyboard-mask" @click="hidePlateKeyboard"></view>
    <view v-if="plateKeyboardVisible" class="plate-keyboard" @click.stop>
      <view class="keyboard-header">
        <text>{{ licenseNumber || '请输入车牌号' }}</text>
        <text class="keyboard-done" @click="hidePlateKeyboard">完成</text>
      </view>
      <view v-for="(row, rowIndex) in currentKeyboardRows" :key="rowIndex" class="keyboard-row">
        <view
          v-for="key in row"
          :key="key"
          class="keyboard-key"
          :class="{ action: isActionKey(key), disabled: isPlateKeyDisabled(key) }"
          @click="handlePlateKey(key)"
        >
          <text>{{ key }}</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      plateKeyboardVisible: false,
      plateKeyboardMode: 'province',
      activePlateIndex: 0,
      provinceRows: [
        ['京', '津', '渝', '沪', '冀', '晋', '辽', '吉', '黑', '苏'],
        ['浙', '皖', '闽', '赣', '鲁', '豫', '鄂', '湘', '粤', '琼'],
        ['川', '贵', '云', '陕', '甘', '青', '蒙', '桂', '宁', '新'],
        ['ABC', '藏', '使', '领', '警', '学', '港', '澳', '删']
      ],
      letterRows: [
        ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0'],
        ['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
        ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L'],
        ['中文', 'Z', 'X', 'C', 'V', 'B', 'N', 'M', '删']
      ]
    }
  },
  computed: {
    licenseNumber() {
      return (this.value || '').trim().toUpperCase()
    },
    plateSlots() {
      const chars = this.licenseNumber.split('')
      const slots = []
      for (let index = 0; index < 8; index += 1) {
        slots.push(chars[index] || '')
      }
      return slots
    },
    currentPlateIndex() {
      return Math.min(this.activePlateIndex, 7)
    },
    currentKeyboardRows() {
      return this.plateKeyboardMode === 'province' ? this.provinceRows : this.letterRows
    }
  },
  methods: {
    showPlateKeyboard(index) {
      const endIndex = Math.min(this.licenseNumber.length, 7)
      this.activePlateIndex = typeof index === 'number' ? Math.min(index, endIndex) : endIndex
      this.plateKeyboardMode = this.activePlateIndex === 0 ? 'province' : 'letter'
      this.plateKeyboardVisible = true
    },
    hidePlateKeyboard() {
      this.plateKeyboardVisible = false
    },
    isActionKey(key) {
      return key === 'ABC' || key === '中文' || key === '删'
    },
    isPlateKeyDisabled(key) {
      return this.currentPlateIndex === 0 && this.plateKeyboardMode === 'letter' && key !== '中文' && key !== '删'
    },
    handlePlateKey(key) {
      if (key === 'ABC') {
        this.plateKeyboardMode = 'letter'
        return
      }
      if (key === '中文') {
        this.plateKeyboardMode = 'province'
        return
      }
      if (key === '删') {
        this.deletePlateChar()
        return
      }
      if (!this.isPlateKeyDisabled(key)) {
        this.setPlateChar(key)
      }
    },
    setPlateChar(key) {
      const chars = this.licenseNumber.split('')
      const index = Math.min(this.currentPlateIndex, chars.length, 7)
      chars[index] = key
      this.$emit('input', chars.join('').trim().toUpperCase().slice(0, 8))
      this.activePlateIndex = Math.min(index + 1, 7)
      this.plateKeyboardMode = this.activePlateIndex === 0 ? 'province' : 'letter'
    },
    deletePlateChar() {
      const chars = this.licenseNumber.split('')
      if (!chars.length) {
        this.activePlateIndex = 0
        this.plateKeyboardMode = 'province'
        return
      }
      const deleteIndex = this.activePlateIndex >= chars.length ? chars.length - 1 : this.activePlateIndex
      chars.splice(deleteIndex, 1)
      this.$emit('input', chars.join(''))
      this.activePlateIndex = Math.max(deleteIndex, 0)
      this.plateKeyboardMode = this.activePlateIndex === 0 ? 'province' : 'letter'
    }
  }
}
</script>

<style lang="scss" scoped>
.plate-input-wrap {
  display: flex;
  align-items: center;
  gap: 12rpx;
}

.plate-box {
  width: 60rpx;
  height: 60rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1rpx solid #ddd;
  border-radius: 4rpx;
  background: #fff;
}

.plate-box.active {
  border-color: #2979ff;
}

.plate-dot {
  width: 10rpx;
  height: 10rpx;
  border-radius: 50%;
  background: #2979ff;
}

.plate-keyboard-mask {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  z-index: 800;
}

.plate-keyboard {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 801;
  padding: 12rpx;
  background: #d7dbe2;
}

.keyboard-header, .keyboard-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10rpx;
}

.keyboard-row {
  justify-content: center;
  gap: 7rpx;
}

.keyboard-key {
  min-width: 62rpx;
  height: 72rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8rpx;
  background: #fff;
}

.keyboard-key.action { background: #c4c9d2; }
.keyboard-key.disabled { opacity: .4; }
.keyboard-done { color: #2979ff; }
</style>
