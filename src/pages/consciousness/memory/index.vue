<template>
  <view class="memory-center">
    <view class="module-header">
      <text class="title">记忆中心</text>
      <view class="memory-stats">
        <view class="stat-item">
          <text class="stat-value">{{ totalMemories }}</text>
          <text class="stat-label">记忆片段</text>
        </view>
        <view class="stat-item">
          <text class="stat-value">{{ totalDiaries }}</text>
          <text class="stat-label">日常记录</text>
        </view>
      </view>
    </view>

    <view class="memory-container">
      <!-- 导航标签 -->
      <view class="nav-tabs">
        <button 
          class="tab-btn"
          :class="{ active: currentTab === 'memories' }"
          @click="currentTab = 'memories'"
        >
          生命记忆
        </button>
        <button 
          class="tab-btn"
          :class="{ active: currentTab === 'diary' }"
          @click="currentTab = 'diary'"
        >
          日常心声
        </button>
        <button 
          class="tab-btn"
          :class="{ active: currentTab === 'biography' }"
          @click="currentTab = 'biography'"
        >
          个人传记
        </button>
      </view>

      <!-- 生命记忆模块 -->
      <view v-if="currentTab === 'memories'" class="memories-section">
        <view class="section-header">
          <text class="section-title">生命记忆</text>
          <view class="timeline-filter">
            <select v-model="memoryFilter" class="filter-select">
              <option value="all">全部</option>
              <option value="childhood">童年</option>
              <option value="youth">青年</option>
              <option value="adulthood">成年</option>
            </select>
          </view>
        </view>

        <!-- 记忆引导卡片 -->
        <view class="memory-guides">
          <view 
            v-for="(guide, index) in memoryGuides"
            :key="index"
            class="guide-card"
            @click="startMemoryWithGuide(guide)"
          >
            <view class="guide-icon">
              <text class="icon">{{ guide.icon }}</text>
            </view>
            <view class="guide-content">
              <text class="guide-title">{{ guide.title }}</text>
              <text class="guide-question">{{ guide.question }}</text>
            </view>
          </view>
        </view>

        <!-- 记忆时间线 -->
        <view class="timeline">
          <view 
            v-for="(memory, index) in filteredMemories"
            :key="index"
            class="memory-item"
          >
            <view class="timeline-dot"></view>
            <view class="memory-content">
              <view class="memory-header">
                <text class="memory-title">{{ memory.title }}</text>
                <text class="memory-time">{{ memory.time }}</text>
              </view>
              <text class="memory-text">{{ memory.content }}</text>
              <view class="memory-tags">
                <text 
                  v-for="(tag, tIndex) in memory.tags"
                  :key="tIndex"
                  class="tag"
                >
                  {{ tag }}
                </text>
              </view>
            </view>
          </view>
        </view>

        <!-- 添加记忆按钮 -->
        <view class="float-btn" @click="showMemoryEditor">
          <text class="btn-text">记录记忆</text>
        </view>

        <!-- 记忆编辑器 -->
        <view v-if="showEditor" class="memory-editor">
          <view class="editor-header">
            <input 
              type="text"
              v-model="currentMemory.title"
              class="title-input"
              placeholder="记忆标题"
            >
            <view class="editor-actions">
              <button 
                class="action-btn save"
                @click="saveMemory"
              >
                保存
              </button>
              <button 
                class="action-btn close"
                @click="closeEditor"
              >
                关闭
              </button>
            </view>
          </view>
          <view class="editor-body">
            <input 
              type="text"
              v-model="currentMemory.time"
              class="time-input"
              placeholder="时间（例如：2000年）"
            >
            <textarea
              v-model="currentMemory.content"
              class="content-input"
              placeholder="写下这段记忆..."
            ></textarea>
            <view class="tags-input">
              <input 
                type="text"
                v-model="newTag"
                class="tag-input"
                placeholder="添加标签"
                @keyup.enter="addTag"
              >
              <view class="tags-list">
                <text 
                  v-for="(tag, index) in currentMemory.tags"
                  :key="index"
                  class="tag"
                >
                  {{ tag }}
                  <text class="remove-tag" @click="removeTag(index)">×</text>
                </text>
              </view>
            </view>
          </view>
        </view>
      </view>

      <!-- 日常心声模块 -->
      <view v-if="currentTab === 'diary'" class="diary-section">
        <view class="section-header">
          <text class="section-title">日常心声</text>
          <view class="diary-actions">
            <button 
              class="action-btn import"
              @click="showImportModal"
            >
              导入社交媒体
            </button>
            <view class="mood-filter">
              <select v-model="moodFilter" class="filter-select">
                <option value="all">全部心情</option>
                <option value="happy">开心</option>
                <option value="sad">难过</option>
                <option value="angry">生气</option>
                <option value="peaceful">平静</option>
              </select>
            </view>
          </view>
        </view>

        <!-- 社交媒体导入模态框 -->
        <view v-if="showImport" class="import-modal">
          <view class="modal-content">
            <view class="modal-header">
              <text class="modal-title">导入社交媒体</text>
              <button class="close-btn" @click="closeImportModal">×</button>
            </view>
            <view class="platform-list">
              <view 
                v-for="(platform, index) in socialPlatforms"
                :key="index"
                class="platform-item"
                @click="importFromPlatform(platform)"
              >
                <view class="platform-icon">
                  <text class="icon">{{ platform.icon }}</text>
                </view>
                <text class="platform-name">{{ platform.name }}</text>
                <text class="platform-desc">{{ platform.description }}</text>
              </view>
            </view>
          </view>
        </view>

        <!-- 日记列表 -->
        <view class="diary-list">
          <view 
            v-for="(diary, index) in filteredDiaries"
            :key="index"
            class="diary-card"
          >
            <view class="diary-header">
              <text class="diary-date">{{ diary.date }}</text>
              <text class="diary-mood">{{ diary.mood }}</text>
            </view>
            <text class="diary-content">{{ diary.content }}</text>
          </view>
        </view>

        <!-- 添加日记按钮 -->
        <view class="float-btn" @click="showDiaryEditor">
          <text class="btn-text">写日记</text>
        </view>

        <!-- 日记编辑器 -->
        <view v-if="showDiaryEditor" class="diary-editor">
          <view class="editor-header">
            <view class="date-picker">
              <text class="current-date">{{ currentDate }}</text>
            </view>
            <view class="mood-selector">
              <button 
                v-for="(mood, index) in moods"
                :key="index"
                class="mood-btn"
                :class="{ active: currentDiary.mood === mood.value }"
                @click="currentDiary.mood = mood.value"
              >
                {{ mood.label }}
              </button>
            </view>
          </view>
          <textarea
            v-model="currentDiary.content"
            class="diary-content-input"
            placeholder="今天发生了什么？有什么想说的？"
          ></textarea>
          <view class="editor-footer">
            <button 
              class="action-btn save"
              @click="saveDiary"
            >
              保存
            </button>
            <button 
              class="action-btn close"
              @click="closeDiaryEditor"
            >
              关闭
            </button>
          </view>
        </view>
      </view>

      <!-- 个人传记模块 -->
      <view v-if="currentTab === 'biography'" class="biography-section">
        <view class="section-header">
          <text class="section-title">个人传记</text>
          <button 
            class="new-chapter-btn"
            @click="addNewChapter"
          >
            新增章节
          </button>
        </view>

        <!-- 传记章节列表 -->
        <view class="chapters-grid">
          <view 
            v-for="(chapter, index) in biographyChapters"
            :key="index"
            class="chapter-card"
            @click="openChapter(chapter)"
          >
            <view class="chapter-cover">
              <text class="chapter-number">{{ index + 1 }}</text>
              <text class="chapter-title">{{ chapter.title }}</text>
              <text class="chapter-preview">{{ chapter.content.slice(0, 100) }}...</text>
            </view>
          </view>
          <view 
            class="chapter-card add-chapter"
            @click="addNewChapter"
          >
            <view class="add-chapter-content">
              <text class="add-icon">+</text>
              <text class="add-text">新增章节</text>
            </view>
          </view>
        </view>

        <!-- 章节阅读器 -->
        <view v-if="showChapterReader" class="chapter-reader">
          <view class="reader-header">
            <view class="reader-nav">
              <button 
                class="nav-btn"
                :disabled="!hasPreviousChapter"
                @click="readPreviousChapter"
              >
                上一章
              </button>
              <text class="chapter-title">{{ currentChapter.title }}</text>
              <button 
                class="nav-btn"
                :disabled="!hasNextChapter"
                @click="readNextChapter"
              >
                下一章
              </button>
            </view>
            <view class="reader-actions">
              <button 
                class="action-btn edit"
                @click="editChapter"
              >
                编辑
              </button>
              <button 
                class="action-btn close"
                @click="closeReader"
              >
                关闭
              </button>
            </view>
          </view>
          <view class="reader-content">
            <text class="chapter-text">{{ currentChapter.content }}</text>
          </view>
        </view>

        <!-- 章节编辑器 -->
        <view v-if="showChapterEditor" class="chapter-editor">
          <view class="editor-header">
            <input 
              type="text"
              v-model="currentChapter.title"
              class="title-input"
              placeholder="章节标题"
            >
            <view class="editor-actions">
              <button 
                class="action-btn polish"
                @click="showAIPolish"
              >
                AI润色
              </button>
              <button 
                class="action-btn save"
                @click="saveChapter"
              >
                保存
              </button>
              <button 
                class="action-btn close"
                @click="closeChapterEditor"
              >
                关闭
              </button>
            </view>
          </view>
          <view class="editor-body">
            <view class="writing-guide">
              <text class="guide-title">写作建议</text>
              <view class="guide-tips">
                <text class="tip">• 从一个具体的场景或故事开始</text>
                <text class="tip">• 描述当时的感受和想法</text>
                <text class="tip">• 记录这段经历对你的影响</text>
                <text class="tip">• 分享你的思考和感悟</text>
              </view>
              <view class="example-toggle" @click="toggleExample">
                <text class="toggle-text">查看示例 {{ showExample ? '▼' : '▶' }}</text>
              </view>
              <view v-if="showExample" class="writing-example">
                <text class="example-title">示例：童年与教育</text>
                <text class="example-content">那是1967年的一个下午，在惠普公司的实验室里，我第一次见到了电脑。那个巨大的机器发出嗡嗡的声响，屏幕上闪烁的绿色文字让我着迷。我的养父保罗·乔布斯正在向我解释这台机器的工作原理。

这次经历深深地影响了我。它让我意识到，科技不仅仅是冰冷的机器，而是能够改变世界的工具。我的养父母给了我最好的教育，他们不仅支持我的兴趣，更教会了我追求完美的态度。

在里德学院的书法课上，我学习到了字体的优雅与平衡。这些看似与计算机无关的知识，后来却成为了Mac与个人计算机革命的重要一环。</text>
              </view>
            </view>
            <textarea
              v-model="currentChapter.content"
              class="chapter-content-input"
              placeholder="开始撰写这一章节..."
            ></textarea>
          </view>
        </view>

        <!-- AI润色模态框 -->
        <view v-if="showPolishModal" class="polish-modal">
          <view class="modal-content">
            <view class="modal-header">
              <text class="modal-title">AI润色</text>
              <button class="close-btn" @click="closePolishModal">×</button>
            </view>
            <view class="polish-options">
              <button 
                v-for="(style, index) in polishStyles"
                :key="index"
                class="style-btn"
                :class="{ active: selectedStyle === style.value }"
                @click="selectedStyle = style.value"
              >
                {{ style.label }}
              </button>
            </view>
            <button 
              class="polish-btn"
              @click="applyPolish"
            >
              开始润色
            </button>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  name: 'MemoryCenter',
  data() {
    return {
      currentTab: 'memories',
      totalMemories: 0,
      totalDiaries: 0,
      showEditor: false,
      showDiaryEditor: false,
      showChapterEditor: false,
      showChapterReader: false,
      showGuide: false,
      showAIWriter: false,
      memoryFilter: 'all',
      moodFilter: 'all',
      currentMemory: {
        title: '',
        time: '',
        content: '',
        tags: []
      },
      currentDiary: {
        date: '',
        mood: '',
        content: ''
      },
      currentChapter: {
        title: '',
        content: ''
      },
      newTag: '',
      memories: [
        {
          id: '1',
          title: '第一次见到电脑',
          time: '1967年',
          content: '在惠普公司的实验室里，我第一次见到了电脑。那是一个巨大的机器，但它让我着迷。我意识到，这些机器不仅仅是工具，它们是人类思想的延伸。',
          tags: ['童年', '科技']
        },
        {
          id: '2',
          title: '创立苹果',
          time: '1976年',
          content: '在车库里，我和沃兹尼亚克一起创造了第一台苹果电脑。我们相信，电脑应该是个人的，应该让每个人都能使用。这个信念一直指引着我。',
          tags: ['创业', '科技']
        },
        {
          id: '3',
          title: '被苹果解雇',
          time: '1985年',
          content: '被自己创立的公司解雇，这是我人生中最痛苦的经历。但正是这次失败，让我重新思考什么是真正重要的。',
          tags: ['挫折', '成长']
        },
        {
          id: '4',
          title: '重返苹果',
          time: '1997年',
          content: '回到苹果时，公司濒临破产。我意识到，我们需要重新定义什么是伟大的产品。不是技术，而是技术与人文的结合。',
          tags: ['回归', '创新']
        }
      ],
      diaries: [
        {
          id: '1',
          date: '2005年6月12日',
          mood: 'peaceful',
          content: "今天在斯坦福大学的毕业典礼上演讲。我告诉学生们：'Stay Hungry, Stay Foolish'。这是我从《Whole Earth Catalog》中学到的，也是我一直以来的人生信条。",
          source: '演讲'
        },
        {
          id: '2',
          date: '2007年1月9日',
          mood: 'happy',
          content: '今天发布了iPhone。这不是一部手机，而是一个革命性的设备。它改变了人们的生活方式，就像Mac改变了个人电脑一样。',
          source: '发布会'
        }
      ],
      biographyChapters: [
        {
          title: '童年与教育',
          progress: 80,
          content: '我的养父母给了我最好的教育。他们教会我追求卓越，教会我思考。在里德学院的日子里，我学习了书法，这后来影响了Mac的字体设计。',
          memories: ['1']
        },
        {
          title: '创业之路',
          progress: 90,
          content: '创立苹果是我人生中最重要的事情之一。我们相信，科技应该服务于人性，而不是相反。这种理念贯穿了我们的所有产品。',
          memories: ['2']
        },
        {
          title: '挫折与重生',
          progress: 100,
          content: '被苹果解雇是我人生中最痛苦的经历，但也是最有价值的。它让我重新思考什么是真正重要的，让我有机会创办NeXT和皮克斯。',
          memories: ['3']
        },
        {
          title: '最后的辉煌',
          progress: 100,
          content: '回到苹果后，我们重新定义了什么是伟大的产品。iMac、iPod、iPhone、iPad，每一个产品都体现了我们对完美的追求。',
          memories: ['4']
        }
      ],
      moods: [
        { label: '开心', value: 'happy' },
        { label: '难过', value: 'sad' },
        { label: '生气', value: 'angry' },
        { label: '平静', value: 'peaceful' }
      ],
      memoryGuides: [
        {
          icon: '💡',
          title: '创新时刻',
          question: '你第一次意识到科技可以改变世界是什么时候？'
        },
        {
          icon: '🎯',
          title: '人生转折',
          question: '你经历过的最重要的失败是什么？它如何改变了你？'
        },
        {
          icon: '🎨',
          title: '设计理念',
          question: "你如何定义'完美'？在你的工作中，你追求什么样的设计？",
        },
        {
          icon: '🌱',
          title: '成长经历',
          question: '哪些经历塑造了你的价值观和人生观？'
        }
      ],
      showImport: false,
      socialPlatforms: [
        {
          icon: '📱',
          name: '微博',
          description: '导入你的微博内容'
        },
        {
          icon: '📝',
          name: '即刻',
          description: '导入你的即刻动态'
        },
        {
          icon: '📸',
          name: '小红书',
          description: '导入你的小红书笔记'
        },
        {
          icon: '📅',
          name: '演讲',
          description: '导入你的演讲内容'
        }
      ],
      aiPrompt: '',
      hasPreviousChapter: false,
      hasNextChapter: false,
      showExample: false,
      showPolishModal: false,
      selectedStyle: 'narrative',
      polishStyles: [
        { label: '叙事优化', value: 'narrative' },
        { label: '文学风格', value: 'literary' },
        { label: '简洁清晰', value: 'concise' },
        { label: '细节丰富', value: 'detailed' }
      ]
    }
  },
  computed: {
    currentDate() {
      const now = new Date()
      return `${now.getFullYear()}年${now.getMonth() + 1}月${now.getDate()}日`
    },
    filteredMemories() {
      if (this.memoryFilter === 'all') return this.memories
      return this.memories.filter(memory => memory.tags.includes(this.memoryFilter))
    },
    filteredDiaries() {
      if (this.moodFilter === 'all') return this.diaries
      return this.diaries.filter(diary => diary.mood === this.moodFilter)
    }
  },
  methods: {
    showMemoryEditor() {
      this.currentMemory = {
        title: '',
        time: '',
        content: '',
        tags: []
      }
      this.showEditor = true
    },
    closeEditor() {
      this.showEditor = false
    },
    saveMemory() {
      if (!this.currentMemory.title || !this.currentMemory.content) {
        uni.showToast({
          title: '请填写完整信息',
          icon: 'none'
        })
        return
      }

      const memory = {
        ...this.currentMemory,
        id: Date.now().toString()
      }
      this.memories.unshift(memory)
      this.totalMemories++
      
      // 自动更新传记内容
      this.updateBiography(memory)
      
      this.closeEditor()
    },
    updateBiography(memory) {
      // 根据记忆内容自动更新相关章节
      const relevantChapters = this.biographyChapters.filter(chapter => {
        if (memory.tags.includes('童年') && chapter.title === '童年与教育') return true
        if (memory.tags.includes('创业') && chapter.title === '创业之路') return true
        if (memory.tags.includes('挫折') && chapter.title === '挫折与重生') return true
        if (memory.tags.includes('创新') && chapter.title === '最后的辉煌') return true
        return false
      })
      
      relevantChapters.forEach(chapter => {
        chapter.memories.push(memory.id)
        chapter.progress = Math.min(100, chapter.memories.length * 25)
        
        // 使用AI生成章节内容
        this.generateChapterContent(chapter)
      })
    },
    
    generateChapterContent(chapter) {
      // 模拟AI生成内容
      const relatedMemories = this.memories.filter(m => chapter.memories.includes(m.id))
      const memoryTexts = relatedMemories.map(m => m.content).join('\n')
      
      // 这里应该调用实际的AI接口
      // 目前使用简单的文本拼接作为示例
      chapter.content = memoryTexts
    },
    addTag() {
      if (this.newTag && !this.currentMemory.tags.includes(this.newTag)) {
        this.currentMemory.tags.push(this.newTag)
        this.newTag = ''
      }
    },
    removeTag(index) {
      this.currentMemory.tags.splice(index, 1)
    },
    showDiaryEditor() {
      this.currentDiary = {
        date: this.currentDate,
        mood: '',
        content: ''
      }
      this.showDiaryEditor = true
    },
    closeDiaryEditor() {
      this.showDiaryEditor = false
    },
    saveDiary() {
      if (!this.currentDiary.content) {
        uni.showToast({
          title: '请填写日记内容',
          icon: 'none'
        })
        return
      }

      const diary = {
        ...this.currentDiary,
        id: Date.now().toString()
      }
      this.diaries.unshift(diary)
      this.totalDiaries++
      this.closeDiaryEditor()
    },
    showBiographyGuide() {
      this.showGuide = true
    },
    closeBiographyGuide() {
      this.showGuide = false
    },
    showBiographyEditor(chapter) {
      this.currentChapter = {
        ...chapter,
        content: chapter.content || ''
      }
      this.showChapterEditor = true
    },
    closeChapterEditor() {
      this.showChapterEditor = false
      this.currentChapter = {
        title: '',
        content: '',
        memories: []
      }
      // 返回到传记列表视图
      this.showChapterReader = false
    },
    saveChapter() {
      if (!this.currentChapter.title || !this.currentChapter.content) {
        uni.showToast({
          title: '请填写完整信息',
          icon: 'none'
        })
        return
      }

      const index = this.biographyChapters.findIndex(
        chapter => chapter.title === this.currentChapter.title
      )
      
      if (index !== -1) {
        this.biographyChapters[index] = {
          ...this.currentChapter,
          progress: 100
        }
      } else {
        this.biographyChapters.push({
          ...this.currentChapter,
          progress: 100,
          memories: []
        })
      }
      
      uni.showToast({
        title: '保存成功',
        icon: 'success'
      })
      
      // 关闭编辑器，返回到传记列表视图
      this.closeChapterEditor()
    },
    startMemoryWithGuide(guide) {
      this.currentMemory = {
        title: guide.title,
        time: '',
        content: '',
        tags: []
      }
      this.showEditor = true
    },
    showImportModal() {
      this.showImport = true
    },
    closeImportModal() {
      this.showImport = false
    },
    importFromPlatform(platform) {
      // 模拟导入演讲内容
      if (platform.name === '演讲') {
        const newDiary = {
          id: Date.now().toString(),
          date: this.currentDate,
          mood: 'peaceful',
          content: "今天在斯坦福大学的毕业典礼上演讲。我告诉学生们：'Stay Hungry, Stay Foolish'。这是我从《Whole Earth Catalog》中学到的，也是我一直以来的人生信条。",
          source: '演讲'
        }
        this.diaries.unshift(newDiary)
        this.totalDiaries++
        
        uni.showToast({
          title: '导入成功',
          icon: 'success'
        })
      } else {
        uni.showToast({
          title: `正在从${platform.name}导入内容...`,
          icon: 'none'
        })
      }
      
      this.closeImportModal()
    },
    showAIWriter() {
      this.showAIWriter = true
    },
    closeAIWriter() {
      this.showAIWriter = false
      this.aiPrompt = ''
    },
    generateBiography() {
      if (!this.aiPrompt) {
        uni.showToast({
          title: '请描述你想要生成的内容',
          icon: 'none'
        })
        return
      }

      // 模拟AI生成传记内容
      const selectedChapter = this.biographyChapters.find(
        chapter => chapter.title.toLowerCase().includes(this.aiPrompt.toLowerCase())
      )
      
      if (selectedChapter) {
        // 这里应该调用实际的AI接口
        // 目前使用简单的文本作为示例
        selectedChapter.content = `根据您的要求，我为您生成了一段关于${selectedChapter.title}的内容。这段内容融合了您的记忆和经历，展现了您独特的人生轨迹。`
        selectedChapter.progress = 100
        
        uni.showToast({
          title: '生成成功',
          icon: 'success'
        })
      } else {
        uni.showToast({
          title: '请选择具体的章节',
          icon: 'none'
        })
      }
      
      this.closeAIWriter()
    },
    addNewChapter() {
      this.currentChapter = {
        title: '',
        content: '',
        memories: []
      }
      this.showChapterEditor = true
    },
    
    openChapter(chapter) {
      this.currentChapter = { ...chapter }
      this.showChapterReader = true
      this.updateChapterNavigation()
    },
    
    closeReader() {
      this.showChapterReader = false
    },
    
    editChapter() {
      this.showChapterReader = false
      this.showChapterEditor = true
    },
    
    updateChapterNavigation() {
      const currentIndex = this.biographyChapters.findIndex(
        chapter => chapter.title === this.currentChapter.title
      )
      this.hasPreviousChapter = currentIndex > 0
      this.hasNextChapter = currentIndex < this.biographyChapters.length - 1
    },
    
    readPreviousChapter() {
      const currentIndex = this.biographyChapters.findIndex(
        chapter => chapter.title === this.currentChapter.title
      )
      if (currentIndex > 0) {
        this.currentChapter = { ...this.biographyChapters[currentIndex - 1] }
        this.updateChapterNavigation()
      }
    },
    
    readNextChapter() {
      const currentIndex = this.biographyChapters.findIndex(
        chapter => chapter.title === this.currentChapter.title
      )
      if (currentIndex < this.biographyChapters.length - 1) {
        this.currentChapter = { ...this.biographyChapters[currentIndex + 1] }
        this.updateChapterNavigation()
      }
    },
    toggleExample() {
      this.showExample = !this.showExample
    },
    
    showAIPolish() {
      this.showPolishModal = true
    },
    
    closePolishModal() {
      this.showPolishModal = false
      this.selectedStyle = 'narrative'
    },
    
    applyPolish() {
      // 这里应该调用实际的AI接口
      uni.showToast({
        title: '正在润色中...',
        icon: 'loading',
        duration: 2000
      })
      
      setTimeout(() => {
        // 模拟AI润色结果
        const polishedContent = this.currentChapter.content + '\n\n[AI润色后的内容会替换这里]'
        this.currentChapter.content = polishedContent
        
        this.closePolishModal()
        
        uni.showToast({
          title: '润色完成',
          icon: 'success'
        })
      }, 2000)
    }
  }
}
</script>

<style lang="scss">
.memory-center {
  height: 100%;
  
  .module-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 30px;
    
    .title {
      font-size: 24px;
      font-weight: bold;
    }
    
    .memory-stats {
      display: flex;
      gap: 20px;
      
      .stat-item {
        text-align: right;
        
        .stat-value {
          font-size: 24px;
          font-weight: bold;
          display: block;
        }
        
        .stat-label {
          font-size: 14px;
          color: rgba(255, 255, 255, 0.6);
        }
      }
    }
  }
  
  .memory-container {
    .nav-tabs {
      display: flex;
      gap: 16px;
      margin-bottom: 24px;
      
      .tab-btn {
        padding: 8px 24px;
        border-radius: 8px;
        font-size: 16px;
        background: transparent;
        border: 1px solid rgba(255, 255, 255, 0.2);
        color: rgba(255, 255, 255, 0.8);
        cursor: pointer;
        transition: all 0.3s;
        
        &:hover {
          background: rgba(255, 255, 255, 0.05);
        }
        
        &.active {
          background: rgba(255, 255, 255, 0.1);
          border-color: rgba(255, 255, 255, 0.3);
          color: #fff;
        }
      }
    }
    
    .section-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 24px;
      
      .section-title {
        font-size: 20px;
        font-weight: 500;
      }
      
      .filter-select {
        background: rgba(255, 255, 255, 0.1);
        border: 1px solid rgba(255, 255, 255, 0.2);
        border-radius: 6px;
        padding: 8px 12px;
        color: #fff;
        font-size: 14px;
        min-width: 120px;
        
        &:focus {
          outline: none;
          border-color: rgba(255, 255, 255, 0.3);
        }
      }
    }
    
    .timeline {
      position: relative;
      padding-left: 20px;
      
      &::before {
        content: '';
        position: absolute;
        left: 0;
        top: 0;
        bottom: 0;
        width: 2px;
        background: rgba(255, 255, 255, 0.1);
      }
      
      .memory-item {
        position: relative;
        margin-bottom: 24px;
        
        .timeline-dot {
          position: absolute;
          left: -25px;
          top: 8px;
          width: 12px;
          height: 12px;
          border-radius: 50%;
          background: rgba(255, 255, 255, 0.2);
        }
        
        .memory-content {
          background: rgba(255, 255, 255, 0.05);
          border-radius: 12px;
          padding: 16px;
          
          .memory-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
            
            .memory-title {
              font-size: 16px;
              font-weight: 500;
            }
            
            .memory-time {
              font-size: 12px;
              color: rgba(255, 255, 255, 0.4);
            }
          }
          
          .memory-text {
            font-size: 14px;
            color: rgba(255, 255, 255, 0.8);
            margin-bottom: 12px;
            display: block;
          }
          
          .memory-tags {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            
            .tag {
              padding: 4px 8px;
              border-radius: 4px;
              font-size: 12px;
              background: rgba(255, 255, 255, 0.1);
              color: rgba(255, 255, 255, 0.8);
            }
          }
        }
      }
    }
    
    .diary-list {
      display: flex;
      flex-direction: column;
      gap: 16px;
      
      .diary-card {
        background: rgba(255, 255, 255, 0.05);
        border-radius: 12px;
        padding: 16px;
        
        .diary-header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 12px;
          
          .diary-date {
            font-size: 14px;
            color: rgba(255, 255, 255, 0.6);
          }
          
          .diary-mood {
            font-size: 12px;
            padding: 4px 8px;
            border-radius: 4px;
            background: rgba(255, 255, 255, 0.1);
          }
        }
        
        .diary-content {
          font-size: 14px;
          color: rgba(255, 255, 255, 0.8);
          line-height: 1.6;
        }
      }
    }
    
    .biography-section {
      .section-header {
        .new-chapter-btn {
          padding: 8px 16px;
          border-radius: 8px;
          font-size: 14px;
          background: rgba(255, 255, 255, 0.1);
          border: 1px solid rgba(255, 255, 255, 0.2);
          color: #fff;
          cursor: pointer;
          transition: all 0.3s;
          
          &:hover {
            background: rgba(255, 255, 255, 0.15);
            border-color: rgba(255, 255, 255, 0.3);
          }
        }
      }

      .chapters-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
        gap: 24px;
        padding: 24px 0;
        
        .chapter-card {
          aspect-ratio: 3/4;
          background: rgba(255, 255, 255, 0.05);
          border-radius: 16px;
          overflow: hidden;
          cursor: pointer;
          transition: all 0.3s;
          border: 1px solid rgba(255, 255, 255, 0.1);
          
          &:hover {
            transform: translateY(-4px);
            background: rgba(255, 255, 255, 0.08);
            border-color: rgba(255, 255, 255, 0.2);
          }
          
          .chapter-cover {
            height: 100%;
            padding: 24px;
            display: flex;
            flex-direction: column;
            
            .chapter-number {
              font-size: 48px;
              font-weight: 200;
              color: rgba(255, 255, 255, 0.2);
              margin-bottom: 16px;
            }
            
            .chapter-title {
              font-size: 20px;
              font-weight: 500;
              margin-bottom: 16px;
              line-height: 1.4;
            }
            
            .chapter-preview {
              font-size: 14px;
              color: rgba(255, 255, 255, 0.6);
              line-height: 1.6;
              display: -webkit-box;
              -webkit-line-clamp: 4;
              -webkit-box-orient: vertical;
              overflow: hidden;
            }
          }
          
          &.add-chapter {
            background: transparent;
            border: 2px dashed rgba(255, 255, 255, 0.1);
            display: flex;
            align-items: center;
            justify-content: center;
            
            &:hover {
              border-color: rgba(255, 255, 255, 0.3);
              background: transparent;
            }
            
            .add-chapter-content {
              text-align: center;
              
              .add-icon {
                font-size: 32px;
                color: rgba(255, 255, 255, 0.3);
                margin-bottom: 8px;
                display: block;
              }
              
              .add-text {
                font-size: 16px;
                color: rgba(255, 255, 255, 0.4);
              }
            }
          }
        }
      }

      .chapter-reader {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: #1f1f1f;
        z-index: 1000;
        display: flex;
        flex-direction: column;
        
        .reader-header {
          padding: 24px;
          border-bottom: 1px solid rgba(255, 255, 255, 0.1);
          display: flex;
          justify-content: space-between;
          align-items: center;
          background: rgba(0, 0, 0, 0.2);
          
          .reader-nav {
            display: flex;
            align-items: center;
            gap: 24px;
            
            .nav-btn {
              padding: 8px 16px;
              border-radius: 8px;
              font-size: 14px;
              background: rgba(255, 255, 255, 0.1);
              border: 1px solid rgba(255, 255, 255, 0.2);
              color: #fff;
              cursor: pointer;
              transition: all 0.3s;
              
              &:disabled {
                opacity: 0.5;
                cursor: not-allowed;
              }
              
              &:not(:disabled):hover {
                background: rgba(255, 255, 255, 0.15);
              }
            }
            
            .chapter-title {
              font-size: 20px;
              font-weight: 500;
            }
          }
          
          .reader-actions {
            display: flex;
            gap: 12px;
          }
        }
        
        .reader-content {
          flex: 1;
          padding: 48px;
          max-width: 800px;
          margin: 0 auto;
          width: 100%;
          overflow-y: auto;
          
          .chapter-text {
            font-size: 16px;
            line-height: 1.8;
            color: rgba(255, 255, 255, 0.9);
            white-space: pre-wrap;
          }
        }
      }

      .chapter-editor {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: #1f1f1f;
        z-index: 1000;
        display: flex;
        flex-direction: column;
        
        .editor-header {
          padding: 24px;
          border-bottom: 1px solid rgba(255, 255, 255, 0.1);
          display: flex;
          justify-content: space-between;
          align-items: center;
          background: rgba(0, 0, 0, 0.2);
          
          .title-input {
            background: rgba(255, 255, 255, 0.1);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 8px;
            padding: 12px 16px;
            font-size: 18px;
            font-weight: 500;
            color: #fff;
            width: 70%;
            transition: all 0.3s;
            
            &:focus {
              outline: none;
              border-color: rgba(255, 255, 255, 0.3);
              background: rgba(255, 255, 255, 0.15);
            }

            &::placeholder {
              color: rgba(255, 255, 255, 0.4);
            }
          }
          
          .editor-actions {
            display: flex;
            gap: 12px;
            
            .action-btn {
              padding: 10px 20px;
              border-radius: 8px;
              font-size: 14px;
              font-weight: 500;
              cursor: pointer;
              transition: all 0.3s;
              
              &.polish {
                background: rgba(255, 255, 255, 0.15);
                margin-right: 12px;
                
                &:hover {
                  background: rgba(255, 255, 255, 0.2);
                }
              }
              
              &.save {
                background: rgba(255, 255, 255, 0.1);
                color: #fff;
                border: 1px solid rgba(255, 255, 255, 0.2);
                
                &:hover {
                  background: rgba(255, 255, 255, 0.15);
                  border-color: rgba(255, 255, 255, 0.3);
                }
              }
              
              &.close {
                background: transparent;
                border: 1px solid rgba(255, 255, 255, 0.2);
                color: rgba(255, 255, 255, 0.8);
                
                &:hover {
                  background: rgba(255, 255, 255, 0.05);
                  border-color: rgba(255, 255, 255, 0.3);
                }
              }
            }
          }
        }
        
        .editor-body {
          flex: 1;
          padding: 24px;
          display: flex;
          flex-direction: column;
          gap: 20px;
          overflow-y: auto;
          max-width: 800px;
          margin: 0 auto;
          width: 100%;
          
          .writing-guide {
            background: rgba(255, 255, 255, 0.05);
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 24px;
            
            .guide-title {
              font-size: 18px;
              font-weight: 500;
              margin-bottom: 16px;
              display: block;
            }
            
            .guide-tips {
              display: flex;
              flex-direction: column;
              gap: 8px;
              margin-bottom: 16px;
              
              .tip {
                font-size: 14px;
                color: rgba(255, 255, 255, 0.8);
                line-height: 1.6;
              }
            }
            
            .example-toggle {
              cursor: pointer;
              margin-bottom: 12px;
              
              .toggle-text {
                color: rgba(255, 255, 255, 0.6);
                font-size: 14px;
                
                &:hover {
                  color: rgba(255, 255, 255, 0.8);
                }
              }
            }
            
            .writing-example {
              background: rgba(0, 0, 0, 0.2);
              border-radius: 8px;
              padding: 16px;
              
              .example-title {
                font-size: 16px;
                font-weight: 500;
                margin-bottom: 12px;
                display: block;
              }
              
              .example-content {
                font-size: 14px;
                color: rgba(255, 255, 255, 0.8);
                line-height: 1.8;
                white-space: pre-wrap;
              }
            }
          }
          
          .time-input,
          .content-input,
          .tag-input {
            background: rgba(255, 255, 255, 0.1);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 8px;
            padding: 12px 16px;
            color: #fff;
            font-size: 16px;
            width: 100%;
            transition: all 0.3s;
            
            &:focus {
              outline: none;
              border-color: rgba(255, 255, 255, 0.3);
              background: rgba(255, 255, 255, 0.15);
            }

            &::placeholder {
              color: rgba(255, 255, 255, 0.4);
            }
          }
          
          .content-input {
            min-height: 200px;
            line-height: 1.6;
            resize: none;
          }
        }
      }
    }
  }
}

.float-btn {
  position: fixed;
  right: 24px;
  bottom: 24px;
  padding: 12px 24px;
  border-radius: 8px;
  font-size: 16px;
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.2);
  cursor: pointer;
  transition: all 0.3s;
  
  &:hover {
    background: rgba(255, 255, 255, 0.15);
    transform: translateY(-2px);
  }
  
  .btn-text {
    font-weight: 500;
  }
}

.memory-guides {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;
  margin-bottom: 24px;
  
  .guide-card {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    padding: 16px;
    cursor: pointer;
    transition: all 0.3s;
    display: flex;
    align-items: center;
    gap: 16px;
    
    &:hover {
      background: rgba(255, 255, 255, 0.08);
      transform: translateY(-2px);
    }
    
    .guide-icon {
      width: 48px;
      height: 48px;
      border-radius: 50%;
      background: rgba(255, 255, 255, 0.1);
      display: flex;
      align-items: center;
      justify-content: center;
      
      .icon {
        font-size: 24px;
      }
    }
    
    .guide-content {
      flex: 1;
      
      .guide-title {
        font-size: 16px;
        font-weight: 500;
        margin-bottom: 4px;
        display: block;
      }
      
      .guide-question {
        font-size: 14px;
        color: rgba(255, 255, 255, 0.6);
        line-height: 1.4;
      }
    }
  }
}

.import-modal,
.ai-writer-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  
  .modal-content {
    background: rgba(31, 31, 31, 0.95);
    border-radius: 16px;
    padding: 24px;
    width: 90%;
    max-width: 500px;
    backdrop-filter: blur(10px);
    
    .modal-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 24px;
      
      .modal-title {
        font-size: 20px;
        font-weight: 500;
      }
      
      .close-btn {
        background: transparent;
        border: none;
        color: rgba(255, 255, 255, 0.6);
        font-size: 24px;
        cursor: pointer;
        
        &:hover {
          color: #fff;
        }
      }
    }
  }
}

.platform-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
  
  .platform-item {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    padding: 16px;
    cursor: pointer;
    transition: all 0.3s;
    display: flex;
    align-items: center;
    gap: 16px;
    
    &:hover {
      background: rgba(255, 255, 255, 0.08);
    }
    
    .platform-icon {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: rgba(255, 255, 255, 0.1);
      display: flex;
      align-items: center;
      justify-content: center;
      
      .icon {
        font-size: 20px;
      }
    }
    
    .platform-name {
      font-size: 16px;
      font-weight: 500;
      margin-bottom: 4px;
      display: block;
    }
    
    .platform-desc {
      font-size: 14px;
      color: rgba(255, 255, 255, 0.6);
    }
  }
}

.ai-writer-content {
  .prompt-input {
    width: 100%;
    height: 120px;
    background: rgba(255, 255, 255, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 8px;
    padding: 16px;
    color: #fff;
    font-size: 16px;
    margin-bottom: 16px;
    resize: none;
    
    &:focus {
      outline: none;
      border-color: rgba(255, 255, 255, 0.3);
      background: rgba(255, 255, 255, 0.15);
    }
  }
  
  .generate-btn {
    width: 100%;
    padding: 12px;
    border-radius: 8px;
    font-size: 16px;
    background: rgba(255, 255, 255, 0.1);
    color: #fff;
    border: 1px solid rgba(255, 255, 255, 0.2);
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover {
      background: rgba(255, 255, 255, 0.15);
    }
  }
}

.action-btn {
  padding: 8px 16px;
  border-radius: 8px;
  font-size: 14px;
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.2);
  cursor: pointer;
  transition: all 0.3s;
  
  &:hover {
    background: rgba(255, 255, 255, 0.15);
  }
  
  &.import {
    background: rgba(255, 255, 255, 0.05);
    margin-right: 16px;
  }
  
  &.ai-write {
    background: rgba(255, 255, 255, 0.05);
    margin-right: 16px;
  }
}

.polish-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1100;
  
  .modal-content {
    background: rgba(31, 31, 31, 0.95);
    border-radius: 16px;
    padding: 24px;
    width: 90%;
    max-width: 500px;
    backdrop-filter: blur(10px);
    
    .polish-options {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 12px;
      margin: 20px 0;
      
      .style-btn {
        padding: 12px;
        border-radius: 8px;
        font-size: 14px;
        background: rgba(255, 255, 255, 0.1);
        border: 1px solid rgba(255, 255, 255, 0.2);
        color: #fff;
        cursor: pointer;
        transition: all 0.3s;
        
        &:hover {
          background: rgba(255, 255, 255, 0.15);
        }
        
        &.active {
          background: rgba(255, 255, 255, 0.2);
          border-color: rgba(255, 255, 255, 0.4);
        }
      }
    }
    
    .polish-btn {
      width: 100%;
      padding: 12px;
      border-radius: 8px;
      font-size: 16px;
      background: rgba(255, 255, 255, 0.15);
      color: #fff;
      border: 1px solid rgba(255, 255, 255, 0.3);
      cursor: pointer;
      transition: all 0.3s;
      
      &:hover {
        background: rgba(255, 255, 255, 0.2);
      }
    }
  }
}
</style> 