<template>
  <view class="tab-bar">
    <view 
      class="tab-item" 
      :class="{ active: currentTab === 'past' }"
      @tap="switchTab('past')"
    >
      <text class="tab-text">往</text>
    </view>
    <view 
      class="tab-item" 
      :class="{ active: currentTab === 'now' }"
      @tap="switchTab('now')"
    >
      <text class="tab-text">今</text>
    </view>
    <view 
      class="tab-item" 
      :class="{ active: currentTab === 'future' }"
      @tap="switchTab('future')"
    >
      <text class="tab-text">未</text>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['update:modelValue'])
const props = defineProps({
  modelValue: {
    type: String,
    default: 'now'
  }
})

const currentTab = ref(props.modelValue)

const switchTab = (tab) => {
  currentTab.value = tab
  emit('update:modelValue', tab)
  uni.switchTab({
    url: '/pages/' + tab + '/index'
  })
}
</script>

<style lang="scss">
.tab-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100rpx;
  background: #fff;
  display: flex;
  justify-content: space-around;
  align-items: center;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.05);
  
  .tab-item {
    flex: 1;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: relative;
    
    &::after {
      content: '';
      position: absolute;
      bottom: 0;
      width: 0;
      height: 2px;
      background: #000;
      transition: width 0.3s ease;
    }
    
    &.active {
      &::after {
        width: 20px;
      }
      
      .tab-text {
        color: #000;
        font-weight: 500;
      }
    }
    
    .tab-text {
      font-size: 16px;
      color: #666;
      transition: all 0.3s ease;
    }
  }
}
</style> 