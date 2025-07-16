<template>
  <view class="digital-assets">
    <view class="module-header">
      <text class="title">数字资产管理</text>
      <view class="asset-stats">
        <view class="stat-item">
          <text class="stat-value">{{ totalAssetValue }}</text>
          <text class="stat-label">总资产估值</text>
        </view>
        <view class="stat-item">
          <text class="stat-value">{{ monthlyRevenue }}</text>
          <text class="stat-label">月度收益</text>
        </view>
      </view>
    </view>

    <view class="assets-container">
      <!-- 意识体看板 -->
      <view class="consciousness-dashboard">
        <view class="section-header">
          <text class="section-title">意识体看板</text>
          <view class="dashboard-tabs">
            <button 
              class="tab-btn"
              :class="{ active: currentTab === 'owned' }"
              @click="currentTab = 'owned'"
            >
              我的意识体
            </button>
            <button 
              class="tab-btn"
              :class="{ active: currentTab === 'purchased' }"
              @click="currentTab = 'purchased'"
            >
              已获取意识体
            </button>
          </view>
        </view>

        <view class="consciousness-cards">
          <view 
            v-for="(consciousness, index) in currentConsciousness"
            :key="index"
            class="consciousness-detail-card"
          >
            <view class="card-main">
              <view class="consciousness-avatar">
                <image 
                  :src="consciousness.avatar || '/static/avatar.jpg'"
                  class="avatar-img"
                />
                <view 
                  class="status-indicator"
                  :class="consciousness.status"
                ></view>
              </view>
              <view class="consciousness-info">
                <view class="info-header">
                  <text class="consciousness-name">{{ consciousness.name }}</text>
                  <view class="header-actions">
                    <text class="consciousness-type">{{ consciousness.type }}</text>
                    <button 
                      class="action-btn publish"
                      :class="{ 'published': consciousness.status === 'published' }"
                      @click="togglePublish(consciousness)"
                    >
                      {{ consciousness.status === 'published' ? '下架' : '发布' }}
                    </button>
                  </view>
                </view>
                <text class="consciousness-desc">{{ consciousness.description }}</text>
                <view class="consciousness-attributes">
                  <view class="attribute-item">
                    <text class="attribute-icon">🧠</text>
                    <text class="attribute-label">意识完整度</text>
                    <text class="attribute-value">{{ consciousness.completeness }}%</text>
                  </view>
                  <view class="attribute-item">
                    <text class="attribute-icon">⚡</text>
                    <text class="attribute-label">活跃度</text>
                    <text class="attribute-value">{{ consciousness.activity }}%</text>
                  </view>
                  <view class="attribute-item">
                    <text class="attribute-icon">🔄</text>
                    <text class="attribute-label">同步状态</text>
                    <text class="attribute-value">{{ consciousness.syncStatus }}</text>
                  </view>
                </view>
                <view class="tags-container">
                  <view 
                    v-for="(tag, tagIndex) in consciousness.tags"
                    :key="tagIndex"
                    class="tag"
                  >
                    {{ tag }}
                  </view>
                </view>
              </view>
            </view>
            
            <!-- 定价设置部分 -->
            <view class="pricing-section" v-if="currentTab === 'owned'">
              <view class="pricing-header">
                <text class="pricing-title">定价设置</text>
                <button 
                  class="edit-btn"
                  @click="togglePricingEdit(consciousness)"
                >
                  {{ consciousness.isEditing ? '保存' : '编辑' }}
                </button>
              </view>
              <view class="pricing-options">
                <view class="price-option">
                  <checkbox 
                    v-model="consciousness.pricingModel.preview.enabled"
                    :disabled="!consciousness.isEditing"
                  />
                  <text class="option-label">免费阅览</text>
                  <input 
                    v-if="consciousness.pricingModel.preview.enabled"
                    type="number"
                    v-model="consciousness.pricingModel.preview.duration"
                    :disabled="!consciousness.isEditing"
                    class="duration-input"
                    placeholder="试用时长(分钟)"
                  />
                </view>
                <view class="price-option">
                  <checkbox 
                    v-model="consciousness.pricingModel.rent.enabled"
                    :disabled="!consciousness.isEditing"
                  />
                  <text class="option-label">租赁模式</text>
                  <input 
                    v-if="consciousness.pricingModel.rent.enabled"
                    type="number"
                    v-model="consciousness.pricingModel.rent.price"
                    :disabled="!consciousness.isEditing"
                    class="price-input"
                    placeholder="每月租金"
                  />
                </view>
                <view class="price-option">
                  <checkbox 
                    v-model="consciousness.pricingModel.buy.enabled"
                    :disabled="!consciousness.isEditing"
                  />
                  <text class="option-label">买断模式</text>
                  <input 
                    v-if="consciousness.pricingModel.buy.enabled"
                    type="number"
                    v-model="consciousness.pricingModel.buy.price"
                    :disabled="!consciousness.isEditing"
                    class="price-input"
                    placeholder="买断价格"
                  />
                </view>
              </view>
            </view>

            <view class="card-footer">
              <view class="stats">
                <view class="stat-item">
                  <text class="stat-value">{{ consciousness.interactions }}</text>
                  <text class="stat-label">互动次数</text>
                </view>
                <view class="stat-item">
                  <text class="stat-value">{{ consciousness.rating }}</text>
                  <text class="stat-label">评分</text>
                </view>
                <view class="stat-item">
                  <text class="stat-value">{{ consciousness.lastSync }}</text>
                  <text class="stat-label">最近同步</text>
                </view>
              </view>
              <view class="actions">
                <button 
                  class="action-btn sync"
                  @click="syncConsciousness(consciousness)"
                >
                  同步
                </button>
                <button 
                  class="action-btn interact"
                  @click="interactWithConsciousness(consciousness)"
                >
                  对话
                </button>
                <button 
                  class="action-btn more"
                  @click="showMoreOptions(consciousness)"
                >
                  更多
                </button>
              </view>
            </view>
          </view>
        </view>
      </view>

      <!-- 数字凭证托管 -->
      <view class="credentials-section">
        <view class="section-header">
          <text class="section-title">数字凭证托管</text>
          <text class="section-desc">安全加密存储您的数字身份</text>
        </view>
        <view class="credentials-grid">
          <view 
            v-for="(platform, index) in platforms"
            :key="index"
            class="platform-card"
            :class="{ 'is-connected': platform.isConnected }"
          >
            <text class="platform-icon">{{ platform.icon }}</text>
            <view class="platform-info">
              <text class="platform-name">{{ platform.name }}</text>
              <text class="platform-status">{{ platform.isConnected ? '已连接' : '未连接' }}</text>
            </view>
            <button 
              class="platform-action"
              @click="handlePlatformAction(platform)"
            >
              {{ platform.isConnected ? '管理' : '连接' }}
            </button>
          </view>
        </view>
      </view>

      <!-- 思想资产评估 -->
      <view class="valuation-section">
        <view class="section-header">
          <text class="section-title">思想资产评估</text>
          <text class="section-desc">基于AI的数据价值分析</text>
        </view>
        <view class="valuation-cards">
          <view class="valuation-card knowledge">
            <text class="card-title">知识储备</text>
            <text class="value-score">{{ knowledgeScore }}</text>
            <view class="value-metrics">
              <view class="metric-item">
                <text class="metric-label">专业深度</text>
                <text class="metric-value">{{ metrics.professionalDepth }}</text>
              </view>
              <view class="metric-item">
                <text class="metric-label">知识广度</text>
                <text class="metric-value">{{ metrics.knowledgeBreadth }}</text>
              </view>
            </view>
          </view>
          <view class="valuation-card experience">
            <text class="card-title">经验价值</text>
            <text class="value-score">{{ experienceScore }}</text>
            <view class="value-metrics">
              <view class="metric-item">
                <text class="metric-label">实践经验</text>
                <text class="metric-value">{{ metrics.practicalExperience }}</text>
              </view>
              <view class="metric-item">
                <text class="metric-label">创新能力</text>
                <text class="metric-value">{{ metrics.innovationCapability }}</text>
              </view>
            </view>
          </view>
          <view class="valuation-card potential">
            <text class="card-title">发展潜力</text>
            <text class="value-score">{{ potentialScore }}</text>
            <view class="value-metrics">
              <view class="metric-item">
                <text class="metric-label">成长趋势</text>
                <text class="metric-value">{{ metrics.growthTrend }}</text>
              </view>
              <view class="metric-item">
                <text class="metric-label">市场价值</text>
                <text class="metric-value">{{ metrics.marketValue }}</text>
              </view>
            </view>
          </view>
        </view>
      </view>

      <!-- 意识体市场 -->
      <view class="market-section">
        <view class="section-header">
          <text class="section-title">意识体市场</text>
          <text class="section-desc">发现和获取优质意识资产</text>
        </view>
        <view class="market-filters">
          <input 
            type="text" 
            class="search-input"
            v-model="searchQuery"
            placeholder="搜索意识体..."
          >
          <select v-model="selectedCategory" class="category-select">
            <option value="">全部类别</option>
            <option 
              v-for="category in categories"
              :key="category.value"
              :value="category.value"
            >
              {{ category.label }}
            </option>
          </select>
        </view>
        <view class="consciousness-grid">
          <view 
            v-for="(item, index) in filteredMarketItems"
            :key="index"
            class="consciousness-card"
          >
            <view class="card-header">
              <text class="consciousness-name">{{ item.name }}</text>
              <text class="consciousness-price">{{ formatPrice(item.price) }}</text>
            </view>
            <text class="consciousness-desc">{{ item.description }}</text>
            <view class="consciousness-stats">
              <view class="stat-item">
                <text class="stat-label">评分</text>
                <text class="stat-value">{{ item.rating }}/10</text>
              </view>
              <view class="stat-item">
                <text class="stat-label">类型</text>
                <text class="stat-value">{{ item.type }}</text>
              </view>
            </view>
            <view class="card-actions">
              <button 
                class="action-btn preview"
                @click="previewConsciousness(item)"
              >
                预览
              </button>
              <button 
                class="action-btn"
                :class="item.purchaseType"
                @click="handlePurchase(item)"
              >
                {{ item.purchaseType === 'buy' ? '购买' : '租赁' }}
              </button>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  name: 'DigitalAssets',
  data() {
    return {
      totalAssetValue: '¥128,500',
      monthlyRevenue: '¥15,800',
      myConsciousness: [
        {
          name: '专业技术意识体',
          description: '包含多年软件开发和系统架构设计经验',
          completeness: 95,
          views: 1280,
          revenue: 8500,
          status: 'published',
          isEditing: false,
          pricingModel: {
            preview: {
              enabled: true,
              duration: 30
            },
            rent: {
              enabled: true,
              price: 1200
            },
            buy: {
              enabled: true,
              price: 25000
            }
          }
        },
        {
          name: '创意设计意识体',
          description: '设计思维与艺术创作能力的结晶',
          completeness: 88,
          views: 860,
          revenue: 4200,
          status: 'draft',
          isEditing: false,
          pricingModel: {
            preview: {
              enabled: true,
              duration: 15
            },
            rent: {
              enabled: true,
              price: 800
            },
            buy: {
              enabled: false,
              price: 0
            }
          }
        }
      ],
      searchQuery: '',
      selectedCategory: '',
      knowledgeScore: 8.7,
      experienceScore: 9.2,
      potentialScore: 8.9,
      metrics: {
        professionalDepth: '专家级',
        knowledgeBreadth: '广泛',
        practicalExperience: '丰富',
        innovationCapability: '较强',
        growthTrend: '上升',
        marketValue: '高'
      },
      platforms: [
        {
          icon: '🔑',
          name: 'DreamID',
          isConnected: true
        },
        {
          icon: '💼',
          name: '专业认证',
          isConnected: true
        },
        {
          icon: '🌐',
          name: '社交账号',
          isConnected: false
        },
        {
          icon: '📱',
          name: '数字身份',
          isConnected: true
        }
      ],
      categories: [
        { label: '专业技能', value: 'professional' },
        { label: '创意艺术', value: 'creative' },
        { label: '科研学术', value: 'academic' },
        { label: '商业洞见', value: 'business' }
      ],
      marketItems: [
        {
          name: '量子物理学者意识体',
          price: 25000,
          description: '顶尖物理学家的专业知识与研究经验',
          rating: 9.5,
          type: '专业技能',
          purchaseType: 'buy'
        },
        {
          name: '创意设计师思维模型',
          price: 800,
          description: '富有创造力的设计思维与美学鉴赏',
          rating: 8.8,
          type: '创意艺术',
          purchaseType: 'rent'
        },
        {
          name: '投资分析师决策系统',
          price: 15000,
          description: '专业的市场分析与投资决策能力',
          rating: 9.2,
          type: '商业洞见',
          purchaseType: 'buy'
        }
      ],
      currentTab: 'owned',
      ownedConsciousness: [
        {
          id: 1,
          name: '技术专家意识体',
          type: '专业技能',
          description: '专注于软件架构设计和系统优化的技术专家意识体，具备丰富的项目经验和问题解决能力。',
          avatar: '/static/avatar.jpg',
          completeness: 95,
          activity: 88,
          syncStatus: '已同步',
          status: 'active',
          tags: ['技术架构', '系统设计', '性能优化', 'AI开发'],
          interactions: 1280,
          rating: 4.9,
          lastSync: '10分钟前',
          isEditing: false,
          pricingModel: {
            preview: {
              enabled: true,
              duration: 30
            },
            rent: {
              enabled: true,
              price: 1200
            },
            buy: {
              enabled: true,
              price: 25000
            }
          }
        },
        {
          id: 2,
          name: '创意设计意识体',
          type: '创意艺术',
          description: '融合艺术创作与设计思维的创意意识体，善于视觉设计和用户体验优化。',
          avatar: '/static/avatar.jpg',
          completeness: 92,
          activity: 85,
          syncStatus: '同步中',
          status: 'syncing',
          tags: ['UI设计', 'UX设计', '创意思维', '艺术创作'],
          interactions: 960,
          rating: 4.8,
          lastSync: '1小时前',
          isEditing: false,
          pricingModel: {
            preview: {
              enabled: true,
              duration: 15
            },
            rent: {
              enabled: true,
              price: 800
            },
            buy: {
              enabled: false,
              price: 0
            }
          }
        }
      ],
      purchasedConsciousness: [
        {
          id: 3,
          name: '量子物理学者',
          type: '科研学术',
          description: '专注于量子力学和粒子物理研究的学者意识体，具备深厚的理论基础和研究经验。',
          avatar: '/static/avatar.jpg',
          completeness: 98,
          activity: 92,
          syncStatus: '已同步',
          status: 'active',
          tags: ['量子力学', '粒子物理', '理论研究', '学术创新'],
          interactions: 2100,
          rating: 4.95,
          lastSync: '5分钟前'
        }
      ]
    }
  },
  computed: {
    filteredMarketItems() {
      return this.marketItems.filter(item => {
        const matchesSearch = item.name.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
                            item.description.toLowerCase().includes(this.searchQuery.toLowerCase())
        const matchesCategory = !this.selectedCategory || item.type === this.selectedCategory
        return matchesSearch && matchesCategory
      })
    },
    currentConsciousness() {
      return this.currentTab === 'owned' ? this.ownedConsciousness : this.purchasedConsciousness
    }
  },
  methods: {
    getStatusText(status) {
      const statusMap = {
        published: '已发布',
        draft: '草稿',
        reviewing: '审核中'
      }
      return statusMap[status] || status
    },
    togglePricingEdit(consciousness) {
      consciousness.isEditing = !consciousness.isEditing
    },
    togglePublish(consciousness) {
      const asset = this.myConsciousness.find(a => a.name === consciousness.name)
      if (asset.status === 'published') {
        asset.status = 'draft'
      } else {
        // 这里可以添加发布前的验证逻辑
        asset.status = 'published'
      }
    },
    previewAsset(asset) {
      // 预览意识体资产
    },
    handlePlatformAction(platform) {
      // 处理平台连接/管理逻辑
    },
    formatPrice(price) {
      return `¥${price.toLocaleString()}`
    },
    previewConsciousness(item) {
      // 预览意识体逻辑
    },
    handlePurchase(item) {
      // 处理购买/租赁逻辑
    },
    syncConsciousness(consciousness) {
      // 同步意识体数据
      consciousness.syncStatus = '同步中'
      consciousness.status = 'syncing'
      // 模拟同步过程
      setTimeout(() => {
        consciousness.syncStatus = '已同步'
        consciousness.status = 'active'
        consciousness.lastSync = '刚刚'
      }, 2000)
    },
    interactWithConsciousness(consciousness) {
      // 打开对话界面
    },
    showMoreOptions(consciousness) {
      // 显示更多操作选项
    }
  }
}
</script>

<style lang="scss">
.digital-assets {
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
    
    .asset-stats {
      display: flex;
      gap: 20px;
      
      .stat-item {
        text-align: right;
        
        .stat-value {
          font-size: 24px;
          font-weight: bold;
          color: #1890ff;
          display: block;
        }
        
        .stat-label {
          font-size: 14px;
          color: rgba(255, 255, 255, 0.6);
        }
      }
    }
  }
  
  .assets-container {
    display: grid;
    gap: 30px;
    
    .section-header {
      margin-bottom: 20px;
      
      .section-title {
        font-size: 18px;
        font-weight: 500;
        margin-bottom: 4px;
        display: block;
      }
      
      .section-desc {
        font-size: 14px;
        color: rgba(255, 255, 255, 0.6);
      }
    }
    
    .consciousness-dashboard {
      margin-bottom: 30px;
      
      .dashboard-tabs {
        display: flex;
        gap: 16px;
        margin-top: 12px;
        
        .tab-btn {
          padding: 8px 20px;
          border-radius: 8px;
          font-size: 14px;
          background: transparent;
          border: 1px solid rgba(255, 255, 255, 0.2);
          color: rgba(255, 255, 255, 0.8);
          cursor: pointer;
          transition: all 0.3s;
          
          &:hover {
            background: rgba(255, 255, 255, 0.05);
          }
          
          &.active {
            background: #1890ff;
            border-color: #1890ff;
            color: #fff;
          }
        }
      }
      
      .consciousness-cards {
        display: grid;
        gap: 24px;
        margin-top: 24px;
        
        .consciousness-detail-card {
          background: rgba(255, 255, 255, 0.05);
          border-radius: 16px;
          padding: 24px;
          
          .card-main {
            display: flex;
            gap: 24px;
            margin-bottom: 24px;
            
            .consciousness-avatar {
              position: relative;
              
              .avatar-img {
                width: 120px;
                height: 120px;
                border-radius: 16px;
                object-fit: cover;
              }
              
              .status-indicator {
                position: absolute;
                bottom: 8px;
                right: 8px;
                width: 12px;
                height: 12px;
                border-radius: 50%;
                border: 2px solid rgba(0, 0, 0, 0.2);
                
                &.active {
                  background: #52c41a;
                }
                
                &.syncing {
                  background: #faad14;
                  animation: pulse 1.5s infinite;
                }
                
                &.inactive {
                  background: #ff4d4f;
                }
              }
            }
            
            .consciousness-info {
              flex: 1;
              
              .info-header {
                display: flex;
                justify-content: space-between;
                align-items: center;
                margin-bottom: 12px;
                
                .consciousness-name {
                  font-size: 20px;
                  font-weight: 500;
                }
                
                .header-actions {
                  display: flex;
                  align-items: center;
                  gap: 12px;
                  
                  .action-btn.publish {
                    padding: 4px 12px;
                    border-radius: 6px;
                    font-size: 14px;
                    background: #1890ff;
                    color: #fff;
                    border: none;
                    cursor: pointer;
                    
                    &:hover {
                      background: #40a9ff;
                    }
                    
                    &.published {
                      background: #ff4d4f;
                      
                      &:hover {
                        background: #ff7875;
                      }
                    }
                  }
                }
              }
              
              .consciousness-desc {
                font-size: 14px;
                color: rgba(255, 255, 255, 0.6);
                line-height: 1.5;
                margin-bottom: 16px;
              }
              
              .consciousness-attributes {
                display: flex;
                gap: 24px;
                margin-bottom: 16px;
                
                .attribute-item {
                  display: flex;
                  align-items: center;
                  gap: 8px;
                  
                  .attribute-icon {
                    font-size: 16px;
                  }
                  
                  .attribute-label {
                    font-size: 14px;
                    color: rgba(255, 255, 255, 0.4);
                  }
                  
                  .attribute-value {
                    font-size: 14px;
                    color: #1890ff;
                    font-weight: 500;
                  }
                }
              }
              
              .tags-container {
                display: flex;
                flex-wrap: wrap;
                gap: 8px;
                
                .tag {
                  padding: 4px 12px;
                  border-radius: 12px;
                  font-size: 12px;
                  background: rgba(255, 255, 255, 0.05);
                  color: rgba(255, 255, 255, 0.8);
                }
              }
            }
          }
          
          .pricing-section {
            background: rgba(255, 255, 255, 0.02);
            border-radius: 12px;
            padding: 16px;
            margin: 16px 0;
            
            .pricing-header {
              display: flex;
              justify-content: space-between;
              align-items: center;
              margin-bottom: 16px;
              
              .pricing-title {
                font-size: 16px;
                font-weight: 500;
              }
              
              .edit-btn {
                padding: 4px 12px;
                border-radius: 6px;
                font-size: 14px;
                background: transparent;
                border: 1px solid rgba(255, 255, 255, 0.2);
                color: #fff;
                cursor: pointer;
                
                &:hover {
                  background: rgba(255, 255, 255, 0.05);
                }
              }
            }
            
            .pricing-options {
              display: grid;
              gap: 16px;
              
              .price-option {
                display: flex;
                align-items: center;
                gap: 12px;
                
                .option-label {
                  font-size: 14px;
                  color: rgba(255, 255, 255, 0.8);
                  min-width: 80px;
                }
                
                .price-input,
                .duration-input {
                  background: rgba(255, 255, 255, 0.1);
                  border: 1px solid rgba(255, 255, 255, 0.2);
                  border-radius: 6px;
                  padding: 6px 12px;
                  color: #fff;
                  font-size: 14px;
                  width: 120px;
                  
                  &:disabled {
                    opacity: 0.5;
                    cursor: not-allowed;
                  }
                  
                  &:focus {
                    border-color: #1890ff;
                    outline: none;
                  }
                }
              }
            }
          }

          .card-footer {
            border-top: 1px solid rgba(255, 255, 255, 0.1);
            padding-top: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            
            .stats {
              display: flex;
              gap: 24px;
              
              .stat-item {
                text-align: center;
                
                .stat-value {
                  font-size: 16px;
                  font-weight: 500;
                  color: rgba(255, 255, 255, 0.9);
                  display: block;
                }
                
                .stat-label {
                  font-size: 12px;
                  color: rgba(255, 255, 255, 0.4);
                  margin-top: 4px;
                }
              }
            }
            
            .actions {
              display: flex;
              gap: 12px;
              
              .action-btn {
                padding: 8px 16px;
                border-radius: 8px;
                font-size: 14px;
                border: none;
                cursor: pointer;
                
                &.sync {
                  background: rgba(255, 255, 255, 0.1);
                  color: #fff;
                  
                  &:hover {
                    background: rgba(255, 255, 255, 0.15);
                  }
                }
                
                &.interact {
                  background: #1890ff;
                  color: #fff;
                  
                  &:hover {
                    background: #40a9ff;
                  }
                }
                
                &.more {
                  background: transparent;
                  border: 1px solid rgba(255, 255, 255, 0.2);
                  color: rgba(255, 255, 255, 0.8);
                  
                  &:hover {
                    background: rgba(255, 255, 255, 0.05);
                  }
                }
              }
            }
          }
        }
      }
    }
    
    .credentials-section {
      .credentials-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
        gap: 20px;
        
        .platform-card {
          background: rgba(255, 255, 255, 0.05);
          border-radius: 16px;
          padding: 20px;
          display: flex;
          align-items: center;
          gap: 12px;
          
          &.is-connected {
            border: 1px solid rgba(24, 144, 255, 0.3);
          }
          
          .platform-icon {
            font-size: 24px;
          }
          
          .platform-info {
            flex: 1;
            
            .platform-name {
              font-size: 16px;
              font-weight: 500;
              margin-bottom: 4px;
              display: block;
            }
            
            .platform-status {
              font-size: 12px;
              color: rgba(255, 255, 255, 0.4);
            }
          }
          
          .platform-action {
            padding: 6px 12px;
            border-radius: 6px;
            font-size: 14px;
            background: #1890ff;
            border: none;
            color: #fff;
            cursor: pointer;
            
            &:hover {
              background: #40a9ff;
            }
          }
        }
      }
    }
    
    .valuation-section {
      .valuation-cards {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
        gap: 20px;
        
        .valuation-card {
          background: rgba(255, 255, 255, 0.05);
          border-radius: 16px;
          padding: 20px;
          
          .card-title {
            font-size: 16px;
            font-weight: 500;
            margin-bottom: 12px;
            display: block;
          }
          
          .value-score {
            font-size: 36px;
            font-weight: bold;
            color: #1890ff;
            margin-bottom: 20px;
            display: block;
          }
          
          .value-metrics {
            display: grid;
            gap: 12px;
            
            .metric-item {
              display: flex;
              justify-content: space-between;
              align-items: center;
              
              .metric-label {
                font-size: 14px;
                color: rgba(255, 255, 255, 0.6);
              }
              
              .metric-value {
                font-size: 14px;
                color: rgba(255, 255, 255, 0.8);
              }
            }
          }
          
          &.knowledge {
            background: linear-gradient(135deg, rgba(24, 144, 255, 0.1) 0%, rgba(255, 255, 255, 0.05) 100%);
          }
          
          &.experience {
            background: linear-gradient(135deg, rgba(250, 84, 28, 0.1) 0%, rgba(255, 255, 255, 0.05) 100%);
          }
          
          &.potential {
            background: linear-gradient(135deg, rgba(82, 196, 26, 0.1) 0%, rgba(255, 255, 255, 0.05) 100%);
          }
        }
      }
    }
    
    .market-section {
      .market-filters {
        display: flex;
        gap: 16px;
        margin-bottom: 20px;
        
        .search-input,
        .category-select {
          background: rgba(255, 255, 255, 0.1);
          border: 1px solid rgba(255, 255, 255, 0.2);
          border-radius: 8px;
          padding: 8px 12px;
          color: #fff;
          font-size: 14px;
          
          &:focus {
            border-color: #1890ff;
            outline: none;
          }
        }
        
        .search-input {
          flex: 1;
        }
        
        .category-select {
          width: 200px;
          
          option {
            background: #1f1f1f;
          }
        }
      }
      
      .consciousness-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
        gap: 20px;
        
        .consciousness-card {
          background: rgba(255, 255, 255, 0.05);
          border-radius: 16px;
          padding: 20px;
          
          .card-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
            
            .consciousness-name {
              font-size: 16px;
              font-weight: 500;
            }
            
            .consciousness-price {
              font-size: 16px;
              color: #1890ff;
            }
          }
          
          .consciousness-desc {
            font-size: 14px;
            color: rgba(255, 255, 255, 0.6);
            margin-bottom: 16px;
            line-height: 1.4;
          }
          
          .consciousness-stats {
            display: flex;
            gap: 16px;
            margin-bottom: 16px;
            
            .stat-item {
              .stat-label {
                font-size: 12px;
                color: rgba(255, 255, 255, 0.4);
                margin-bottom: 4px;
                display: block;
              }
              
              .stat-value {
                font-size: 14px;
                color: rgba(255, 255, 255, 0.8);
              }
            }
          }
          
          .card-actions {
            display: flex;
            gap: 12px;
            
            .action-btn {
              flex: 1;
              padding: 8px;
              border-radius: 8px;
              font-size: 14px;
              border: none;
              cursor: pointer;
              
              &.preview {
                background: rgba(255, 255, 255, 0.1);
                color: #fff;
                
                &:hover {
                  background: rgba(255, 255, 255, 0.15);
                }
              }
              
              &.buy {
                background: #1890ff;
                color: #fff;
                
                &:hover {
                  background: #40a9ff;
                }
              }
              
              &.rent {
                background: #13c2c2;
                color: #fff;
                
                &:hover {
                  background: #36cfc9;
                }
              }
            }
          }
        }
      }
    }
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.2);
    opacity: 0.8;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
</style> 