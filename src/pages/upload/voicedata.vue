<template>
  <view class="voicedata-container">
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
      <scroll-view 
        class="scroll-container" 
        scroll-y="true"
        :scroll-with-animation="true"
      >
        <view class="voice-content">
          <!-- 标题区域 -->
          <view class="header">
            <text class="page-title">音色上传</text>
            <text class="page-subtitle">声音是灵魂的载体，让意识体拥有您的声音</text>
          </view>

          <!-- 录音指引 -->
          <view class="guide-section">
            <view class="guide-item">
              <text class="guide-title">时长要求</text>
              <text class="guide-content">1分钟</text>
            </view>

            <view class="guide-item">
              <text class="guide-title">朗读文本</text>
              <view class="reading-text">
                <text class="text-content">从前，庄周梦见自己变成了蝴蝶，一只翩翩起舞的蝴蝶。他十分惬意舒畅，悠然自得，根本不知道自己原本是庄周。突然梦醒，他才惊觉自己分明是庄周。可他却疑惑起来，不知是庄周做梦变成了蝴蝶呢，还是蝴蝶做梦变成了庄周？</text>
              </view>
            </view>

            <view class="guide-item">
              <text class="guide-title">录制要求</text>
              <text class="guide-content">• 使用高质量的录音设备</text>
              <text class="guide-content">• 确保声音清晰、无杂音</text>
              <text class="guide-content">• 在安静的环境中进行录制</text>
            </view>

            <view class="guide-item">
              <text class="guide-title">支持格式</text>
              <text class="guide-content">mp3 / wav</text>
            </view>
          </view>

          <!-- 录音控制区 -->
          <view class="record-section">
            <view class="record-status" v-if="isRecording">
              <text class="timer">{{ formatTime(recordingTime) }}</text>
              <view class="wave-animation">
                <view class="wave" v-for="i in 5" :key="i"></view>
              </view>
            </view>

            <view class="record-controls">
              <button 
                class="control-btn" 
                :class="{ 'recording': isRecording }"
                @click="toggleRecording"
              >
                <text class="btn-icon">{{ isRecording ? '■' : '●' }}</text>
                <text class="btn-text">{{ isRecording ? '停止录制' : '开始录制' }}</text>
              </button>

              <!-- 试听控制按钮 -->
              <button 
                class="control-btn preview"
                v-if="!isRecording && recordingFile && !isPlaying"
                @click="togglePreview"
              >
                <text class="btn-icon">{{ isPlaying ? '■' : '▶' }}</text>
                <text class="btn-text">{{ isPlaying ? '停止试听' : '试听录音' }}</text>
              </button>

              <button 
                class="control-btn upload"
                v-if="!isRecording && recordingFile && !isPlaying"
                @click="uploadRecording"
              >
                <text class="btn-icon">↑</text>
                <text class="btn-text">上传录音</text>
              </button>
            </view>

            <!-- 音频播放器 -->
            <view class="audio-player" v-if="isPlaying">
              <view class="progress-bar">
                <view 
                  class="progress" 
                  :style="{ width: audioProgress + '%' }"
                ></view>
              </view>
              <text class="time-info">{{ formatTime(currentTime) }} / {{ formatTime(duration) }}</text>
            </view>

            <input 
              type="file" 
              accept=".mp3,.wav" 
              class="file-input"
              @change="handleFileUpload"
              ref="fileInput"
            />
            
            <button 
              class="upload-btn"
              @click="openFileSelector"
            >
              <text class="btn-text">或者上传已有录音文件</text>
            </button>
          </view>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      isRecording: false,
      recordingTime: 0,
      recordingFile: null,
      timer: null,
      isPlaying: false,
      audioPlayer: null,
      audioProgress: 0,
      currentTime: 0,
      duration: 0,
      audioUrl: null,
      recorder: null,
      chunks: []
    }
  },

  methods: {
    toggleRecording() {
      if (this.isRecording) {
        this.stopRecording()
      } else {
        this.startRecording()
      }
    },

    async startRecording() {
      try {
        // 清除之前的录音
        if (this.audioUrl) {
          URL.revokeObjectURL(this.audioUrl)
          this.audioUrl = null
        }
        this.recordingFile = null
        this.isPlaying = false
        if (this.audioPlayer) {
          this.audioPlayer.pause()
          this.audioPlayer = null
        }
        
        // 请求麦克风权限
        const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
        this.recorder = new MediaRecorder(stream)
        this.chunks = []
        
        this.recorder.ondataavailable = (e) => {
          this.chunks.push(e.data)
        }
        
        this.recorder.onstop = () => {
          const blob = new Blob(this.chunks, { type: 'audio/wav' })
          this.recordingFile = new File([blob], 'recording.wav', { type: 'audio/wav' })
          this.audioUrl = URL.createObjectURL(this.recordingFile)
          
          // 关闭麦克风
          stream.getTracks().forEach(track => track.stop())
        }
        
        this.recorder.start()
        this.isRecording = true
        this.recordingTime = 0
        this.timer = setInterval(() => {
          this.recordingTime++
          if (this.recordingTime >= 60) {
            this.stopRecording()
          }
        }, 1000)

        uni.showToast({
          title: '开始录音',
          icon: 'none'
        })
      } catch (err) {
        uni.showToast({
          title: '无法访问麦克风',
          icon: 'none'
        })
        console.error('录音错误:', err)
      }
    },

    stopRecording() {
      if (this.recorder && this.recorder.state === 'recording') {
        this.recorder.stop()
      }
      this.isRecording = false
      clearInterval(this.timer)
      
      uni.showToast({
        title: '录音已保存',
        icon: 'success'
      })
    },

    togglePreview() {
      if (this.isPlaying) {
        this.stopPreview()
      } else {
        this.startPreview()
      }
    },

    startPreview() {
      if (!this.audioPlayer) {
        this.audioPlayer = new Audio(this.audioUrl)
        this.audioPlayer.addEventListener('timeupdate', this.updateProgress)
        this.audioPlayer.addEventListener('loadedmetadata', () => {
          this.duration = this.audioPlayer.duration
        })
        this.audioPlayer.addEventListener('ended', this.stopPreview)
      }
      this.audioPlayer.play()
      this.isPlaying = true
    },

    stopPreview() {
      if (this.audioPlayer) {
        this.audioPlayer.pause()
        this.audioPlayer.currentTime = 0
      }
      this.isPlaying = false
      this.audioProgress = 0
      this.currentTime = 0
    },

    updateProgress() {
      if (this.audioPlayer) {
        this.currentTime = this.audioPlayer.currentTime
        this.audioProgress = (this.audioPlayer.currentTime / this.audioPlayer.duration) * 100
      }
    },

    formatTime(seconds) {
      const mins = Math.floor(seconds / 60)
      const secs = seconds % 60
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
    },

    openFileSelector() {
      this.$refs.fileInput.click()
    },

    handleFileUpload(event) {
      const file = event.target.files[0]
      if (file) {
        if (file.size > 10 * 1024 * 1024) { // 10MB限制
          uni.showToast({
            title: '文件大小不能超过10MB',
            icon: 'none'
          })
          return
        }
        // 清除之前的录音
        if (this.audioUrl) {
          URL.revokeObjectURL(this.audioUrl)
        }
        this.recordingFile = file
        this.audioUrl = URL.createObjectURL(file)
        this.stopPreview()
        uni.showToast({
          title: '文件已选择',
          icon: 'success'
        })
      }
    },

    async uploadRecording() {
      if (!this.recordingFile) {
        uni.showToast({
          title: '请先录制或上传音频',
          icon: 'none'
        })
        return
      }

      uni.showLoading({
        title: '上传中...'
      })

      try {
        // 确保globalData存在
        if (!getApp().globalData) {
          getApp().globalData = {}
        }
        if (!getApp().globalData.uploadData) {
          getApp().globalData.uploadData = {}
        }

        // 保存音频文件到全局状态
        getApp().globalData.uploadData.voiceFile = this.recordingFile
        
        uni.hideLoading()
        uni.showToast({
          title: '上传成功',
          icon: 'success'
        })
        
        // 跳转到下一步
        setTimeout(() => {
          uni.navigateTo({
            url: '/pages/upload/imagedata'
          })
        }, 1500)
      } catch (err) {
        uni.hideLoading()
        uni.showToast({
          title: '上传失败',
          icon: 'none'
        })
        console.error('上传错误:', err)
      }
    },

    beforeDestroy() {
      // 清理资源
      if (this.audioUrl) {
        URL.revokeObjectURL(this.audioUrl)
      }
      if (this.audioPlayer) {
        this.audioPlayer.removeEventListener('timeupdate', this.updateProgress)
        this.audioPlayer = null
      }
      if (this.recorder) {
        this.recorder.stream?.getTracks().forEach(track => track.stop())
      }
    }
  }
}
</script>

<style lang="scss">
// 继承之前的配色方案
$primary-purple: rgba(171, 130, 255, 0.85);
$primary-purple-glow: rgba(171, 130, 255, 0.2);
$text-white: rgba(255, 255, 255, 0.95);
$text-gray: rgba(255, 255, 255, 0.75);
$bg-dark: #121212;
$bg-gray: rgba(255, 255, 255, 0.08);

.voicedata-container {
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

    .scroll-container {
      height: 100%;
      padding: 40px;
    }

    .voice-content {
      max-width: 800px;
      margin: 0 auto;
      padding-bottom: 40px;
      min-height: 100%;
      display: flex;
      flex-direction: column;
      gap: 40px;

      .header {
        text-align: center;
        padding-bottom: 20px;

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

      .guide-section {
        display: flex;
        flex-direction: column;
        gap: 30px;
        padding: 30px;
        background: rgba(171, 130, 255, 0.05);
        border-radius: 16px;
        backdrop-filter: blur(10px);

        .guide-item {
          .guide-title {
            font-size: 18px;
            color: $primary-purple;
            margin-bottom: 12px;
            display: block;
            font-weight: 500;
          }

          .guide-content {
            font-size: 16px;
            color: $text-white;
            line-height: 1.6;
            display: block;
          }

          .reading-text {
            padding: 20px;
            background: rgba(255, 255, 255, 0.05);
            border-radius: 12px;
            border-left: 4px solid $primary-purple;

            .text-content {
              font-size: 16px;
              color: $text-white;
              line-height: 1.8;
              letter-spacing: 0.5px;
            }
          }
        }
      }

      .record-section {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 20px;

        .record-status {
          display: flex;
          flex-direction: column;
          align-items: center;
          gap: 10px;

          .timer {
            font-size: 24px;
            color: $primary-purple;
            font-weight: 500;
          }

          .wave-animation {
            display: flex;
            gap: 4px;
            align-items: center;
            height: 40px;

            .wave {
              width: 4px;
              height: 20px;
              background: $primary-purple;
              border-radius: 2px;
              animation: wave 1s ease-in-out infinite;

              @for $i from 1 through 5 {
                &:nth-child(#{$i}) {
                  animation-delay: $i * 0.1s;
                }
              }
            }
          }
        }

        .record-controls {
          display: flex;
          gap: 20px;
          flex-wrap: wrap;
          justify-content: center;

          .control-btn {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            padding: 12px 24px;
            background: $bg-gray;
            border: 1px solid $primary-purple;
            border-radius: 24px;
            cursor: pointer;
            transition: all 0.3s ease;
            gap: 8px;

            &:hover {
              background: rgba(171, 130, 255, 0.15);
              box-shadow: 0 5px 25px rgba(171, 130, 255, 0.25);
              transform: translateY(-2px);
            }

            &.recording {
              background: $primary-purple;
              animation: pulse 2s infinite;

              .btn-icon,
              .btn-text {
                color: $text-white;
              }
            }

            &.preview {
              background: rgba(171, 130, 255, 0.2);
              
              .btn-icon {
                color: $primary-purple;
              }
              
              .btn-text {
                color: $text-white;
              }

              &:hover {
                background: rgba(171, 130, 255, 0.3);
              }
            }

            .btn-icon {
              font-size: 18px;
              color: $primary-purple;
            }

            .btn-text {
              font-size: 16px;
              color: $text-white;
            }

            &.upload {
              background: $primary-purple;

              .btn-icon,
              .btn-text {
                color: $text-white;
              }
            }
          }
        }

        .audio-player {
          width: 100%;
          max-width: 400px;
          padding: 20px;
          background: rgba(171, 130, 255, 0.1);
          border-radius: 12px;
          margin-top: 20px;

          .progress-bar {
            width: 100%;
            height: 4px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 2px;
            overflow: hidden;
            margin-bottom: 8px;

            .progress {
              height: 100%;
              background: $primary-purple;
              transition: width 0.1s linear;
            }
          }

          .time-info {
            font-size: 14px;
            color: $text-gray;
            text-align: center;
          }
        }

        .file-input {
          display: none;
        }

        .upload-btn {
          background: none;
          border: none;
          padding: 12px 24px;
          cursor: pointer;

          .btn-text {
            font-size: 14px;
            color: $text-gray;
            text-decoration: underline;
          }

          &:hover .btn-text {
            color: $primary-purple;
          }
        }
      }
    }
  }
}

@keyframes wave {
  0%, 100% {
    height: 20px;
  }
  50% {
    height: 40px;
  }
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(171, 130, 255, 0.4);
  }
  70% {
    box-shadow: 0 0 0 10px rgba(171, 130, 255, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(171, 130, 255, 0);
  }
}
</style> 