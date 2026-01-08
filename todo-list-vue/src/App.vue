<script setup>
import { ref, onMounted, computed } from 'vue'
import TodoItem from './components/TodoItem.vue'
import TodoChart from './components/TodoChart.vue'
import { Plus } from '@element-plus/icons-vue'

// 待办事项列表
const todos = ref([])

// 表单数据
const formData = ref({
  title: '',
  description: '',
  category: '',
  priority: ''
})

// 搜索关键词
const searchQuery = ref('')

// 从localStorage加载数据
const loadTodos = () => {
  const saved = localStorage.getItem('todos')
  if (saved) {
    todos.value = JSON.parse(saved)
  }
}

// 保存到localStorage
const saveTodos = () => {
  localStorage.setItem('todos', JSON.stringify(todos.value))
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
  if (!searchQuery.value.trim()) {
    return todos.value
  }
  
  const query = searchQuery.value.toLowerCase()
  return todos.value.filter(todo => 
    todo.title.toLowerCase().includes(query) || 
    (todo.description && todo.description.toLowerCase().includes(query))
  )
})

// 数据统计
const stats = computed(() => {
  return {
    total: todos.value.length,
    completed: todos.value.filter(t => t.completed).length,
    active: todos.value.filter(t => !t.completed).length,
    categories: {
      work: todos.value.filter(t => t.category === 'work').length,
      study: todos.value.filter(t => t.category === 'study').length,
      life: todos.value.filter(t => t.category === 'life').length,
      other: todos.value.filter(t => t.category === 'other').length
    }
  }
})

// 初始化加载数据
onMounted(loadTodos)
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
    
    <!-- 搜索功能 -->
    <div class="search">
      <input 
        type="text" 
        v-model="searchQuery"
        placeholder="搜索待办事项..."
        class="search-input"
      />
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
              <el-select 
                v-model="formData.category"
                placeholder="请选择分类"
                size="large"
                clearable
              >
                <el-option label="工作" value="work" />
                <el-option label="学习" value="study" />
                <el-option label="生活" value="life" />
                <el-option label="其他" value="other" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="优先级">
              <el-select 
                v-model="formData.priority"
                placeholder="请选择优先级"
                size="large"
                clearable
              >
                <el-option label="高" value="high" />
                <el-option label="中" value="medium" />
                <el-option label="低" value="low" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        
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
    
    <!-- 待办事项列表 -->
    <div class="todo-list">
      <h2>待办列表</h2>
      <div v-if="filteredTodos.length === 0" class="empty-state">
        {{ searchQuery ? '没有找到匹配的待办事项' : '还没有待办事项，快来添加一个吧！' }}
      </div>
      <TodoItem 
        v-for="todo in filteredTodos" 
        :key="todo.id"
        :todo="todo"
        @toggle="toggleTodo"
        @delete="deleteTodo"
        @update="updateTodo"
      />
    </div>
  </div>
</template>

<style scoped>
.app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  background-color: var(--bg-color);
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

.empty-state {
  text-align: center;
  color: var(--text-secondary);
  padding: 40px 20px;
  background-color: var(--bg-color);
  border-radius: 8px;
  border: 2px dashed var(--border-color);
}
</style>
