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
            <button class="import-btn" @click="showImportModal = true">
              <i class="fas fa-plus"></i>
              导入网站
            </button>
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

// 导入功能相关状态
const showImportModal = ref(false)
const importUrl = ref('')
const isFetching = ref(false)
const errorMessage = ref('')
const siteInfo = ref({
  name: '',
  description: '',
  url: '',
  icon: 'fas fa-link',
  categoryId: '',
  tagsInput: ''
})

// 加载数据
onMounted(async () => {
  try {
    const response = await fetch('/data/sites.json')
    const data = await response.json()
    allData.value = data.categories
    categories.value = data.categories
  } catch (error) {
    console.error('加载数据失败:', error)
  }
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

// 关闭弹窗
const closeModal = () => {
  showImportModal.value = false
  resetImportForm()
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
  const newSiteId = `site-${timestamp}-${randomStr}`

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
    tags: tags
  }

  // 找到对应的分类并添加网站
  const categoryIndex = allData.value.findIndex(cat => cat.id === siteInfo.value.categoryId)
  if (categoryIndex !== -1) {
    allData.value[categoryIndex].sites.push(newSite)
    
    // 显示成功提示
    alert(`成功添加 "${newSite.name}" 到 "${allData.value[categoryIndex].name}" 分类！`)
    
    // 关闭弹窗
    closeModal()
    
    // 切换到对应分类
    selectCategory(siteInfo.value.categoryId)
  } else {
    errorMessage.value = '添加失败：找不到指定的分类'
  }
}
</script>

