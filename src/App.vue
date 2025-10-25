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
            <a 
              v-for="site in category.sites" 
              :key="site.id"
              :href="site.url"
              target="_blank"
              rel="noopener noreferrer"
              class="site-card"
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
            <i class="fas fa-plus-circle"></i>
            导入网站
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
            添加到列表
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

// 获取网站信息
const fetchSiteInfo = async () => {
  if (!importUrl.value.trim()) {
    errorMessage.value = '请输入有效的URL'
    return
  }

  // 验证URL格式
  try {
    new URL(importUrl.value)
  } catch (e) {
    errorMessage.value = 'URL格式不正确，请输入完整的URL（包括 http:// 或 https://）'
    return
  }

  isFetching.value = true
  errorMessage.value = ''

  try {
    // 使用代理服务获取网站信息（避免CORS问题）
    // 这里使用allorigins.win作为代理服务
    const proxyUrl = `https://api.allorigins.win/get?url=${encodeURIComponent(importUrl.value)}`
    const response = await fetch(proxyUrl)
    const data = await response.json()
    
    if (!data.contents) {
      throw new Error('无法获取网站内容')
    }

    // 解析HTML内容
    const parser = new DOMParser()
    const doc = parser.parseFromString(data.contents, 'text/html')
    
    // 提取网站标题
    let title = doc.querySelector('title')?.textContent || ''
    const ogTitle = doc.querySelector('meta[property="og:title"]')?.getAttribute('content')
    if (ogTitle) title = ogTitle
    
    // 提取网站描述
    let description = doc.querySelector('meta[name="description"]')?.getAttribute('content') || ''
    const ogDescription = doc.querySelector('meta[property="og:description"]')?.getAttribute('content')
    if (ogDescription) description = ogDescription
    
    // 提取关键词作为标签
    const keywords = doc.querySelector('meta[name="keywords"]')?.getAttribute('content') || ''
    
    // 根据URL推测分类
    const url = new URL(importUrl.value)
    const hostname = url.hostname.toLowerCase()
    let suggestedCategory = ''
    
    if (hostname.includes('github') || hostname.includes('gitlab') || hostname.includes('code')) {
      suggestedCategory = 'dev-tools'
    } else if (hostname.includes('figma') || hostname.includes('dribbble') || hostname.includes('design')) {
      suggestedCategory = 'design'
    } else if (hostname.includes('notion') || hostname.includes('trello') || hostname.includes('productivity')) {
      suggestedCategory = 'productivity'
    } else if (hostname.includes('learn') || hostname.includes('course') || hostname.includes('edu')) {
      suggestedCategory = 'learning'
    }
    
    // 填充表单
    siteInfo.value = {
      name: title || url.hostname,
      description: description || '暂无描述',
      url: importUrl.value,
      icon: 'fas fa-link',
      categoryId: suggestedCategory,
      tagsInput: keywords ? keywords.split(',').slice(0, 3).join(',') : ''
    }
    
  } catch (error) {
    console.error('获取网站信息失败:', error)
    errorMessage.value = '获取网站信息失败，请手动填写信息'
    
    // 即使失败也提供基本信息
    try {
      const url = new URL(importUrl.value)
      siteInfo.value = {
        name: url.hostname,
        description: '',
        url: importUrl.value,
        icon: 'fas fa-link',
        categoryId: '',
        tagsInput: ''
      }
    } catch (e) {
      // URL解析失败
    }
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

// 添加网站到列表
const addSite = () => {
  if (!canAddSite.value) {
    errorMessage.value = '请填写完整信息'
    return
  }

  // 生成唯一ID
  const timestamp = Date.now()
  const randomStr = Math.random().toString(36).substring(2, 9)
  const newSiteId = `custom-${timestamp}-${randomStr}`

  // 处理标签
  const tags = siteInfo.value.tagsInput
    ? siteInfo.value.tagsInput.split(',').map(tag => tag.trim()).filter(tag => tag)
    : []

  // 创建新网站对象
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

  // 保存到 localStorage
  const customSites = loadCustomData()
  customSites.push(newSite)
  saveCustomData(customSites)

  // 重新加载并合并数据
  allData.value = mergeData(defaultData.value, customSites)
  categories.value = allData.value

  // 找到分类名称
  const category = allData.value.find(cat => cat.id === siteInfo.value.categoryId)
  const categoryName = category ? category.name : '列表'
  
  // 显示成功提示
  alert(`成功添加 "${newSite.name}" 到 "${categoryName}" 分类！\n\n数据已保存到浏览器，刷新页面不会丢失。`)
  
  // 关闭弹窗
  closeModal()
  
  // 切换到对应分类
  selectCategory(siteInfo.value.categoryId)
}
</script>

