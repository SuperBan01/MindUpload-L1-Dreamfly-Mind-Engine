<template>
  <view class="upload-container">
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
      <!-- 欢迎内容 -->
      <view class="welcome-content">
        <!-- 公司介绍 -->
        <view class="section intro" :class="{ 'appear': showIntro }">
          <text class="title">Everlasting AI</text>
          <text class="subtitle">生生科技</text>
          <text class="description">你好！我們是生生科技Everlasting AI，是当下这个宇宙中的第一家意识上傳公司。</text>
          <text class="description">在這裡，我們爲您提供一條管道，讓您暸解如何上传您的思想，並讓您在接下來的千百年中留下自己的数字备份。</text>
        </view>

        <!-- 意识上传范式 -->
        <view class="section paradigm" :class="{ 'appear': showParadigm }">
          <text class="description">基于意识上传第一范式，我們定義了4種阶段的思維上傳（具体内容参见公司官网
            <text class="website-link" @click="openWebsite">everlasting.zone</text>
          )。在云己，我们为您提供基於梦蝶心智模型的L I级意识上傳（基于个人心智数据训练个人意识备份）。</text>
        </view>

        <!-- 步骤1 -->
        <view class="section step" :class="{ 'appear': showStep1 }">
          <text class="step-title">#step1</text>
          <text class="description">要上傳您的思想，您需要在 upme 中上传您的思想文本数据。並且我們將在內測期期間提供免费的意识token，以驅動 Dreamfly Mind Model。您的心智数据将被壓統为10M大小的意识体文件（capy）。</text>
        </view>

        <!-- 步骤2 -->
        <view class="section step" :class="{ 'appear': showStep2 }">
          <text class="step-title">#step2</text>
          <text class="description">在最新的心智數據上传完毕後，您可以用梦蝶心智模型來驱动您自己的 Mindcopy。與自己的意识体的對話非常有趣，这可以給我們一個關於"我是誰"、"我如何成为当下自己"的哲學省思。</text>
          <text class="description">順便說一句，你可以通過TTFD （TuringTestForDigitallife）来评估意识体（upme)与自我意识的拟合度，这是我们与梦蝶实验室、南京大学联合推出的一种用于评估真实自我与数字自我拟合度的基准测试。</text>
        </view>

        <!-- 上传按钮 -->
        <view class="upload-section" :class="{ 'appear': showUpload }">
          <text class="start-text">現在，讓我們從 Upload Your MindData 開始吧！</text>
          <button class="upload-button" @click="startUpload">
            <text class="button-text">Upload Now</text>
            <text class="button-icon">↗</text>
          </button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      showIntro: false,
      showParadigm: false,
      showStep1: false,
      showStep2: false,
      showUpload: false
    }
  },

  onLoad() {
    this.startAnimation()
  },

  methods: {
    async startAnimation() {
      // 依次显示各个部分
      this.showIntro = true
      await this.wait(1000)
      this.showParadigm = true
      await this.wait(1000)
      this.showStep1 = true
      await this.wait(1000)
      this.showStep2 = true
      await this.wait(1000)
      this.showUpload = true
    },

    wait(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },

    startUpload() {
      // 跳转到上传页面
      uni.navigateTo({
        url: '/pages/upload/minddata'
      })
    },

    openWebsite() {
      // 打开官网
      uni.navigateTo({
        url: 'https://www.everlasting.zone/#/'
      })
    }
  }
}
</script>

<style lang="scss">
// 定义新的配色变量
$primary-purple: rgba(171, 130, 255, 0.85);     // 更亮的紫色
$primary-purple-glow: rgba(171, 130, 255, 0.2); // 紫色光晕
$text-white: rgba(255, 255, 255, 0.95);
$text-gray: rgba(255, 255, 255, 0.75);
$bg-dark: #121212;
$bg-gray: rgba(255, 255, 255, 0.08);

.upload-container {
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
    height: 100%;
    padding: 40px;
    overflow-y: auto;
    background: linear-gradient(to bottom, rgba(0,0,0,0.8), rgba(0,0,0,0.4));

    .welcome-content {
      max-width: 800px;
      margin: 0 auto;
      padding: 40px;

      .section {
        margin-bottom: 40px;
        opacity: 0;
        transform: translateY(20px);
        transition: all 0.8s ease;

        &.appear {
          opacity: 1;
          transform: translateY(0);
        }

        .title {
          font-size: 48px;
          font-weight: bold;
          color: $text-white;
          margin-bottom: 8px;
          display: block;
          text-shadow: 0 0 15px rgba(171, 130, 255, 0.4);
        }

        .subtitle {
          font-size: 24px;
          color: $text-gray;
          margin-bottom: 24px;
          display: block;
        }

        .step-title {
          font-size: 28px;
          color: $primary-purple;
          margin-bottom: 16px;
          display: block;
          text-shadow: 0 0 12px rgba(171, 130, 255, 0.4);
          font-weight: 500;
        }

        .description {
          font-size: 16px;
          line-height: 1.8;
          color: $text-white;
          margin-bottom: 16px;
          display: block;
          text-shadow: 0 0 20px rgba(0,0,0,0.8);
          
          .website-link {
            color: $primary-purple;
            text-decoration: underline;
            cursor: pointer;
            transition: all 0.3s ease;
            font-weight: 500;
            
            &:hover {
              opacity: 0.9;
              text-shadow: 0 0 12px rgba(171, 130, 255, 0.6);
            }
          }
        }
      }

      .upload-section {
        text-align: center;
        margin-top: 60px;
        opacity: 0;
        transform: translateY(20px);
        transition: all 0.8s ease;

        &.appear {
          opacity: 1;
          transform: translateY(0);
        }

        .start-text {
          font-size: 20px;
          color: $text-white;
          margin-bottom: 24px;
          display: block;
        }

        .upload-button {
          display: inline-flex;
          align-items: center;
          justify-content: center;
          padding: 16px 40px;
          background: $bg-gray;
          border: 1px solid $primary-purple;
          border-radius: 30px;
          cursor: pointer;
          transition: all 0.3s ease;
          backdrop-filter: blur(5px);

          &:hover {
            background: rgba(171, 130, 255, 0.15);
            box-shadow: 0 5px 25px rgba(171, 130, 255, 0.25);
            transform: translateY(-2px);
          }

          .button-text {
            font-size: 18px;
            color: $text-white;
            margin-right: 8px;
            font-weight: 500;
          }

          .button-icon {
            font-size: 20px;
            color: $primary-purple;
            text-shadow: 0 0 8px rgba(171, 130, 255, 0.6);
          }
        }
      }
    }
  }
}
</style> 