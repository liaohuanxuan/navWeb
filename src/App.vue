<template>
  <div class="app">
    <!-- 头部 -->
    <header class="header">
      <div class="container">
        <div class="header-content">
          <div class="logo">
            <i class="fas fa-compass"></i>
            <h1>个人工具导航</h1>
          </div>
          <div class="header-actions">
            <div class="search-box">
              <i class="fas fa-search"></i>
              <input 
                type="text" 
                v-model="searchQuery" 
                placeholder="搜索工具或标签..."
                @input="handleSearch"
              >
            </div>
            <div class="action-buttons">
              <button class="import-btn" @click="showImportModal = true">
                <i class="fas fa-plus"></i>
                导入网站
              </button>
              <button class="manage-btn" @click="showManageModal = true">
                <i class="fas fa-cog"></i>
                数据管理
              </button>
            </div>
          </div>
        </div>
      </div>
    </header>

    <!-- 分类筛选 -->
    <div class="category-filter">
      <div class="container">
        <div class="categories">
          <button 
            class="category-btn"
            :class="{ active: selectedCategory === '' }"
            @click="selectCategory('')"
          >
            <i class="fas fa-th"></i>
            全部
          </button>
          <button 
            v-for="category in categories" 
            :key="category.id"
            class="category-btn"
            :class="{ active: selectedCategory === category.id }"
            @click="selectCategory(category.id)"
          >
            <i :class="category.icon"></i>
            {{ category.name }}
          </button>
        </div>
      </div>
    </div>

    <!-- 主内容区 -->
    <main class="main-content">
      <div class="container">
        <div v-if="filteredCategories.length === 0" class="no-results">
          <i class="fas fa-search"></i>
          <p>没有找到相关内容</p>
        </div>
        
        <div v-for="category in filteredCategories" :key="category.id" class="category-section">
          <div class="category-header">
            <i :class="category.icon"></i>
            <h2>{{ category.name }}</h2>
            <span class="count">{{ category.sites.length }}</span>
          </div>
          
          <div class="sites-grid">
            <div 
              v-for="site in category.sites" 
              :key="site.id"
              class="site-card-wrapper"
              :class="{ 'is-custom': site.isCustom }"
            >
              <div class="site-card">
                <!-- 自定义网站的操作按钮 -->
                <div v-if="site.isCustom" class="site-actions">
                  <button 
                    class="action-btn edit-btn" 
                    @click.stop="editSite(site, category.id)"
                  >
                    编辑
                  </button>
                  <button 
                    class="action-btn delete-btn" 
                    @click.stop="confirmDeleteSite(site)"
                  >
                    删除
                  </button>
                </div>
                
                <a 
                  :href="site.url"
                  target="_blank"
                  rel="noopener noreferrer"
                  class="site-card-link"
                >
                  <div class="site-icon">
                    <i :class="site.icon || 'fas fa-link'"></i>
                  </div>
                  <div class="site-info">
                    <h3 class="site-title">{{ site.name }}</h3>
                    <p class="site-description">{{ site.description }}</p>
                    <div class="site-tags" v-if="site.tags && site.tags.length">
                      <span v-for="tag in site.tags" :key="tag" class="tag">{{ tag }}</span>
                    </div>
                  </div>
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- 页脚 -->
    <footer class="footer">
      <div class="container">
        <p>&copy; 2025 个人工具导航 | 让工作更高效</p>
      </div>
    </footer>

    <!-- 数据管理弹窗 -->
    <div v-if="showManageModal" class="modal-overlay" @click="closeManageModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>
            <i class="fas fa-database"></i>
            数据管理
          </h3>
          <button class="close-btn" @click="closeManageModal">
            <i class="fas fa-times"></i>
          </button>
        </div>
        
        <div class="modal-body">
          <div class="manage-section">
            <h4><i class="fas fa-info-circle"></i> 数据统计</h4>
            <div class="stats-grid">
              <div class="stat-item">
                <div class="stat-label">默认网站</div>
                <div class="stat-value">{{ defaultSitesCount }}</div>
              </div>
              <div class="stat-item">
                <div class="stat-label">自定义网站</div>
                <div class="stat-value custom">{{ customSitesCount }}</div>
              </div>
              <div class="stat-item">
                <div class="stat-label">总计</div>
                <div class="stat-value total">{{ totalSitesCount }}</div>
              </div>
            </div>
          </div>

          <div class="manage-section">
            <h4><i class="fas fa-download"></i> 导出配置</h4>
            <p class="section-desc">将所有数据（默认+自定义）导出为 JSON 文件，可用于备份或迁移到其他设备。</p>
            <button class="action-btn export-btn" @click="exportData">
              <i class="fas fa-file-download"></i>
              导出配置文件
            </button>
          </div>

          <div class="manage-section">
            <h4><i class="fas fa-upload"></i> 导入配置</h4>
            <p class="section-desc">上传之前导出的配置文件，恢复所有自定义数据。</p>
            <input 
              type="file" 
              ref="fileInput" 
              accept=".json"
              style="display: none"
              @change="importData"
            >
            <button class="action-btn import-btn" @click="$refs.fileInput.click()">
              <i class="fas fa-file-upload"></i>
              选择配置文件
            </button>
          </div>

          <div class="manage-section danger-section">
            <h4><i class="fas fa-trash-alt"></i> 清除数据</h4>
            <p class="section-desc">清除所有自定义添加的网站，恢复到默认状态。此操作不可恢复！</p>
            <button class="action-btn danger-btn" @click="confirmClearData">
              <i class="fas fa-exclamation-triangle"></i>
              清除所有自定义数据
            </button>
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-cancel" @click="closeManageModal">关闭</button>
        </div>
      </div>
    </div>

    <!-- 导入网站弹窗 -->
    <div v-if="showImportModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>
            <i :class="isEditMode ? 'fas fa-edit' : 'fas fa-plus-circle'"></i>
            {{ isEditMode ? '编辑网站' : '导入网站' }}
          </h3>
          <button class="close-btn" @click="closeModal">
            <i class="fas fa-times"></i>
          </button>
        </div>
        
        <div class="modal-body">
          <div class="form-group">
            <label>网站URL</label>
            <input 
              type="url" 
              v-model="importUrl" 
              placeholder="请输入网站URL，例如：https://github.com"
              @input="clearError"
              class="url-input"
            >
            <button 
              class="fetch-btn" 
              @click="fetchSiteInfo"
              :disabled="isFetching || !importUrl.trim()"
            >
              <i class="fas fa-sync-alt" :class="{ spinning: isFetching }"></i>
              {{ isFetching ? '获取中...' : '自动获取信息' }}
            </button>
          </div>

          <div v-if="errorMessage" class="error-message">
            <i class="fas fa-exclamation-circle"></i>
            {{ errorMessage }}
          </div>

          <div v-if="siteInfo.name" class="site-preview">
            <h4>网站预览</h4>
            <div class="preview-card">
              <div class="form-group">
                <label>网站名称</label>
                <input type="text" v-model="siteInfo.name" placeholder="网站名称">
              </div>
              
              <div class="form-group">
                <label>网站描述</label>
                <textarea v-model="siteInfo.description" placeholder="网站描述" rows="3"></textarea>
              </div>

              <div class="form-group">
                <label>选择分类</label>
                <select v-model="siteInfo.categoryId">
                  <option value="">请选择分类</option>
                  <option v-for="cat in categories" :key="cat.id" :value="cat.id">
                    {{ cat.name }}
                  </option>
                </select>
              </div>

              <div class="form-group">
                <label>图标类名（Font Awesome）</label>
                <input type="text" v-model="siteInfo.icon" placeholder="例如：fas fa-code">
                <small>访问 <a href="https://fontawesome.com/icons" target="_blank">Font Awesome</a> 查找图标</small>
              </div>

              <div class="form-group">
                <label>标签（用逗号分隔）</label>
                <input type="text" v-model="siteInfo.tagsInput" placeholder="例如：开发,工具,效率">
              </div>
            </div>
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-cancel" @click="closeModal">取消</button>
          <button 
            class="btn-confirm" 
            @click="addSite"
            :disabled="!canAddSite"
          >
            <i class="fas fa-check"></i>
            {{ isEditMode ? '保存修改' : '添加到列表' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const searchQuery = ref('')
const selectedCategory = ref('')
const categories = ref([])
const allData = ref([])
const defaultData = ref([]) // 存储默认数据

// 导入功能相关状态
const showImportModal = ref(false)
const showManageModal = ref(false)
const importUrl = ref('')
const isFetching = ref(false)
const errorMessage = ref('')
const fileInput = ref(null)
const isEditMode = ref(false)
const editingSiteId = ref('')
const siteInfo = ref({
  name: '',
  description: '',
  url: '',
  icon: 'fas fa-link',
  categoryId: '',
  tagsInput: ''
})

// 数据统计
const defaultSitesCount = computed(() => {
  let count = 0
  defaultData.value.forEach(cat => {
    count += cat.sites.length
  })
  return count
})

const customSitesCount = computed(() => {
  return loadCustomData().length
})

const totalSitesCount = computed(() => {
  let count = 0
  allData.value.forEach(cat => {
    count += cat.sites.length
  })
  return count
})

// localStorage 键名
const STORAGE_KEY = 'navWeb_customSites'

// 加载默认数据
const loadDefaultData = async () => {
  try {
    const response = await fetch('./data/sites.json')
    const data = await response.json()
    defaultData.value = JSON.parse(JSON.stringify(data.categories)) // 深拷贝
    return data.categories
  } catch (error) {
    console.error('加载默认数据失败:', error)
    return []
  }
}

// 从 localStorage 加载自定义数据
const loadCustomData = () => {
  try {
    const stored = localStorage.getItem(STORAGE_KEY)
    return stored ? JSON.parse(stored) : []
  } catch (error) {
    console.error('加载自定义数据失败:', error)
    return []
  }
}

// 保存自定义数据到 localStorage
const saveCustomData = (customSites) => {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(customSites))
  } catch (error) {
    console.error('保存数据失败:', error)
    alert('保存失败：' + error.message)
  }
}

// 合并默认数据和自定义数据
const mergeData = (defaultCategories, customSites) => {
  const merged = JSON.parse(JSON.stringify(defaultCategories)) // 深拷贝默认数据
  
  // 将自定义网站添加到对应分类
  customSites.forEach(site => {
    const category = merged.find(cat => cat.id === site.categoryId)
    if (category) {
      // 标记为自定义数据
      category.sites.push({ ...site, isCustom: true })
    }
  })
  
  return merged
}

// 初始化数据
onMounted(async () => {
  const defaults = await loadDefaultData()
  const customs = loadCustomData()
  allData.value = mergeData(defaults, customs)
  categories.value = allData.value
})

// 选择分类
const selectCategory = (categoryId) => {
  selectedCategory.value = categoryId
  searchQuery.value = ''
}

// 搜索处理
const handleSearch = () => {
  selectedCategory.value = ''
}

// 过滤后的分类
const filteredCategories = computed(() => {
  let result = allData.value

  // 按分类筛选
  if (selectedCategory.value) {
    result = result.filter(cat => cat.id === selectedCategory.value)
  }

  // 按搜索关键词筛选
  if (searchQuery.value.trim()) {
    const query = searchQuery.value.toLowerCase().trim()
    result = result.map(category => {
      const filteredSites = category.sites.filter(site => {
        const matchName = site.name.toLowerCase().includes(query)
        const matchDescription = site.description.toLowerCase().includes(query)
        const matchTags = site.tags && site.tags.some(tag => tag.toLowerCase().includes(query))
        return matchName || matchDescription || matchTags
      })
      return {
        ...category,
        sites: filteredSites
      }
    }).filter(category => category.sites.length > 0)
  }

  return result
})

// 关闭导入弹窗
const closeModal = () => {
  showImportModal.value = false
  isEditMode.value = false
  editingSiteId.value = ''
  resetImportForm()
}

// 关闭管理弹窗
const closeManageModal = () => {
  showManageModal.value = false
}

// 导出数据
const exportData = () => {
  const customSites = loadCustomData()
  const exportObj = {
    version: '1.0',
    exportTime: new Date().toISOString(),
    customSites: customSites
  }
  
  const dataStr = JSON.stringify(exportObj, null, 2)
  const blob = new Blob([dataStr], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = `navWeb-config-${Date.now()}.json`
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
  
  alert(`成功导出配置文件！\n\n包含 ${customSites.length} 个自定义网站。`)
}

// 导入数据
const importData = (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const content = e.target.result
      const importObj = JSON.parse(content)
      
      // 验证数据格式
      if (!importObj.customSites || !Array.isArray(importObj.customSites)) {
        throw new Error('配置文件格式不正确')
      }
      
      // 保存导入的数据
      saveCustomData(importObj.customSites)
      
      // 重新加载数据
      allData.value = mergeData(defaultData.value, importObj.customSites)
      categories.value = allData.value
      
      alert(`成功导入配置！\n\n导入了 ${importObj.customSites.length} 个自定义网站。`)
      closeManageModal()
    } catch (error) {
      console.error('导入失败:', error)
      alert('导入失败：' + error.message)
    }
  }
  reader.readAsText(file)
  
  // 清空 input，允许重复选择同一文件
  event.target.value = ''
}

// 确认清除数据
const confirmClearData = () => {
  const customCount = customSitesCount.value
  if (customCount === 0) {
    alert('当前没有自定义数据。')
    return
  }
  
  const confirmed = confirm(
    `确定要清除所有自定义数据吗？\n\n` +
    `这将删除 ${customCount} 个自定义网站。\n` +
    `此操作不可恢复！\n\n` +
    `建议先导出配置进行备份。`
  )
  
  if (confirmed) {
    clearCustomData()
  }
}

// 清除自定义数据
const clearCustomData = () => {
  try {
    localStorage.removeItem(STORAGE_KEY)
    allData.value = mergeData(defaultData.value, [])
    categories.value = allData.value
    alert('已清除所有自定义数据，恢复到默认状态。')
    closeManageModal()
  } catch (error) {
    console.error('清除数据失败:', error)
    alert('清除失败：' + error.message)
  }
}

// 重置导入表单
const resetImportForm = () => {
  importUrl.value = ''
  errorMessage.value = ''
  siteInfo.value = {
    name: '',
    description: '',
    url: '',
    icon: 'fas fa-link',
    categoryId: '',
    tagsInput: ''
  }
}

// 清除错误信息
const clearError = () => {
  errorMessage.value = ''
}

// 智能识别网站信息（基于域名）
const getSmartSiteInfo = (url) => {
  const hostname = url.hostname.toLowerCase()
  const domain = hostname.replace('www.', '')
  
  // 常见网站数据库
  const knownSites = {
    // 开发工具
    'github.com': { name: 'GitHub', desc: '全球最大的代码托管平台', icon: 'fab fa-github', category: 'dev-tools', tags: '代码托管,开源,协作' },
    'gitlab.com': { name: 'GitLab', desc: 'DevOps 平台', icon: 'fab fa-gitlab', category: 'dev-tools', tags: '代码托管,CI/CD' },
    'stackoverflow.com': { name: 'Stack Overflow', desc: '程序员问答社区', icon: 'fab fa-stack-overflow', category: 'dev-tools', tags: '问答,社区' },
    'npmjs.com': { name: 'NPM', desc: 'Node.js 包管理器', icon: 'fab fa-npm', category: 'dev-tools', tags: 'JavaScript,包管理' },
    'code.visualstudio.com': { name: 'VS Code', desc: '微软开发的代码编辑器', icon: 'fas fa-code', category: 'dev-tools', tags: '编辑器,IDE' },
    
    // 设计资源
    'figma.com': { name: 'Figma', desc: '在线协作设计工具', icon: 'fas fa-pen-nib', category: 'design', tags: '设计,原型,协作' },
    'dribbble.com': { name: 'Dribbble', desc: '设计师作品展示平台', icon: 'fab fa-dribbble', category: 'design', tags: '设计,灵感' },
    'behance.net': { name: 'Behance', desc: 'Adobe 设计作品平台', icon: 'fab fa-behance', category: 'design', tags: '设计,作品集' },
    'unsplash.com': { name: 'Unsplash', desc: '免费高质量图片库', icon: 'fas fa-image', category: 'design', tags: '图片,免费' },
    
    // AI工具
    'openai.com': { name: 'OpenAI', desc: 'AI 研究和部署公司', icon: 'fas fa-robot', category: 'ai-tools', tags: 'AI,ChatGPT' },
    'chat.openai.com': { name: 'ChatGPT', desc: 'OpenAI 智能对话AI', icon: 'fas fa-comments', category: 'ai-tools', tags: 'AI,对话' },
    'midjourney.com': { name: 'Midjourney', desc: 'AI 绘画工具', icon: 'fas fa-paint-brush', category: 'ai-tools', tags: 'AI,绘画' },
    
    // 效率工具
    'notion.so': { name: 'Notion', desc: '全能笔记和协作工具', icon: 'fas fa-book', category: 'productivity', tags: '笔记,协作' },
    'trello.com': { name: 'Trello', desc: '看板式项目管理工具', icon: 'fab fa-trello', category: 'productivity', tags: '项目管理,看板' },
    'feishu.cn': { name: '飞书', desc: '字节跳动企业协作平台', icon: 'fas fa-comments', category: 'productivity', tags: '协作,办公' },
    
    // 学习平台
    'developer.mozilla.org': { name: 'MDN Web Docs', desc: 'Web 技术文档', icon: 'fas fa-book-open', category: 'learning', tags: '文档,前端' },
    'freecodecamp.org': { name: 'freeCodeCamp', desc: '免费编程学习平台', icon: 'fab fa-free-code-camp', category: 'learning', tags: '编程,学习' },
  }
  
  // 查找已知网站
  if (knownSites[domain]) {
    const site = knownSites[domain]
    return {
      name: site.name,
      description: site.desc,
      icon: site.icon,
      categoryId: site.category,
      tagsInput: site.tags
    }
  }
  
  // 智能推测分类
  let categoryId = ''
  let icon = 'fas fa-link'
  let tags = []
  
  if (domain.includes('github') || domain.includes('gitlab') || domain.includes('code') || domain.includes('dev')) {
    categoryId = 'dev-tools'
    icon = 'fas fa-code'
    tags = ['开发', '工具']
  } else if (domain.includes('design') || domain.includes('figma') || domain.includes('sketch')) {
    categoryId = 'design'
    icon = 'fas fa-palette'
    tags = ['设计']
  } else if (domain.includes('ai') || domain.includes('gpt') || domain.includes('chat')) {
    categoryId = 'ai-tools'
    icon = 'fas fa-robot'
    tags = ['AI']
  } else if (domain.includes('notion') || domain.includes('docs') || domain.includes('note')) {
    categoryId = 'productivity'
    icon = 'fas fa-tasks'
    tags = ['效率', '工具']
  } else if (domain.includes('learn') || domain.includes('edu') || domain.includes('course')) {
    categoryId = 'learning'
    icon = 'fas fa-graduation-cap'
    tags = ['学习']
  } else if (domain.includes('tool') || domain.includes('util')) {
    categoryId = 'utilities'
    icon = 'fas fa-tools'
    tags = ['工具']
  }
  
  // 从域名提取名称
  const nameParts = domain.split('.')
  const name = nameParts[0].charAt(0).toUpperCase() + nameParts[0].slice(1)
  
  return {
    name: name,
    description: `${name} - ${url.hostname}`,
    icon: icon,
    categoryId: categoryId,
    tagsInput: tags.join(',')
  }
}

// 使用代理获取网站信息（带超时）
const fetchWithProxy = async (url, timeout = 8000) => {
  const controller = new AbortController()
  const timeoutId = setTimeout(() => controller.abort(), timeout)
  
  try {
    // 尝试多个代理服务，使用最快的
    const proxies = [
      `https://api.allorigins.win/raw?url=${encodeURIComponent(url)}`,
      `https://corsproxy.io/?${encodeURIComponent(url)}`
    ]
    
    const response = await Promise.race(
      proxies.map(proxy => 
        fetch(proxy, { signal: controller.signal })
          .then(res => res.ok ? res.text() : Promise.reject())
      )
    )
    
    clearTimeout(timeoutId)
    return response
  } catch (error) {
    clearTimeout(timeoutId)
    throw error
  }
}

// 获取网站信息
const fetchSiteInfo = async () => {
  if (!importUrl.value.trim()) {
    errorMessage.value = '请输入有效的URL'
    return
  }

  // 验证URL格式
  let url
  try {
    url = new URL(importUrl.value)
  } catch (e) {
    errorMessage.value = 'URL格式不正确，请输入完整的URL（包括 http:// 或 https://）'
    return
  }

  isFetching.value = true
  errorMessage.value = ''

  try {
    // 先使用智能识别快速填充基本信息
    const smartInfo = getSmartSiteInfo(url)
    siteInfo.value = {
      name: smartInfo.name,
      description: smartInfo.description,
      url: importUrl.value,
      icon: smartInfo.icon,
      categoryId: smartInfo.categoryId,
      tagsInput: smartInfo.tagsInput
    }
    
    // 如果是已知网站，直接返回，不需要抓取
    const domain = url.hostname.toLowerCase().replace('www.', '')
    const knownDomains = ['github.com', 'gitlab.com', 'stackoverflow.com', 'npmjs.com', 
                          'figma.com', 'dribbble.com', 'unsplash.com', 'notion.so', 
                          'trello.com', 'openai.com', 'chat.openai.com', 'midjourney.com']
    
    if (knownDomains.includes(domain)) {
      isFetching.value = false
      return
    }
    
    // 尝试抓取更详细的信息（后台进行，不阻塞用户）
    try {
      const html = await fetchWithProxy(importUrl.value, 8000)
      const parser = new DOMParser()
      const doc = parser.parseFromString(html, 'text/html')
      
      // 提取更准确的信息
      const title = doc.querySelector('meta[property="og:title"]')?.getAttribute('content') ||
                    doc.querySelector('title')?.textContent ||
                    smartInfo.name
      
      const description = doc.querySelector('meta[property="og:description"]')?.getAttribute('content') ||
                          doc.querySelector('meta[name="description"]')?.getAttribute('content') ||
                          smartInfo.description
      
      const keywords = doc.querySelector('meta[name="keywords"]')?.getAttribute('content') || ''
      
      // 更新信息
      siteInfo.value = {
        name: title.trim(),
        description: description.trim(),
        url: importUrl.value,
        icon: smartInfo.icon,
        categoryId: smartInfo.categoryId,
        tagsInput: keywords ? keywords.split(',').slice(0, 3).map(t => t.trim()).join(',') : smartInfo.tagsInput
      }
    } catch (fetchError) {
      // 抓取失败不影响，已经有智能识别的基本信息
      console.log('详细信息获取失败，使用智能识别信息')
    }
    
  } catch (error) {
    console.error('获取网站信息失败:', error)
    errorMessage.value = '自动识别完成，请确认信息'
  } finally {
    isFetching.value = false
  }
}

// 判断是否可以添加网站
const canAddSite = computed(() => {
  return siteInfo.value.name && 
         siteInfo.value.description && 
         siteInfo.value.categoryId &&
         siteInfo.value.url
})

// 添加或更新网站
const addSite = () => {
  if (!canAddSite.value) {
    errorMessage.value = '请填写完整信息'
    return
  }

  // 处理标签
  const tags = siteInfo.value.tagsInput
    ? siteInfo.value.tagsInput.split(',').map(tag => tag.trim()).filter(tag => tag)
    : []

  const customSites = loadCustomData()

  if (isEditMode.value) {
    // 编辑模式：更新现有网站
    const siteIndex = customSites.findIndex(s => s.id === editingSiteId.value)
    if (siteIndex !== -1) {
      customSites[siteIndex] = {
        ...customSites[siteIndex],
        name: siteInfo.value.name,
        description: siteInfo.value.description,
        url: siteInfo.value.url,
        icon: siteInfo.value.icon || 'fas fa-link',
        tags: tags,
        categoryId: siteInfo.value.categoryId
      }
      
      saveCustomData(customSites)
      allData.value = mergeData(defaultData.value, customSites)
      categories.value = allData.value
      
      alert(`成功更新 "${siteInfo.value.name}"！`)
      closeModal()
      selectCategory(siteInfo.value.categoryId)
    }
  } else {
    // 添加模式：创建新网站
    const timestamp = Date.now()
    const randomStr = Math.random().toString(36).substring(2, 9)
    const newSiteId = `custom-${timestamp}-${randomStr}`

    const newSite = {
      id: newSiteId,
      name: siteInfo.value.name,
      description: siteInfo.value.description,
      url: siteInfo.value.url,
      icon: siteInfo.value.icon || 'fas fa-link',
      tags: tags,
      categoryId: siteInfo.value.categoryId,
      isCustom: true
    }

    customSites.push(newSite)
    saveCustomData(customSites)
    allData.value = mergeData(defaultData.value, customSites)
    categories.value = allData.value

    const category = allData.value.find(cat => cat.id === siteInfo.value.categoryId)
    const categoryName = category ? category.name : '列表'
    
    alert(`成功添加 "${newSite.name}" 到 "${categoryName}" 分类！\n\n数据已保存到浏览器，刷新页面不会丢失。`)
    closeModal()
    selectCategory(siteInfo.value.categoryId)
  }
}

// 编辑网站
const editSite = (site, categoryId) => {
  isEditMode.value = true
  editingSiteId.value = site.id
  
  siteInfo.value = {
    name: site.name,
    description: site.description,
    url: site.url,
    icon: site.icon || 'fas fa-link',
    categoryId: categoryId,
    tagsInput: site.tags ? site.tags.join(',') : ''
  }
  
  showImportModal.value = true
}

// 确认删除网站
const confirmDeleteSite = (site) => {
  const confirmed = confirm(
    `确定要删除 "${site.name}" 吗？\n\n此操作不可恢复！`
  )
  
  if (confirmed) {
    deleteSite(site.id)
  }
}

// 删除网站
const deleteSite = (siteId) => {
  const customSites = loadCustomData()
  const filteredSites = customSites.filter(s => s.id !== siteId)
  
  saveCustomData(filteredSites)
  allData.value = mergeData(defaultData.value, filteredSites)
  categories.value = allData.value
  
  alert('删除成功！')
}
</script>

