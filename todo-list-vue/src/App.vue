<script setup>
import { ref, onMounted, computed, onUnmounted, watch } from 'vue'
import TodoItem from './components/TodoItem.vue'
import TodoChart from './components/TodoChart.vue'
import { Plus, Delete, CircleCheck, CircleClose, Download, Upload, Edit, Filter } from '@element-plus/icons-vue'
import draggable from 'vuedraggable'

// 待办事项列表
const todos = ref([])

// 表单数据
const formData = ref({
  title: '',
  description: '',
  category: '',
  priority: '',
  tags: []
})

// 搜索关键词
const searchQuery = ref('')

// 文件输入引用，用于导入数据
const fileInputRef = ref(null)

// 筛选条件
const filterConditions = ref({
  status: '', // all, completed, active
  category: '',
  priority: '',
  tags: []
})

// 筛选面板显示状态
const showFilterPanel = ref(false)

// 从localStorage加载数据
const loadTodos = () => {
  const saved = localStorage.getItem('todos')
  if (saved) {
    todos.value = JSON.parse(saved)
  }
  
  // 加载自定义分类和优先级
  loadCustomCategories()
  loadCustomPriorities()
}

// 保存到localStorage
const saveTodos = () => {
  localStorage.setItem('todos', JSON.stringify(todos.value))
}

// 自定义分类和优先级
const customCategories = ref([
  { id: 1, name: '工作', color: 'primary' },
  { id: 2, name: '学习', color: 'info' },
  { id: 3, name: '生活', color: 'success' },
  { id: 4, name: '其他', color: 'warning' }
])

const customPriorities = ref([
  { id: 1, name: '高', color: 'danger' },
  { id: 2, name: '中', color: 'warning' },
  { id: 3, name: '低', color: 'success' }
])

// 标签管理
const tags = ref([
  { id: 1, name: '重要', color: '#ef4444' },
  { id: 2, name: '紧急', color: '#f59e0b' },
  { id: 3, name: '日常', color: '#10b981' }
])

const newTag = ref({ name: '', color: '#3b82f6' })
const showTagManager = ref(false)

// 从localStorage加载标签
const loadTags = () => {
  const saved = localStorage.getItem('tags')
  if (saved) {
    tags.value = JSON.parse(saved)
  }
}

// 保存标签到localStorage
const saveTags = () => {
  localStorage.setItem('tags', JSON.stringify(tags.value))
}

// 确保在加载数据时也加载标签
loadTags()

// 从localStorage加载自定义分类
const loadCustomCategories = () => {
  const saved = localStorage.getItem('customCategories')
  if (saved) {
    customCategories.value = JSON.parse(saved)
  }
}

// 从localStorage加载自定义优先级
const loadCustomPriorities = () => {
  const saved = localStorage.getItem('customPriorities')
  if (saved) {
    customPriorities.value = JSON.parse(saved)
  }
}

// 保存自定义分类到localStorage
const saveCustomCategories = () => {
  localStorage.setItem('customCategories', JSON.stringify(customCategories.value))
}

// 保存自定义优先级到localStorage
const saveCustomPriorities = () => {
  localStorage.setItem('customPriorities', JSON.stringify(customPriorities.value))
}

// 分类和优先级管理相关状态
const showCategoryManager = ref(false)
const showPriorityManager = ref(false)
const newCategory = ref({ name: '', color: 'primary' })
const newPriority = ref({ name: '', color: 'warning' })

// 添加自定义分类
const addCategory = () => {
  if (!newCategory.value.name.trim()) return
  
  const category = {
    id: Date.now(),
    name: newCategory.value.name.trim(),
    color: newCategory.value.color
  }
  
  customCategories.value.push(category)
  saveCustomCategories()
  newCategory.value = { name: '', color: 'primary' }
}

// 删除自定义分类
const deleteCategory = (id) => {
  // 如果有待办事项使用了该分类，不允许删除
  const hasTodosWithCategory = todos.value.some(todo => todo.category === id)
  if (hasTodosWithCategory) {
    alert('该分类已被使用，无法删除')
    return
  }
  
  customCategories.value = customCategories.value.filter(category => category.id !== id)
  saveCustomCategories()
}

// 添加自定义优先级
const addPriority = () => {
  if (!newPriority.value.name.trim()) return
  
  const priority = {
    id: Date.now(),
    name: newPriority.value.name.trim(),
    color: newPriority.value.color
  }
  
  customPriorities.value.push(priority)
  saveCustomPriorities()
  newPriority.value = { name: '', color: 'warning' }
}

// 删除自定义优先级
const deletePriority = (id) => {
  // 如果有待办事项使用了该优先级，不允许删除
  const hasTodosWithPriority = todos.value.some(todo => todo.priority === id)
  if (hasTodosWithPriority) {
    alert('该优先级已被使用，无法删除')
    return
  }
  
  customPriorities.value = customPriorities.value.filter(priority => priority.id !== id)
  saveCustomPriorities()
}

// 添加标签
const addTag = () => {
  if (!newTag.value.name.trim()) return
  
  const tag = {
    id: Date.now(),
    name: newTag.value.name.trim(),
    color: newTag.value.color
  }
  
  tags.value.push(tag)
  saveTags()
  newTag.value = { name: '', color: '#3b82f6' }
}

// 删除标签
const deleteTag = (id) => {
  // 移除所有待办事项中对该标签的引用
  todos.value.forEach(todo => {
    if (Array.isArray(todo.tags)) {
      todo.tags = todo.tags.filter(tagId => tagId !== id)
    }
  })
  
  tags.value = tags.value.filter(tag => tag.id !== id)
  saveTags()
  saveTodos()
}

// 添加待办事项
const addTodo = () => {
  if (!formData.value.title.trim()) return
  
  const newTodo = {
    id: Date.now(),
    title: formData.value.title,
    description: formData.value.description,
    category: formData.value.category,
    priority: formData.value.priority,
    tags: [...formData.value.tags],
    completed: false,
    subtasks: []
  }
  
  todos.value.push(newTodo)
  saveTodos()
  
  // 重置表单
  formData.value.title = ''
  formData.value.description = ''
  formData.value.category = ''
  formData.value.priority = ''
  formData.value.tags = []
}

// 切换待办事项完成状态
const toggleTodo = (id) => {
  const todo = todos.value.find(t => t.id === id)
  if (todo) {
    todo.completed = !todo.completed
    saveTodos()
  }
}

// 删除待办事项
const deleteTodo = (id) => {
  todos.value = todos.value.filter(t => t.id !== id)
  saveTodos()
}

// 更新待办事项
const updateTodo = (updatedTodo) => {
  const index = todos.value.findIndex(t => t.id === updatedTodo.id)
  if (index !== -1) {
    todos.value[index] = {
      ...todos.value[index],
      ...updatedTodo
    }
    saveTodos()
  }
}

// 搜索过滤后的待办事项
const filteredTodos = computed(() => {
  let result = todos.value
  
  // 状态筛选
  if (filterConditions.value.status === 'completed') {
    result = result.filter(todo => todo.completed)
  } else if (filterConditions.value.status === 'active') {
    result = result.filter(todo => !todo.completed)
  }
  
  // 分类筛选
  if (filterConditions.value.category) {
    result = result.filter(todo => todo.category === filterConditions.value.category)
  }
  
  // 优先级筛选
  if (filterConditions.value.priority) {
    result = result.filter(todo => todo.priority === filterConditions.value.priority)
  }
  
  // 标签筛选
  if (filterConditions.value.tags.length > 0) {
    result = result.filter(todo => {
      // 检查待办事项是否包含所有选定的标签
      return filterConditions.value.tags.every(tagId => 
        Array.isArray(todo.tags) && todo.tags.includes(tagId)
      )
    })
  }
  
  // 关键词搜索
  if (searchQuery.value.trim()) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(todo => 
      todo.title.toLowerCase().includes(query) || 
      (todo.description && todo.description.toLowerCase().includes(query))
    )
  }
  
  return result
})

// 批量操作相关状态
const selectedTodos = ref([])
const isAllSelected = computed(() => {
  return filteredTodos.value.length > 0 && selectedTodos.value.length === filteredTodos.value.length
})

// 切换单个待办事项的选中状态
const toggleSelectTodo = (id) => {
  const index = selectedTodos.value.indexOf(id)
  if (index > -1) {
    selectedTodos.value.splice(index, 1)
  } else {
    selectedTodos.value.push(id)
  }
}

// 切换全选/取消全选
const toggleSelectAll = () => {
  if (isAllSelected.value) {
    selectedTodos.value = []
  } else {
    selectedTodos.value = filteredTodos.value.map(todo => todo.id)
  }
}

// 批量删除选中的待办事项
const batchDeleteTodos = () => {
  if (selectedTodos.value.length === 0) return
  todos.value = todos.value.filter(todo => !selectedTodos.value.includes(todo.id))
  saveTodos()
  selectedTodos.value = []
}

// 批量标记为已完成
const batchMarkCompleted = () => {
  if (selectedTodos.value.length === 0) return
  todos.value.forEach(todo => {
    if (selectedTodos.value.includes(todo.id)) {
      todo.completed = true
    }
  })
  saveTodos()
  selectedTodos.value = []
}

// 批量标记为未完成
const batchMarkActive = () => {
  if (selectedTodos.value.length === 0) return
  todos.value.forEach(todo => {
    if (selectedTodos.value.includes(todo.id)) {
      todo.completed = false
    }
  })
  saveTodos()
  selectedTodos.value = []
}

// 懒加载相关状态
const lazyLoadConfig = ref({
  pageSize: 10, // 每页显示的待办事项数量
  currentPage: 1, // 当前页码
  isLoading: false, // 是否正在加载
  hasMore: true // 是否还有更多数据
})

// 当前显示的待办事项（懒加载）
const displayedTodos = computed(() => {
  const start = 0
  const end = lazyLoadConfig.value.currentPage * lazyLoadConfig.value.pageSize
  return filteredTodos.value.slice(start, end)
})

// 加载更多待办事项
const loadMoreTodos = () => {
  if (lazyLoadConfig.value.isLoading || !lazyLoadConfig.value.hasMore) return
  
  lazyLoadConfig.value.isLoading = true
  
  // 模拟加载延迟
  setTimeout(() => {
    lazyLoadConfig.value.currentPage++
    
    // 检查是否还有更多数据
    const totalItems = filteredTodos.value.length
    const displayedItems = lazyLoadConfig.value.currentPage * lazyLoadConfig.value.pageSize
    lazyLoadConfig.value.hasMore = displayedItems < totalItems
    
    lazyLoadConfig.value.isLoading = false
  }, 300)
}

// 监听筛选条件变化，重置懒加载状态
watch(
  [() => filterConditions.value, () => searchQuery.value],
  () => {
    lazyLoadConfig.value.currentPage = 1
    lazyLoadConfig.value.hasMore = true
  }
)

// 监听待办事项列表变化，重置懒加载状态
watch(
  () => todos.value.length,
  () => {
    lazyLoadConfig.value.currentPage = 1
    lazyLoadConfig.value.hasMore = true
  }
)

// 导出数据为JSON文件
const exportData = () => {
  if (todos.value.length === 0) {
    alert('没有数据可以导出')
    return
  }
  
  const dataStr = JSON.stringify(todos.value, null, 2)
  const dataBlob = new Blob([dataStr], { type: 'application/json' })
  const url = URL.createObjectURL(dataBlob)
  const link = document.createElement('a')
  link.href = url
  link.download = `todos-${new Date().toISOString().split('T')[0]}.json`
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
}

// 触发文件选择对话框
const triggerImport = () => {
  if (fileInputRef.value) {
    fileInputRef.value.click()
  }
}

// 导入数据
const importData = (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const importedTodos = JSON.parse(e.target.result)
      if (Array.isArray(importedTodos)) {
        // 合并数据，保留现有数据，添加新数据
        // 可以选择替换或合并，这里选择合并并去重
        const existingIds = new Set(todos.value.map(todo => todo.id))
        const newTodos = importedTodos.filter(todo => !existingIds.has(todo.id))
        
        todos.value = [...todos.value, ...newTodos]
        saveTodos()
        alert(`成功导入 ${newTodos.length} 条数据`)
      } else {
        alert('导入的数据格式不正确')
      }
    } catch (error) {
      alert('导入数据失败：' + error.message)
    }
  }
  reader.onerror = () => {
    alert('读取文件失败')
  }
  reader.readAsText(file)
  
  // 清空文件输入，以便下次可以重新选择同一文件
  event.target.value = ''
}

// 数据统计
const stats = computed(() => {
  // 计算自定义分类统计
  const categoryStats = customCategories.value.reduce((acc, category) => {
    acc[category.id] = todos.value.filter(t => t.category === category.id).length
    return acc
  }, {})
  
  // 添加默认分类统计，确保向后兼容
  categoryStats.work = todos.value.filter(t => t.category === 'work').length
  categoryStats.study = todos.value.filter(t => t.category === 'study').length
  categoryStats.life = todos.value.filter(t => t.category === 'life').length
  categoryStats.other = todos.value.filter(t => t.category === 'other').length
  
  return {
    total: todos.value.length,
    completed: todos.value.filter(t => t.completed).length,
    active: todos.value.filter(t => !t.completed).length,
    categories: categoryStats,
    customCategories: customCategories.value,
    customPriorities: customPriorities.value
  }
})

// 初始化加载数据
onMounted(() => {
  loadTodos()
  
  // 添加键盘事件监听器
  window.addEventListener('keydown', handleKeyDown)
})

// 移除键盘事件监听器
onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
})

// 处理键盘快捷键
const handleKeyDown = (event) => {
  // 检查是否在输入框中（避免在输入时触发快捷键）
  const isInputFocused = ['INPUT', 'TEXTAREA', 'SELECT'].includes(document.activeElement.tagName)
  if (isInputFocused) return
  
  // Ctrl/Cmd + N: 添加新待办事项
  if ((event.ctrlKey || event.metaKey) && event.key === 'n') {
    event.preventDefault()
    // 可以聚焦到添加待办事项的输入框
    const titleInput = document.querySelector('.todo-form .el-input__inner')
    if (titleInput) {
      titleInput.focus()
    }
  }
  
  // Ctrl/Cmd + D: 删除选中的待办事项
  if ((event.ctrlKey || event.metaKey) && event.key === 'd') {
    event.preventDefault()
    batchDeleteTodos()
  }
  
  // Ctrl/Cmd + Enter: 标记选中的待办事项为已完成/未完成
  if ((event.ctrlKey || event.metaKey) && event.key === 'Enter') {
    event.preventDefault()
    if (selectedTodos.value.length > 0) {
      // 检查选中的待办事项是否都已完成
      const allCompleted = todos.value.filter(todo => selectedTodos.value.includes(todo.id)).every(todo => todo.completed)
      if (allCompleted) {
        batchMarkActive()
      } else {
        batchMarkCompleted()
      }
    }
  }
  
  // Ctrl/Cmd + A: 全选/取消全选
  if ((event.ctrlKey || event.metaKey) && event.key === 'a') {
    event.preventDefault()
    toggleSelectAll()
  }
}
</script>

<template>
  <div class="app">
    <h1 class="app-title">待办事项</h1>
    
    <!-- 数据统计 -->
    <div class="stats">
      <div class="stat-item">
        <span class="stat-label">总数</span>
        <span class="stat-value">{{ stats.total }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">已完成</span>
        <span class="stat-value completed">{{ stats.completed }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">进行中</span>
        <span class="stat-value active">{{ stats.active }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">工作</span>
        <span class="stat-value work">{{ stats.categories.work }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">学习</span>
        <span class="stat-value study">{{ stats.categories.study }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">生活</span>
        <span class="stat-value life">{{ stats.categories.life }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">其他</span>
        <span class="stat-value other">{{ stats.categories.other }}</span>
      </div>
    </div>
    
    <!-- 图表统计 -->
    <TodoChart :stats="stats" :todos="todos" />
    
    <!-- 搜索和筛选功能 -->
    <div class="search-filter-container">
      <div class="search">
        <input 
          type="text" 
          v-model="searchQuery"
          placeholder="搜索待办事项..."
          class="search-input"
        />
      </div>
      
      <div class="filter-toggle">
        <el-button 
          type="primary" 
          @click="showFilterPanel = !showFilterPanel"
          size="small"
          :icon="Filter"
        >
          {{ showFilterPanel ? '收起筛选' : '筛选' }}
        </el-button>
      </div>
    </div>
    
    <!-- 筛选面板 -->
    <div v-if="showFilterPanel" class="filter-panel">
      <el-row :gutter="15">
        <el-col :span="6">
          <el-form-item label="状态">
            <el-select 
              v-model="filterConditions.status"
              placeholder="全部"
              size="small"
              clearable
            >
              <el-option label="全部" value="" />
              <el-option label="已完成" value="completed" />
              <el-option label="进行中" value="active" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="分类">
            <el-select 
              v-model="filterConditions.category"
              placeholder="全部"
              size="small"
              clearable
            >
              <el-option label="全部" value="" />
              <el-option 
                v-for="category in customCategories" 
                :key="category.id"
                :label="category.name" 
                :value="category.id" 
              />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="优先级">
            <el-select 
              v-model="filterConditions.priority"
              placeholder="全部"
              size="small"
              clearable
            >
              <el-option label="全部" value="" />
              <el-option 
                v-for="priority in customPriorities" 
                :key="priority.id"
                :label="priority.name" 
                :value="priority.id" 
              />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="6">
          <el-form-item label="标签">
            <el-select 
              v-model="filterConditions.tags"
              placeholder="全部"
              size="small"
              multiple
              clearable
            >
              <el-option 
                v-for="tag in tags" 
                :key="tag.id"
                :label="tag.name" 
                :value="tag.id" 
              />
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
      <div class="filter-actions">
        <el-button 
          type="primary" 
          @click="showFilterPanel = false"
          size="small"
        >
          应用筛选
        </el-button>
        <el-button 
          @click="filterConditions = { status: '', category: '', priority: '', tags: [] }"
          size="small"
        >
          重置筛选
        </el-button>
      </div>
    </div>
    
    <!-- 添加待办事项表单 -->
    <div class="add-todo">
      <h2>添加新待办</h2>
      <el-form :model="formData" label-position="top" class="todo-form">
        <el-form-item label="标题" required>
          <el-input 
            v-model="formData.title"
            placeholder="输入待办事项标题"
            size="large"
            clearable
          />
        </el-form-item>
        
        <el-form-item label="描述（可选）">
          <el-input 
            v-model="formData.description"
            type="textarea"
            placeholder="输入待办事项描述"
            rows="3"
            size="large"
            clearable
          />
        </el-form-item>
        
        <el-row :gutter="15">
          <el-col :span="12">
            <el-form-item label="分类">
              <div class="select-with-manager">
                <el-select 
                  v-model="formData.category"
                  placeholder="请选择分类"
                  size="large"
                  clearable
                >
                  <el-option 
                    v-for="category in customCategories" 
                    :key="category.id"
                    :label="category.name" 
                    :value="category.id" 
                  />
                </el-select>
                <el-button 
                  type="primary" 
                  @click="showCategoryManager = true"
                  size="small"
                  :icon="Edit"
                >
                  管理
                </el-button>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="优先级">
              <div class="select-with-manager">
                <el-select 
                  v-model="formData.priority"
                  placeholder="请选择优先级"
                  size="large"
                  clearable
                >
                  <el-option 
                    v-for="priority in customPriorities" 
                    :key="priority.id"
                    :label="priority.name" 
                    :value="priority.id" 
                  />
                </el-select>
                <el-button 
                  type="primary" 
                  @click="showPriorityManager = true"
                  size="small"
                  :icon="Edit"
                >
                  管理
                </el-button>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        
        <!-- 标签选择 -->
        <el-form-item label="标签（可选）">
          <div class="select-with-manager">
            <el-select 
              v-model="formData.tags"
              placeholder="选择标签（可多选）"
              size="large"
              multiple
              clearable
            >
              <el-option 
                v-for="tag in tags" 
                :key="tag.id"
                :label="tag.name" 
                :value="tag.id" 
              />
            </el-select>
            <el-button 
              type="primary" 
              @click="showTagManager = true"
              size="small"
              :icon="Edit"
            >
              管理
            </el-button>
          </div>
        </el-form-item>
        
        <!-- 标签管理弹窗 -->
        <el-dialog
          v-model="showTagManager"
          title="管理标签"
          width="500px"
          center
        >
          <div class="manager-content">
            <h3>现有标签</h3>
            <div v-if="tags.length === 0" class="empty-state">
              暂无标签，请添加
            </div>
            <div v-else class="tag-list">
              <div 
                v-for="tag in tags" 
                :key="tag.id"
                class="tag-item"
              >
                <div class="tag-info">
                  <span class="tag-color" :style="{ backgroundColor: tag.color }"></span>
                  <span class="tag-name">{{ tag.name }}</span>
                </div>
                <el-button 
                  type="danger" 
                  @click="deleteTag(tag.id)"
                  size="small"
                  :icon="Delete"
                >
                  删除
                </el-button>
              </div>
            </div>
            
            <h3>添加新标签</h3>
            <div class="add-form">
              <el-row :gutter="10">
                <el-col :span="12">
                  <el-input 
                    v-model="newTag.name"
                    placeholder="输入标签名称"
                    size="small"
                  />
                </el-col>
                <el-col :span="8">
                  <el-color-picker 
                    v-model="newTag.color"
                    show-alpha
                    size="small"
                    placeholder="选择颜色"
                  />
                </el-col>
                <el-col :span="4">
                  <el-button 
                    type="primary" 
                    @click="addTag"
                    size="small"
                    :icon="Plus"
                  >
                    添加
                  </el-button>
                </el-col>
              </el-row>
            </div>
          </div>
        </el-dialog>
        
        <!-- 分类管理弹窗 -->
        <el-dialog
          v-model="showCategoryManager"
          title="管理分类"
          width="500px"
          center
        >
          <div class="manager-content">
            <h3>现有分类</h3>
            <div v-if="customCategories.length === 0" class="empty-state">
              暂无分类，请添加
            </div>
            <div v-else class="category-list">
              <div 
                v-for="category in customCategories" 
                :key="category.id"
                class="category-item"
              >
                <div class="category-info">
                  <el-tag :type="category.color">
                    {{ category.name }}
                  </el-tag>
                </div>
                <el-button 
                  type="danger" 
                  @click="deleteCategory(category.id)"
                  size="small"
                  :icon="Delete"
                >
                  删除
                </el-button>
              </div>
            </div>
            
            <h3>添加新分类</h3>
            <div class="add-form">
              <el-row :gutter="10">
                <el-col :span="16">
                  <el-input 
                    v-model="newCategory.name"
                    placeholder="输入分类名称"
                    size="small"
                  />
                </el-col>
                <el-col :span="5">
                  <el-select 
                    v-model="newCategory.color"
                    placeholder="选择颜色"
                    size="small"
                  >
                    <el-option label="蓝色" value="primary" />
                    <el-option label="绿色" value="success" />
                    <el-option label="橙色" value="warning" />
                    <el-option label="红色" value="danger" />
                    <el-option label="紫色" value="info" />
                  </el-select>
                </el-col>
                <el-col :span="3">
                  <el-button 
                    type="primary" 
                    @click="addCategory"
                    size="small"
                    :icon="Plus"
                  >
                    添加
                  </el-button>
                </el-col>
              </el-row>
            </div>
          </div>
        </el-dialog>
        
        <!-- 优先级管理弹窗 -->
        <el-dialog
          v-model="showPriorityManager"
          title="管理优先级"
          width="500px"
          center
        >
          <div class="manager-content">
            <h3>现有优先级</h3>
            <div v-if="customPriorities.length === 0" class="empty-state">
              暂无优先级，请添加
            </div>
            <div v-else class="priority-list">
              <div 
                v-for="priority in customPriorities" 
                :key="priority.id"
                class="priority-item"
              >
                <div class="priority-info">
                  <el-tag :type="priority.color">
                    {{ priority.name }}
                  </el-tag>
                </div>
                <el-button 
                  type="danger" 
                  @click="deletePriority(priority.id)"
                  size="small"
                  :icon="Delete"
                >
                  删除
                </el-button>
              </div>
            </div>
            
            <h3>添加新优先级</h3>
            <div class="add-form">
              <el-row :gutter="10">
                <el-col :span="16">
                  <el-input 
                    v-model="newPriority.name"
                    placeholder="输入优先级名称"
                    size="small"
                  />
                </el-col>
                <el-col :span="5">
                  <el-select 
                    v-model="newPriority.color"
                    placeholder="选择颜色"
                    size="small"
                  >
                    <el-option label="红色" value="danger" />
                    <el-option label="橙色" value="warning" />
                    <el-option label="绿色" value="success" />
                    <el-option label="蓝色" value="primary" />
                    <el-option label="紫色" value="info" />
                  </el-select>
                </el-col>
                <el-col :span="3">
                  <el-button 
                    type="primary" 
                    @click="addPriority"
                    size="small"
                    :icon="Plus"
                  >
                    添加
                  </el-button>
                </el-col>
              </el-row>
            </div>
          </div>
        </el-dialog>
        
        <el-form-item>
          <el-button 
            type="primary" 
            @click="addTodo" 
            size="large"
            :icon="Plus"
            round
            style="width: 100%"
          >
            添加待办
          </el-button>
        </el-form-item>
      </el-form>
    </div>
    
    <!-- 数据管理工具栏 -->
    <div class="data-management">
      <el-button 
        type="primary" 
        @click="exportData"
        size="small"
        :icon="Download"
      >
        导出数据
      </el-button>
      <el-button 
        type="info" 
        @click="triggerImport"
        size="small"
        :icon="Upload"
      >
        导入数据
      </el-button>
      <input 
        ref="fileInputRef" 
        type="file" 
        accept=".json" 
        style="display: none" 
        @change="importData"
      />
    </div>
    
    <!-- 待办事项列表 -->
    <div class="todo-list">
      <div class="todo-list-header">
        <h2>待办列表</h2>
        
        <!-- 批量操作工具栏 -->
        <div v-if="selectedTodos.length > 0" class="batch-actions">
          <el-button 
            type="danger" 
            @click="batchDeleteTodos"
            size="small"
            :icon="Delete"
          >
            批量删除 ({{ selectedTodos.length }})
          </el-button>
          <el-button 
            type="success" 
            @click="batchMarkCompleted"
            size="small"
            :icon="CircleCheck"
          >
            标记为已完成
          </el-button>
          <el-button 
            type="warning" 
            @click="batchMarkActive"
            size="small"
            :icon="CircleClose"
          >
            标记为未完成
          </el-button>
        </div>
      </div>
      
      <!-- 全选复选框 -->
      <div v-if="filteredTodos.length > 0" class="select-all">
        <el-checkbox 
          v-model="isAllSelected" 
          @change="toggleSelectAll"
          size="large"
        >
          全选 ({{ selectedTodos.length }}/{{ filteredTodos.length }})
        </el-checkbox>
      </div>
      
      <div v-if="filteredTodos.length === 0" class="empty-state">
        {{ searchQuery ? '没有找到匹配的待办事项' : '还没有待办事项，快来添加一个吧！' }}
      </div>
      
      <!-- 懒加载列表 -->
      <draggable
        v-model="todos"
        :filter="searchQuery ? '.draggable-disabled' : ''"
        :animation="150"
        :ghost-class="'ghost-item'"
        :chosen-class="'chosen-item'"
        :drag-class="'drag-item'"
        @end="saveTodos"
        item-key="id"
      >
        <template #item="{ element }">
          <TodoItem 
            v-if="displayedTodos.some(todo => todo.id === element.id)"
            :todo="element"
            :is-selected="selectedTodos.includes(element.id)"
            :custom-categories="customCategories"
            :custom-priorities="customPriorities"
            :tags="tags"
            :class="searchQuery ? 'draggable-disabled' : ''"
            @toggle="toggleTodo"
            @delete="deleteTodo"
            @update="updateTodo"
            @toggle-select="toggleSelectTodo"
          />
        </template>
      </draggable>
      
      <!-- 加载更多按钮 -->
      <div v-if="lazyLoadConfig.hasMore" class="load-more">
        <el-button 
          type="primary" 
          @click="loadMoreTodos"
          :loading="lazyLoadConfig.isLoading"
          size="small"
        >
          {{ lazyLoadConfig.isLoading ? '加载中...' : '加载更多' }}
        </el-button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app {
  max-width: 800px;
  margin: 0 auto;
  padding: 10px;
  min-height: 100vh;
  background-color: var(--bg-color);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .app {
    padding: 10px;
  }
  
  .app-title {
    font-size: 2rem;
    margin-bottom: 20px;
  }
  
  .stats {
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
    padding: 15px;
  }
  
  .stat-value {
    font-size: 1.5rem;
  }
  
  .todo-list-header {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .batch-actions {
    width: 100%;
    justify-content: flex-start;
  }
  
  .batch-actions .el-button {
    width: auto;
    flex: 1;
    min-width: 120px;
  }
  
  .el-row {
    margin: 0;
  }
  
  .el-col {
    width: 100%;
    margin: 0;
    padding: 0;
  }
  
  .el-form-item {
    margin-bottom: 15px;
  }
}

.app-title {
  text-align: center;
  color: var(--text-primary);
  margin-bottom: 30px;
  font-size: 2.5rem;
  font-weight: 700;
  background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* 统计样式 */
.stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 15px;
  margin-bottom: 25px;
  padding: 20px;
  background-color: var(--card-color);
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.stat-item {
  text-align: center;
  padding: 15px;
  background-color: var(--bg-color);
  border-radius: 8px;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.stat-label {
  display: block;
  font-size: 0.875rem;
  color: var(--text-secondary);
  margin-bottom: 8px;
  font-weight: 500;
}

.stat-value {
  display: block;
  font-size: 2rem;
  font-weight: bold;
  color: var(--text-primary);
  line-height: 1;
}

.stat-value.completed {
  color: var(--success-color);
}

.stat-value.active {
  color: var(--primary-color);
}

.stat-value.work {
  color: var(--category-work);
}

.stat-value.study {
  color: var(--category-study);
}

.stat-value.life {
  color: var(--category-life);
}

.stat-value.other {
  color: var(--category-other);
}

/* 搜索样式 */
.search {
  margin-bottom: 25px;
}

.search-input {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid var(--border-color);
  border-radius: 12px;
  font-size: 1rem;
  box-sizing: border-box;
  background-color: var(--card-color);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.search-input:focus {
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(139, 92, 246, 0.1);
  outline: none;
}

/* 搜索和筛选容器样式 */
.search-filter-container {
  display: flex;
  gap: 15px;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.filter-toggle {
  flex-shrink: 0;
}

/* 筛选面板样式 */
.filter-panel {
  background-color: var(--card-color);
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  margin-bottom: 20px;
  border: 1px solid var(--border-color);
}

.filter-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid var(--border-color);
}



/* 响应式调整 */
@media (max-width: 768px) {
  .search-filter-container {
    flex-direction: column;
    align-items: stretch;
  }
  
  .filter-toggle {
    width: 100%;
  }
  
  .filter-toggle .el-button {
    width: 100%;
  }
  
  .el-row {
    margin: 0;
  }
  
  .el-col {
    width: 100%;
    margin: 5px 0;
  }
  
  .filter-actions {
    flex-direction: column;
  }
  
  .filter-actions .el-button {
    width: 100%;
  }
}

/* 表单样式 */
.add-todo {
  background-color: var(--card-color);
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  margin-bottom: 25px;
}

.add-todo h2 {
  margin-top: 0;
  color: var(--text-primary);
  font-size: 1.25rem;
  margin-bottom: 20px;
  font-weight: 600;
}

.form-row {
  display: flex;
  gap: 15px;
  margin-bottom: 15px;
  flex-wrap: wrap;
}

.form-group {
  flex: 1;
  min-width: 220px;
}

.form-group.full-width {
  flex: 100%;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  color: var(--text-secondary);
  font-weight: 500;
  font-size: 0.875rem;
}

.form-input,
.form-textarea,
.form-select {
  width: 100%;
  padding: 10px 14px;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  font-size: 0.95rem;
  box-sizing: border-box;
  transition: all 0.3s ease;
}

.form-input:focus,
.form-textarea:focus,
.form-select:focus {
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(139, 92, 246, 0.1);
  outline: none;
}

.form-textarea {
  resize: vertical;
  min-height: 100px;
}

.add-btn {
  background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 4px 6px -1px rgba(139, 92, 246, 0.3);
}

.add-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px -2px rgba(139, 92, 246, 0.4);
}

.add-btn:active {
  transform: translateY(0);
}

/* 待办列表样式 */
.todo-list {
  background-color: var(--card-color);
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.todo-list h2 {
  margin-top: 0;
  color: var(--text-primary);
  font-size: 1.25rem;
  margin-bottom: 20px;
  font-weight: 600;
}

.todo-list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  flex-wrap: wrap;
  gap: 10px;
}

.batch-actions {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.select-all {
  margin-bottom: 15px;
  padding: 10px;
  background-color: var(--bg-color);
  border-radius: 8px;
  border: 1px solid var(--border-color);
}

/* 拖拽排序样式 */
.draggable-disabled {
  pointer-events: none;
}

.ghost-item {
  opacity: 0.5;
  background-color: var(--primary-color);
  border-radius: 12px;
}

.chosen-item {
  transform: scale(1.02);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.15);
  z-index: 100;
}

.drag-item {
  opacity: 0.8;
  transform: rotate(2deg);
  z-index: 100;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

/* 数据管理工具栏样式 */
.data-management {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
  justify-content: flex-start;
  padding: 15px;
  background-color: var(--card-color);
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

/* 选择器与管理按钮组合样式 */
.select-with-manager {
  display: flex;
  gap: 10px;
  align-items: center;
  width: 100%;
}

.select-with-manager .el-select {
  flex: 1;
}

/* 分类和优先级管理弹窗样式 */
.manager-content {
  padding: 10px 0;
}

.manager-content h3 {
  margin: 15px 0 10px 0;
  color: var(--text-primary);
  font-size: 1rem;
  font-weight: 600;
}

.manager-content h3:first-child {
  margin-top: 0;
}

.category-list, .priority-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.category-item, .priority-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: var(--bg-color);
  border-radius: 8px;
  border: 1px solid var(--border-color);
}

.category-info, .priority-info {
  display: flex;
  align-items: center;
}

.add-form {
  margin-top: 15px;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .data-management {
    flex-direction: column;
  }
  
  .data-management .el-button {
    width: 100%;
  }
  
  .select-with-manager {
    flex-direction: column;
    align-items: stretch;
  }
  
  .select-with-manager .el-button {
    width: 100%;
  }
  
  .el-row {
    margin: 0;
  }
  
  .el-col {
    width: 100%;
    margin: 5px 0;
  }
}

.empty-state {
  text-align: center;
  color: var(--text-secondary);
  padding: 40px 20px;
  background-color: var(--bg-color);
  border-radius: 8px;
  border: 2px dashed var(--border-color);
}

/* 加载更多按钮样式 */
.load-more {
  text-align: center;
  margin-top: 20px;
  padding: 15px;
  background-color: var(--bg-color);
  border-radius: 8px;
  border: 1px solid var(--border-color);
}

.load-more .el-button {
  width: 150px;
  font-weight: 500;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .load-more {
    margin-top: 15px;
    padding: 10px;
  }
  
  .load-more .el-button {
    width: 100%;
  }
}
</style>
