<template>
  <view class="terminal-container">
    <!-- 系统信息头部 -->
    <view class="terminal-header">
      <text>MindOS Terminal v1.0.0</text>
      <text>{{ currentTime }}</text>
    </view>
    
    <!-- 主体内容区域 -->
    <view class="main-content">
      <!-- 左侧终端区域 -->
      <scroll-view class="terminal-content" scroll-y :scroll-top="scrollTop" ref="terminalContent">
        <!-- ASCII 艺术图标 -->
        <view class="ascii-art" v-if="showAsciiArt">
          <text v-for="(line, index) in asciiArt" :key="index">{{ line }}</text>
          <view class="welcome-guide">
            <text class="guide-line">/* 欢迎使用 MindOS 意识体终端系统 */</text>
            <text class="guide-line">/* 快速入门: */</text>
            <text class="guide-line">/* 1. 输入 'help' 查看所有可用命令 */</text>
            <text class="guide-line">/* 2. 输入 'load jobs.mind' 加载史蒂夫·乔布斯意识体 */</text>
            <text class="guide-line">/* 3. 加载完成后直接输入文字即可与意识体对话 */</text>
          </view>
          <view v-if="userName" class="welcome-guide">
            <text>欢迎，{{ userName }}！你可以输入 <b>load {{ userName }}.mind</b> 加载你的数字意识体。</text>
          </view>
        </view>

        <!-- 系统消息和对话记录 -->
        <view class="message-container">
          <view v-for="(message, index) in messages" :key="index" 
            :class="['message', message.type]">
            <text v-if="message.type === 'system'" class="timestamp">{{ message.timestamp }}</text>
            <text v-if="message.type === 'chat'" 
              :class="['chat-prefix', message.role]">{{ message.role === 'user' ? '> You:' : '> Mind:' }}</text>
            <text class="content">{{ message.content }}</text>
          </view>
        </view>
      </scroll-view>

      <!-- 右侧状态栏 -->
      <view v-if="showStatus" class="status-panel">
        <view class="status-header">
          <text class="status-title">====== MIND STATUS ======</text>
        </view>
        <view class="status-content">
          <view v-for="(value, key) in mindStatus" :key="key" class="status-item">
            <text class="status-key">{{ key }}</text>
            <text class="status-value" :class="{ 'thinking': key === '系统状态' && isThinking }">
              {{ key === '系统状态' && isThinking ? '思考中...' : value }}
            </text>
          </view>
        </view>
        <view class="status-footer">
          <text class="status-indicator" :class="{ 'active': true, 'thinking': isThinking }">
            {{ isThinking ? '● THINKING' : '● ONLINE' }}
          </text>
        </view>
      </view>
    </view>
    
    <!-- 命令行输入 -->
    <view class="input-area">
      <text class="prompt">{{ currentPrompt }}</text>
      <input 
        type="text"
        v-model="currentInput"
        @confirm="handleCommand"
        :disabled="isProcessing"
        placeholder="Enter command or chat with consciousness..."
        class="command-input"
      />
    </view>
  </view>
</template>

<script>
// 导入预设的.mind文件
import jobsMind from './jobs.mind'
import aristotleMind from './aristotle.mind'
import confuciusMind from './confucius.mind'

const ASCII_ART = [
  "    __  ___           ________    __      __                 ___           ",
  "   /  |/  /___ _____ / / ____/   / /___  / /___ _____  ____/ (_)___  ____ ",
  "  / /|_/ / __ \`/ __ \`/ / / __   / / __ \\/ / __ \`/ __ \\/ __  / / __ \\/ __ \\",
  " / /  / / /_/ / /_/ / / /_/ /  / / /_/ / / /_/ / / / / /_/ / / / / / /_/ /",
  "/_/  /_/\\__,_/\\__, /_/\\____/  /_/\\____/_/\\__,_/_/ /_/\\__,_/_/_/ /_/\\____/ ",
  "             /____/                                                         "
];

export default {
  data() {
    return {
      mindFiles: {
        'jobs.mind': jobsMind,
        'aristotle.mind': aristotleMind,
        'confucius.mind': confuciusMind
      },
      currentTime: '',
      systemStarted: false,
      mindLoaded: false,
      mindFile: '',
      scrollTop: 0,
      command: '',
      systemLogs: [],
      mindStatus: {},
      timer: null,
      asciiArt: ASCII_ART,
      showAsciiArt: true,
      messages: [],
      currentInput: '',
      currentPrompt: '>',
      isProcessing: false,
      showStatus: false,
      apiKey: '',  //填入你的apikey，优先选择硅基流动
      chatContext: [],
      mindPrompt: '',
      tokenCount: 0,
      isThinking: false,
      uploadedMind: null,
      audioContext: null,
      oscillator: null,
      gainNode: null,
      tickTimeout: null,
      currentEmotion: 'neutral',
      emotionSoundConfig: {
        happy: {
          type: 'sawtooth',
          frequency: 800,
          duration: 50,
          interval: 100,
          volume: 0.03,
          tickInterval: [50, 100]
        },
        neutral: {
          type: 'sawtooth',
          frequency: 600,
          duration: 100,
          interval: 150,
          volume: 0.02,
          tickInterval: [100, 200]
        },
        angry: {
          type: 'sawtooth',
          frequency: 700,
          duration: 150,
          interval: 200,
          volume: 0.04,
          tickInterval: [80, 150]
        },
        sad: {
          type: 'sawtooth',
          frequency: 500,
          duration: 200,
          interval: 300,
          volume: 0.015,
          tickInterval: [300, 500]
        }
      },
      remoteMindFiles: [],
      userName: ''
    }
  },
  
  onLoad(options) {
    this.mindFile = options.file || 'unknown.mind'
    this.initTerminal()
    this.updateTime()
    this.timer = setInterval(this.updateTime, 1000)
    
    // 如果有传入文件参数，自动加载该文件
    if (options.file) {
      this.loadMindFile(options.file)
    }
  },
  
  mounted() {
    // 初始化音频
    this.initAudio()
    this.fetchRemoteMindFiles()
    // 获取当前登录用户名
    const userInfo = getApp().globalData.userInfo
    if (userInfo && userInfo.username) {
      this.userName = userInfo.username
    }
  },
  
  onUnload() {
    if (this.timer) {
      clearInterval(this.timer)
    }
  },
  
  methods: {
    updateTime() {
      const now = new Date()
      this.currentTime = now.toLocaleTimeString()
    },
    
    initTerminal() {
      this.systemStarted = true
      this.logMessage('System initialized successfully', 'success')
      this.logMessage('Type "help" for available commands', 'info')
    },
    
    loadMindStatus(file) {
      this.mindFile = file
      this.mindLoaded = true
      
      // 模拟从 jobs.mind 加载数据
      if (file === 'jobs.mind') {
        this.mindStatus = {
          'Name': 'Steve Jobs',
          'Birth': '1955-02-24',
          'Death': '2011-10-05',
          'Occupation': 'Entrepreneur, Industrial designer, Investor, Media proprietor',
          'Companies': 'Apple Inc., NeXT, Pixar',
          'Notable Products': 'Macintosh, iPod, iPhone, iPad',
          'Consciousness Level': '98.5%',
          'Memory Fragments': '1,234,567',
          'Personality Match': '99.2%',
          'Active Neural Connections': '8.9B'
        }
        this.logMessage('Mind file loaded successfully', 'success')
      }
    },
    
    logMessage(message, type = 'info') {
      this.systemLogs.push({ message, type })
      this.$nextTick(() => {
        this.scrollTop = 9999
      })
    },
    
    async handleCommand() {
      if (this.isProcessing) return
      
      const command = this.currentInput.trim().toLowerCase()
      this.messages.push({
        type: 'system',
        content: `> ${this.currentInput}`,
        timestamp: new Date().toLocaleTimeString()
      })
      this.currentInput = ''
      
      if (!command) return
      
      this.isProcessing = true
      
      switch(command) {
        case 'help':
          await this.showHelp()
          break
          
        case 'clear':
          this.clearTerminal()
          break
          
        case 'status':
          if (this.showStatus) {
            await this.typeMessage('意识体状态已加载', 'system')
          } else {
            await this.typeMessage('请先加载意识体文件', 'error')
          }
          break
          
        default:
          if (command.startsWith('load ')) {
            await this.loadMindFile(command.split(' ')[1])
          } else if (this.showStatus) {
            // 如果意识体已加载,将输入视为对话内容
            await this.chatWithMind(command)
          } else {
            await this.typeMessage('未知命令。输入 "help" 获取帮助。', 'error')
          }
      }
      
      this.isProcessing = false
      this.scrollToBottom()
    },
    
    // 计算生命天数
    calculateLifeDays(birthDate) {
      const now = new Date()
      let birth
      
      // 处理BCE日期
      if (birthDate.includes('BCE')) {
        const year = parseInt(birthDate.replace(' BCE', ''))
        birth = new Date(0)
        birth.setFullYear(-year)
      } else {
        birth = new Date(birthDate)
      }
      
      // 计算日期差
      const diffTime = Math.abs(now - birth)
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
      return diffDays.toString()
    },

    async loadMindFile(file) {
      await this.typeMessage(`正在加载意识体文件: ${file}...`, 'system')
      await this.sleep(1000)
      
      try {
        let mindData
        
        // 首先检查是否是用户上传的文件
        const globalData = getApp().globalData || {}
        if (globalData.uploadedMind && globalData.uploadedMind.name === file) {
          console.log('从全局状态加载上传的文件')
          try {
            // 解析上传的文件内容
            const content = globalData.uploadedMind.content
            if (content.includes('export default')) {
              const jsonStr = content.replace(/export\s+default\s+/, '').trim()
              mindData = JSON.parse(jsonStr)
            } else {
              mindData = JSON.parse(content)
            }
          } catch (error) {
            console.error('解析上传文件失败:', error)
            throw new Error('上传文件格式无效')
          }
        }
        
        // 如果不是上传的文件，尝试从预加载文件中获取
        if (!mindData && this.mindFiles[file]) {
          console.log('从预加载文件中加载')
          mindData = this.mindFiles[file]
        }
        
        // 新增：如果file等于当前用户名.mind，优先后端拉取
        if (!mindData && this.userName && file === `${this.userName}.mind`) {
          const res = await uni.request({
            url: `http://localhost:3001/api/mind/${file}`,
            method: 'GET'
          })
          if (res.data.code === 200) {
            let content = res.data.data
            if (content.includes('export default')) {
              content = content.replace(/export\\s+default\\s+/, '').trim()
            }
            mindData = JSON.parse(content)
          } else {
            throw new Error('后端未找到你的mind文件')
          }
        }
        
        // 新增：从后端拉取
        if (!mindData && this.remoteMindFiles.includes(file)) {
          const res = await uni.request({
            url: `http://localhost:3001/api/mind/${file}`,
            method: 'GET'
          })
          if (res.data.code === 200) {
            let content = res.data.data
            if (content.includes('export default')) {
              content = content.replace(/export\s+default\s+/, '').trim()
            }
            mindData = JSON.parse(content)
          } else {
            throw new Error('后端未找到该mind文件')
          }
        }
        
        if (!mindData) {
          throw new Error(`未找到文件: ${file}`)
        }
        
        // 验证文件格式
        if (!this.validateMindData(mindData)) {
          throw new Error('意识体文件格式无效')
        }
        
        // 模拟加载过程
        const loadingSteps = [
          '初始化神经网络...',
          '加载记忆碎片...',
          '同步思维模型...',
          '校准情感参数...',
          '建立意识连接...'
        ]
        
        for (const step of loadingSteps) {
          await this.typeMessage(step, 'system')
          await this.sleep(800)
        }
        
        // 计算生命天数
        const lifeDays = this.calculateLifeDays(mindData.metadata.birth)
        
        // 设置意识体状态
        this.mindStatus = {
          '意识体': mindData.metadata.name,
          '意识等级': `${(mindData.consciousness.anti_program_ratio * 100).toFixed(1)}%`,
          '记忆完整度': `${(mindData.consciousness.connection_degree * 100).toFixed(1)}%`,
          '生命天数': lifeDays,
          '情感模块': '已同步',
          '神经连接': '8.9B活跃',
          '系统状态': '在线',
          '意识Token': '0'
        }
        
        // 设置ASCII艺术图标
        this.asciiArt = mindData.metadata.ascii_art
        
        // 设置意识体的人设prompt
        this.mindPrompt = mindData.metadata.personality_prompt
        
        // 保存加载的意识体数据
        this.uploadedMind = mindData

        this.showStatus = true
        await this.typeMessage('意识体加载完成', 'success')
        this.currentPrompt = `${mindData.metadata.name} >`
        
      } catch (error) {
        console.error('加载文件失败:', error)
        await this.typeMessage(`加载失败: ${error.message}`, 'error')
        return
      }
    },
    
    validateMindData(data) {
      if (!data || typeof data !== 'object') {
        return false
      }
      
      const required = ['metadata', 'memory', 'status', 'consciousness']
      const missing = required.filter(key => !data[key])
      
      if (missing.length > 0) {
        console.error('缺少必要字段:', missing)
        return false
      }
      
      if (!data.metadata.name || !data.metadata.personality_prompt) {
        console.error('缺少必要的元数据字段')
        return false
      }
      
      return true
    },
    
    async chatWithMind(message) {
      try {
        this.isThinking = true
        this.currentEmotion = 'neutral'  // 开始思考时设置为平静状态
        this.startThinkingSound()
        
        // 添加用户消息
        this.messages.push({
          type: 'chat',
          role: 'user',
          content: message
        })

        // 准备API请求
        const response = await fetch('https://api.siliconflow.cn/v1/chat/completions', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${this.apiKey}`
          },
          body: JSON.stringify({
            model: 'deepseek-ai/DeepSeek-V3',
            messages: [
              { role: 'system', content: this.mindPrompt },
              ...this.chatContext,
              { role: 'user', content: message }
            ],
            temperature: 0.7,
            max_tokens: 1024,
            stream: true
          })
        })

        // 处理流式响应
        const reader = response.body.getReader()
        let aiResponse = ''
        let currentTokens = 0
        
        // 添加助手消息
        this.messages.push({
          type: 'chat',
          role: 'assistant',
          content: ''
        })
        
        while (true) {
          const { done, value } = await reader.read()
          if (done) break
          
          const chunk = new TextDecoder().decode(value)
          const lines = chunk.split('\n')
          
          for (const line of lines) {
            if (line.startsWith('data: ')) {
              try {
                const jsonStr = line.slice(5).trim()
                if (jsonStr === '[DONE]') continue
                
                const data = JSON.parse(jsonStr)
                if (data.choices?.[0]?.delta?.content) {
                  aiResponse += data.choices[0].delta.content
                  currentTokens++
                  this.tokenCount += 1
                  this.mindStatus['意识Token'] = this.tokenCount.toString()
                  
                  // 更新最后一条消息
                  this.messages[this.messages.length - 1].content = aiResponse
                  
                  // 每次收到新内容时分析情感并更新声音效果
                  if (currentTokens % 10 === 0) {
                    const emotion = this.analyzeEmotion(aiResponse)
                    if (emotion !== this.currentEmotion) {
                      this.currentEmotion = emotion
                      // 重新启动声音效果
                      this.stopThinkingSound()
                      this.startThinkingSound()
                    }
                  }
                  
                  this.scrollToBottom()
                }
              } catch (e) {
                console.log('解析响应出错:', e)
                continue
              }
            }
          }
        }

        // 更新对话上下文
        this.chatContext.push(
          { role: 'user', content: message },
          { role: 'assistant', content: aiResponse }
        )
        
      } catch (error) {
        console.error('API调用出错:', error)
        await this.typeMessage('与意识体通信出错: ' + error.message, 'error')
      } finally {
        this.isThinking = false
        this.stopThinkingSound()
      }
    },
    
    async showHelp() {
      const helpMessages = [
        '可用命令:',
        'help    - 显示此帮助信息',
        'load    - 加载意识体文件 (例如: load jobs.mind)',
        'status  - 显示意识体状态',
        'clear   - 清空终端',
        '直接输入文本与意识体对话'
      ]
      
      for (const msg of helpMessages) {
        await this.typeMessage(msg, 'system')
      }
      // 新增：专属引导
      if (this.userName) {
        await this.typeMessage(`你已登录，可以输入 load ${this.userName}.mind 加载你的数字意识体`, 'system')
      }
      if (this.remoteMindFiles.length) {
        await this.typeMessage('你还可以加载以下用户mind文件:', 'system')
        for (const f of this.remoteMindFiles) {
          await this.typeMessage(`load ${f}`, 'system')
        }
      }
    },
    
    clearTerminal() {
      this.messages = []
      this.scrollTop = 0
    },
    
    async typeMessage(content, type = 'system') {
      this.messages.push({
        type,
        content,
        timestamp: new Date().toLocaleTimeString()
      })
      await this.sleep(100)
      this.scrollToBottom()
    },
    
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },
    
    scrollToBottom() {
      this.$nextTick(() => {
        const scrollView = this.$refs.terminalContent
        if (scrollView) {
          this.scrollTop = 999999
        }
      })
    },
    
    initAudio() {
      // 创建音频上下文
      this.audioContext = new (window.AudioContext || window.webkitAudioContext)()
      
      // 创建增益节点（控制音量）
      this.gainNode = this.audioContext.createGain()
      this.gainNode.gain.value = 0.1 // 设置音量
      this.gainNode.connect(this.audioContext.destination)
    },
    
    startThinkingSound() {
      if (!this.audioContext) return
      
      const config = this.emotionSoundConfig[this.currentEmotion || 'neutral']
      
      // 创建主音频节点
      this.oscillator = this.audioContext.createOscillator()
      this.oscillator.type = config.type
      
      // 创建音调包络
      const envelope = this.audioContext.createGain()
      envelope.gain.value = 0
      
      // 创建低通滤波器使声音更厚实
      const filter = this.audioContext.createBiquadFilter()
      filter.type = 'lowpass'
      filter.frequency.value = 2000  // 设置截止频率
      filter.Q.value = 1.0          // 设置共振
      
      // 连接节点
      this.oscillator.connect(filter)
      filter.connect(envelope)
      envelope.connect(this.gainNode)
      
      // 启动振荡器
      this.oscillator.start()
      
      // 设置音量
      this.gainNode.gain.value = config.volume
      
      // 创建循环播放的声效
      const playTick = () => {
        // 设置频率，使用更小的随机范围
        this.oscillator.frequency.setValueAtTime(
          config.frequency + Math.random() * 100 - 50,
          this.audioContext.currentTime
        )
        
        // 音量包络
        envelope.gain.cancelScheduledValues(this.audioContext.currentTime)
        envelope.gain.setValueAtTime(0, this.audioContext.currentTime)
        envelope.gain.linearRampToValueAtTime(1, this.audioContext.currentTime + 0.001)
        envelope.gain.linearRampToValueAtTime(0, this.audioContext.currentTime + config.duration / 1000)
        
        // 随机间隔触发下一个音效
        const [minInterval, maxInterval] = config.tickInterval
        const nextInterval = Math.random() * (maxInterval - minInterval) + minInterval
        this.tickTimeout = setTimeout(playTick, nextInterval)
      }
      
      playTick()
    },
    
    stopThinkingSound() {
      if (this.oscillator) {
        clearTimeout(this.tickTimeout)
        this.oscillator.stop()
        this.oscillator.disconnect()
        this.oscillator = null
      }
    },

    // 分析文本情感
    analyzeEmotion(text) {
      // 情感关键词列表
      const emotionKeywords = {
        happy: [
          '很高兴', '太棒了', 'excellent', 'amazing', 'great', '创新', '激动',
          '完美', 'incredible', '突破', '成功', '优秀', '好', '喜欢', '赞',
          '热爱', '期待', '满意', '开心', '棒', '厉害', '精彩', '优雅', '笑',
          '有趣', '快乐', '兴奋', '惊喜'
        ],
        angry: [
          '生气', 'angry', '愤怒', '不行', '差劲', 'terrible', '讨厌', '烦',
          '可恶', '糟糕透了', '荒谬', '不可接受', '过分', '混蛋', '废话',
          '别开玩笑了', '简直', '太差了'
        ],
        sad: [
          '遗憾', '悲伤', '难过', '痛苦', '失望', '伤心', '可惜', '哎',
          '无奈', '叹息', '心痛', '苦恼', '忧伤', '惆怅', '思念', '想念'
        ]
      }

      // 计算情感得分
      let scores = {
        happy: 0,
        angry: 0,
        sad: 0
      }

      // 转换为小写进行匹配
      const lowerText = text.toLowerCase()

      // 统计各情感词出现次数
      Object.entries(emotionKeywords).forEach(([emotion, keywords]) => {
        keywords.forEach(keyword => {
          const regex = new RegExp(keyword.toLowerCase(), 'g')
          const matches = lowerText.match(regex)
          if (matches) {
            scores[emotion] += matches.length
          }
        })
      })

      // 判断情感倾向
      const maxScore = Math.max(...Object.values(scores))
      if (maxScore === 0) return 'neutral'
      
      return Object.entries(scores).find(([_, score]) => score === maxScore)[0]
    },

    // 播放情感音效
    playEmotionSound(emotion) {
      if (!this.audioContext) return
      
      const config = this.emotionSoundConfig[emotion]
      
      // 创建音频节点
      const oscillator = this.audioContext.createOscillator()
      const gainNode = this.audioContext.createGain()
      
      // 设置音频参数
      oscillator.type = config.type
      oscillator.frequency.setValueAtTime(config.frequency, this.audioContext.currentTime)
      gainNode.gain.setValueAtTime(0, this.audioContext.currentTime)
      
      // 连接节点
      oscillator.connect(gainNode)
      gainNode.connect(this.audioContext.destination)
      
      // 设置音量包络
      gainNode.gain.linearRampToValueAtTime(config.volume, this.audioContext.currentTime + 0.01)
      gainNode.gain.linearRampToValueAtTime(0, this.audioContext.currentTime + config.duration / 1000)
      
      // 启动和停止
      oscillator.start(this.audioContext.currentTime)
      oscillator.stop(this.audioContext.currentTime + config.duration / 1000)
      
      // 清理
      setTimeout(() => {
        oscillator.disconnect()
        gainNode.disconnect()
      }, config.duration + 100)
    },

    async fetchRemoteMindFiles() {
      try {
        const res = await uni.request({
          url: 'http://localhost:3001/api/mind-list',
          method: 'GET'
        })
        if (res.data.code === 200) {
          this.remoteMindFiles = res.data.data
        }
      } catch (e) {
        console.error('获取远程mind文件列表失败', e)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.terminal-container {
  height: 100vh;
  background-color: #000;
  color: #fff;
  font-family: 'Courier New', Courier, monospace;
  display: flex;
  flex-direction: column;
  padding: 20px;
}

.terminal-header {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 1px solid #333;
  margin-bottom: 20px;
}

.main-content {
  flex: 1;
  display: flex;
  gap: 20px;
  overflow: hidden;
}

.terminal-content {
  flex: 1;
  overflow-y: auto;
  padding-right: 20px;
  
  .ascii-art {
    margin-bottom: 30px;
    
    text {
      display: block;
      color: #0f0;
      white-space: pre;
      line-height: 1.2;
      font-size: 12px;
    }

    .welcome-guide {
      margin-top: 20px;
      padding: 15px;
      background: rgba(0, 255, 0, 0.05);
      border-left: 2px solid #0f0;
      
      .guide-line {
        display: block;
        color: #0f0;
        font-size: 14px;
        line-height: 1.6;
        opacity: 0.8;
        
        &:hover {
          opacity: 1;
        }
      }
    }
  }
  
  .message-container {
    .message {
      margin-bottom: 8px;
      
      &.system {
        color: #0f0;
        
        .timestamp {
          color: #666;
          margin-right: 8px;
        }
      }
      
      &.error {
        color: #f00;
      }
      
      &.success {
        color: #0f0;
      }
      
      &.chat {
        margin-bottom: 12px;
        
        .chat-prefix {
          color: #666;
          margin-right: 8px;
          
          &.user {
            color: #0ff;
          }
          
          &.assistant {
            color: #0f0;
          }
        }
        
        .content {
          color: #fff;
        }
      }
    }
  }
}

.status-panel {
  width: 300px;
  background: rgba(0, 255, 0, 0.05);
  border-left: 1px solid #333;
  display: flex;
  flex-direction: column;
  
  .status-header {
    padding: 15px;
    text-align: center;
    border-bottom: 1px solid #333;
    
    .status-title {
      color: #0f0;
      font-size: 14px;
      font-weight: bold;
      text-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
    }
  }
  
  .status-content {
    flex: 1;
    padding: 15px;
    overflow-y: auto;
    
    .status-item {
      margin-bottom: 15px;
      
      .status-key {
        display: block;
        color: #0f0;
        font-size: 12px;
        margin-bottom: 5px;
        opacity: 0.8;
      }
      
      .status-value {
        display: block;
        color: #fff;
        font-size: 14px;
        padding: 5px 10px;
        background: rgba(0, 255, 0, 0.1);
        border-radius: 4px;
        
        &.thinking {
          background: rgba(255, 165, 0, 0.2);
          color: #ffa500;
          animation: thinking 2s infinite;
        }
      }
    }
  }
  
  .status-footer {
    padding: 15px;
    text-align: center;
    border-top: 1px solid #333;
    
    .status-indicator {
      color: #666;
      font-size: 12px;
      
      &.active {
        color: #0f0;
        animation: pulse 2s infinite;
      }
      
      &.thinking {
        color: #ffa500;
        animation: thinking 2s infinite;
      }
    }
  }
}

.input-area {
  display: flex;
  align-items: center;
  padding: 10px 0;
  border-top: 1px solid #333;
  margin-top: 20px;
  
  .prompt {
    color: #0f0;
    margin-right: 8px;
  }
  
  .command-input {
    flex: 1;
    background: transparent;
    border: none;
    color: #fff;
    font-family: 'Courier New', Courier, monospace;
    font-size: 16px;
    
    &:focus {
      outline: none;
    }
    
    &::placeholder {
      color: #666;
    }
    
    &:disabled {
      opacity: 0.5;
    }
  }
}

@keyframes pulse {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    opacity: 1;
  }
}

@keyframes thinking {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.3;
  }
  100% {
    opacity: 1;
  }
}
</style> 