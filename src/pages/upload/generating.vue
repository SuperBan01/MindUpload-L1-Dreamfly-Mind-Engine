<template>
  <view class="generating-container">
    <!-- 背景视频 -->
    <video
      class="background-video"
      src="/static/layer.mp4"
      loop
      muted
      autoplay
      :controls="false"
      object-fit="cover"
    ></video>

    <!-- 内容遮罩 -->
    <view class="content-overlay">
      <view class="generating-content">
        <!-- 标题 -->
        <view class="header">
          <text class="page-title">意识体生成中</text>
          <text class="page-subtitle">正在整合您的数据，请稍候...</text>
        </view>

        <!-- 加载动画 -->
        <view class="loading-section">
          <view class="loading-circle">
            <view class="circle-inner"></view>
          </view>
          <text class="loading-text">{{ loadingText }}</text>
          <text class="progress-text">{{ progress }}%</text>
        </view>

        <!-- 生成步骤 -->
        <view class="steps-section">
          <view 
            v-for="(step, index) in steps" 
            :key="index"
            class="step-item"
            :class="{ active: currentStep >= index }"
          >
            <text class="step-icon">{{ step.icon }}</text>
            <text class="step-text">{{ step.text }}</text>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      progress: 0,
      currentStep: 0,
      loadingText: '正在处理您的形象数据...',
      steps: [
        { icon: '🎨', text: '处理形象数据' },
        { icon: '🎵', text: '处理声音数据' },
        { icon: '🧠', text: '处理思想数据' },
        { icon: '✨', text: '生成意识体' }
      ]
    }
  },

  onLoad() {
    this.startGeneration()
  },

  methods: {
    startGeneration() {
      let step = 0
      const totalSteps = this.steps.length
      const stepDuration = 2000 // 每个步骤2秒

      const updateProgress = () => {
        if (step < totalSteps) {
          this.currentStep = step
          this.loadingText = this.steps[step].text
          this.progress = Math.floor((step / totalSteps) * 100)
          
          step++
          setTimeout(updateProgress, stepDuration)
        } else {
          // 生成完成，跳转到完成页面
          setTimeout(() => {
            uni.navigateTo({
              url: '/pages/upload/complete'
            })
          }, 1000)
        }
      }

      updateProgress()
    }
  }
}
</script>

<style lang="scss">
$primary-purple: rgba(171, 130, 255, 0.85);
$primary-purple-glow: rgba(171, 130, 255, 0.2);
$text-white: rgba(255, 255, 255, 0.95);
$text-gray: rgba(255, 255, 255, 0.75);
$bg-dark: #121212;
$bg-gray: rgba(255, 255, 255, 0.08);

.generating-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: $bg-dark;

  .background-video {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    z-index: 1;
    opacity: 0.6;
  }

  .content-overlay {
    position: relative;
    z-index: 2;
    height: 100vh;
    background: linear-gradient(to bottom, rgba(0,0,0,0.8), rgba(0,0,0,0.4));
    display: flex;
    align-items: center;
    justify-content: center;

    .generating-content {
      max-width: 800px;
      width: 100%;
      padding: 40px;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 40px;

      .header {
        text-align: center;

        .page-title {
          font-size: 36px;
          font-weight: bold;
          color: $text-white;
          margin-bottom: 8px;
          display: block;
          text-shadow: 0 0 15px rgba(171, 130, 255, 0.4);
        }

        .page-subtitle {
          font-size: 18px;
          color: $primary-purple;
          display: block;
        }
      }

      .loading-section {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 20px;

        .loading-circle {
          width: 120px;
          height: 120px;
          border-radius: 50%;
          background: rgba(171, 130, 255, 0.1);
          display: flex;
          align-items: center;
          justify-content: center;
          position: relative;
          animation: rotate 2s linear infinite;

          .circle-inner {
            width: 100px;
            height: 100px;
            border-radius: 50%;
            background: $primary-purple;
            box-shadow: 0 0 30px rgba(171, 130, 255, 0.4);
          }
        }

        .loading-text {
          font-size: 18px;
          color: $text-white;
          text-align: center;
        }

        .progress-text {
          font-size: 24px;
          color: $primary-purple;
          font-weight: bold;
        }
      }

      .steps-section {
        display: flex;
        flex-direction: column;
        gap: 16px;
        width: 100%;
        max-width: 400px;

        .step-item {
          display: flex;
          align-items: center;
          gap: 12px;
          padding: 12px 20px;
          background: rgba(255, 255, 255, 0.05);
          border-radius: 12px;
          transition: all 0.3s ease;

          &.active {
            background: rgba(171, 130, 255, 0.1);
            border-left: 4px solid $primary-purple;

            .step-icon {
              color: $primary-purple;
            }

            .step-text {
              color: $text-white;
            }
          }

          .step-icon {
            font-size: 24px;
            color: $text-gray;
          }

          .step-text {
            font-size: 16px;
            color: $text-gray;
          }
        }
      }
    }
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style> 