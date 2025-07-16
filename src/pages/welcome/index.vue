<template>
  <view class="welcome-container">
    <video
      class="background-video"
      src="/static/layer.mp4"
      loop
      muted
      autoplay
      :controls="false"
      object-fit="cover"
    ></video>
    
    <view class="content">
      <view class="header">
        <text class="title">Upload Your</text>
        <text class="title">Consciousness</text>
        <text class="title">to the Cloud</text>
        <text class="subtitle">创建你的数字意识</text>
        <text class="subtitle">永恒存在</text>
      </view>
      
      <view class="description">
        <text class="desc-text">基于梦蝶心智模型</text>
        <text class="desc-text">上传你的思想、声音和形象数据</text>
        <text class="desc-text">生成独特的数字意识体</text>
        <text class="desc-text">让你的存在突破时间与空间的限制</text>
      </view>

      <view class="action-buttons">
        <button class="start-btn" @click="handleStartClick">开始上传</button>
        <button class="explore-btn" @click="goToExplore">探索意识体</button>
      </view>
    </view>

    <view class="footer">
      <text class="footer-text">Powered by Dreamfly Mind Model</text>
    </view>

    <view class="overlay"></view>
  </view>
</template>

<script setup>
import { onMounted } from 'vue'

const handleStartClick = () => {
  uni.navigateTo({
    url: '/pages/upload/index'
  })
}

const goToExplore = () => {
  uni.navigateTo({
    url: '/pages/welcome/explore'
  })
}

onMounted(() => {
  console.log('Welcome page mounted')
})
</script>

<style lang="scss">
@import '@/styles/variables.scss';

.welcome-container {
  min-height: 100vh;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  box-sizing: border-box;
  overflow: hidden;

  .background-video {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    z-index: 1;
  }

  .overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
    z-index: 2;
  }

  .content {
    position: relative;
    z-index: 3;
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 40rpx;
    
    .header {
      text-align: center;
      margin-bottom: 80rpx;

      .title {
        font-size: 64rpx;
        color: #ffffff;
        font-weight: 200;
        line-height: 1.2;
        letter-spacing: 2px;
        display: block;
        text-transform: uppercase;
      }

      .subtitle {
        font-size: 36rpx;
        color: rgba(255, 255, 255, 0.8);
        font-weight: 300;
        line-height: 1.6;
        display: block;
        margin-top: 40rpx;
      }
    }

    .description {
      margin-bottom: 80rpx;
      text-align: center;

      .desc-text {
        font-size: 28rpx;
        color: rgba(255, 255, 255, 0.6);
        line-height: 2;
        display: block;
        font-weight: 300;
        letter-spacing: 1px;
      }
    }

    .action-buttons {
      width: 100%;
      display: flex;
      flex-direction: column;
      gap: 30rpx;
      padding: 0 40rpx;

      .start-btn, .explore-btn {
        height: 96rpx;
        border-radius: 48rpx;
        font-size: 32rpx;
        font-weight: 300;
        display: flex;
        align-items: center;
        justify-content: center;
        background: transparent;
        border: 1px solid rgba(255, 255, 255, 0.3);
        color: #ffffff;
        letter-spacing: 2px;
        transition: all 0.3s ease;

        &:active {
          background: rgba(255, 255, 255, 0.1);
          border-color: rgba(255, 255, 255, 0.5);
        }
      }
    }
  }

  .footer {
    position: relative;
    z-index: 3;
    text-align: center;
    padding: 40rpx 0;

    .footer-text {
      font-size: 24rpx;
      color: rgba(255, 255, 255, 0.4);
      letter-spacing: 1px;
      font-weight: 300;
    }
  }
}

/* 响应式样式 */
@media screen and (min-width: 768px) {
  .welcome-container {
    .content {
      padding: 60rpx;

      .header {
        .title {
          font-size: 88rpx;
        }

        .subtitle {
          font-size: 44rpx;
        }
      }

      .description {
        max-width: 800rpx;
        margin-left: auto;
        margin-right: auto;

        .desc-text {
          font-size: 32rpx;
        }
      }

      .action-buttons {
        max-width: 600rpx;
        margin-left: auto;
        margin-right: auto;
      }
    }
  }
}
</style> 