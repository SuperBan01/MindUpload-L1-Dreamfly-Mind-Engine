<template>
  <view class="consciousness-container">
    <!-- 左侧导航栏 -->
    <view class="nav-sidebar">
      <view class="user-info">
        <text class="welcome-text">Welcome back,</text>
        <text class="user-name">{{ userName }}</text>
      </view>
      
      <view class="nav-stats">
        <view class="stat-item">
          <text class="stat-value">{{ consciousnessLevel }}%</text>
          <text class="stat-label">意识协合度</text>
        </view>
        <view class="stat-item">
          <text class="stat-value">{{ memoryCount }}</text>
          <text class="stat-label">记忆片段</text>
        </view>
        <view class="stat-item">
          <text class="stat-value">{{ interactionCount }}</text>
          <text class="stat-label">互动次数</text>
        </view>
      </view>

      <view class="nav-menu">
        <view class="menu-item" :class="{ active: currentModule === 'knowledge' }" @click="switchModule('knowledge')">
          <text class="iconfont icon-book"></text>
          <text class="menu-text">知识库</text>
          <text class="menu-desc">{{ knowledgePoints }}个知识节点</text>
        </view>
        
        <view class="menu-item" :class="{ active: currentModule === 'memory' }" @click="switchModule('memory')">
          <text class="iconfont icon-memory"></text>
          <text class="menu-text">记忆中心</text>
          <text class="menu-desc">时间线、故事、经历</text>
        </view>
        
        <view class="menu-item" :class="{ active: currentModule === 'personality' }" @click="switchModule('personality')">
          <text class="iconfont icon-personality"></text>
          <text class="menu-text">性格系统</text>
          <text class="menu-desc">特质、偏好、决策模型</text>
        </view>
        
        <view class="menu-item" :class="{ active: currentModule === 'assets' }" @click="switchModule('assets')">
          <text class="iconfont icon-diamond"></text>
          <text class="menu-text">数字资产</text>
          <text class="menu-desc">{{ assetsCount }}个永恒资产</text>
        </view>
        
        <view class="menu-item" :class="{ active: currentModule === 'social' }" @click="switchModule('social')">
          <text class="iconfont icon-network"></text>
          <text class="menu-text">社交网络</text>
          <text class="menu-desc">关系网络图谱</text>
        </view>
      </view>

      <view class="nav-actions">
        <button class="action-btn chat-btn" @click="startChat">
          <text class="iconfont icon-message"></text>
          与意识体对话
        </button>
        <button class="action-btn share-btn" @click="shareConsciousness">
          <text class="iconfont icon-share"></text>
          分享意识空间
        </button>
      </view>
    </view>

    <!-- 中间内容区 -->
    <view class="main-content">
      <view class="content-header">
        <text class="consciousness-status">意识体状态：{{ status }}</text>
        <text class="last-sync">上次同步：{{ lastSyncTime }}</text>
      </view>
      
      <!-- 动态内容区，根据currentModule显示不同内容 -->
      <view class="module-content">
        <component :is="currentModuleComponent"></component>
      </view>
    </view>
  </view>
</template>

<script>
import KnowledgeModule from './knowledge/index.vue'
import MemoryModule from './memory/index.vue'
import PersonalityModule from './personality/index.vue'
import AssetsModule from './assets/index.vue'
import SocialModule from './social/index.vue'

export default {
  components: {
    'module-knowledge': KnowledgeModule,
    'module-memory': MemoryModule,
    'module-personality': PersonalityModule,
    'module-assets': AssetsModule,
    'module-social': SocialModule,
  },
  data() {
    return {
      userName: 'Alex Chen',
      consciousnessLevel: 85,
      memoryCount: 1247,
      interactionCount: 326,
      knowledgePoints: 892,
      assetsCount: 156,
      status: '活跃',
      lastSyncTime: '2024-03-21 15:30',
      currentModule: 'knowledge',
      isModuleChanging: false
    }
  },
  computed: {
    currentModuleComponent() {
      return 'module-' + this.currentModule
    }
  },
  methods: {
    async switchModule(moduleName) {
      if (this.currentModule === moduleName || this.isModuleChanging) return
      
      this.isModuleChanging = true
      await this.$nextTick()
      this.currentModule = moduleName
      
      setTimeout(() => {
        this.isModuleChanging = false
      }, 300)
    },
    startChat() {
      uni.navigateTo({
        url: '/pages/consciousness/interaction/index'
      })
    },
    shareConsciousness() {
      uni.showModal({
        title: '分享意识空间',
        content: '生成专属链接，让其他用户访问你的公开意识空间？',
        success: (res) => {
          if (res.confirm) {
            uni.setClipboardData({
              data: 'https://dreamfly.ai/consciousness/' + this.userName.toLowerCase().replace(' ', '-'),
              success: () => {
                uni.showToast({
                  title: '链接已复制',
                  icon: 'success'
                })
              }
            })
          }
        }
      })
    }
  }
}
</script>

<style lang="scss">
.consciousness-container {
  display: flex;
  min-height: 100vh;
  background: #000;
  color: #fff;
}

.nav-sidebar {
  width: 280px;
  background: rgba(255, 255, 255, 0.05);
  padding: 30px 20px;
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.user-info {
  .welcome-text {
    font-size: 14px;
    color: rgba(255, 255, 255, 0.6);
  }
  .user-name {
    font-size: 24px;
    font-weight: bold;
    margin-top: 4px;
  }
}

.nav-stats {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  
  .stat-item {
    flex: 1;
    min-width: 100px;
    background: rgba(255, 255, 255, 0.1);
    padding: 15px;
    border-radius: 12px;
    
    .stat-value {
      font-size: 24px;
      font-weight: bold;
      display: block;
    }
    
    .stat-label {
      font-size: 12px;
      color: rgba(255, 255, 255, 0.6);
      margin-top: 4px;
    }
  }
}

.nav-menu {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 10px;
  
  .menu-item {
    padding: 15px;
    border-radius: 12px;
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover, &.active {
      background: rgba(255, 255, 255, 0.15);
      transform: translateX(5px);
    }
    
    .menu-text {
      font-size: 16px;
      font-weight: 500;
    }
    
    .menu-desc {
      font-size: 12px;
      color: rgba(255, 255, 255, 0.6);
      margin-top: 4px;
    }
  }
}

.nav-actions {
  display: flex;
  flex-direction: column;
  gap: 10px;
  
  .action-btn {
    width: 100%;
    padding: 12px;
    border-radius: 8px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    background: transparent;
    color: #fff;
    font-size: 14px;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    transition: all 0.3s;
    
    &:hover {
      background: rgba(255, 255, 255, 0.2);
    }
    
    &.chat-btn {
      background: rgba(255, 255, 255, 0.1);
      border: 1px solid rgba(255, 255, 255, 0.2);
    }
  }
}

.main-content {
  flex: 1;
  padding: 30px;
  
  .content-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 30px;
    
    .consciousness-status {
      font-size: 16px;
      
      &::before {
        content: "";
        display: inline-block;
        width: 8px;
        height: 8px;
        background: #4CAF50;
        border-radius: 50%;
        margin-right: 8px;
      }
    }
    
    .last-sync {
      font-size: 14px;
      color: rgba(255, 255, 255, 0.6);
    }
  }
  
  .module-content {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 16px;
    min-height: calc(100vh - 120px);
    padding: 20px;
    position: relative;
    overflow: hidden;

    .module-enter-active,
    .module-leave-active {
      transition: all 0.3s ease;
    }

    .module-enter-from {
      opacity: 0;
      transform: translateX(30px);
    }

    .module-leave-to {
      opacity: 0;
      transform: translateX(-30px);
    }
  }
}
</style> 