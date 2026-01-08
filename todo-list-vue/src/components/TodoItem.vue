<template>
  <div class="todo-item">
    <!-- 查看模式 -->
    <template v-if="!isEditing">
      <div class="todo-content">
        <div class="checkboxes">
          <el-checkbox 
            :checked="isSelected" 
            @change="toggleSelect"
            size="large"
            class="select-checkbox"
          />
          <el-checkbox 
            :checked="todo.completed" 
            @change="toggleCompleted"
            size="large"
            class="complete-checkbox"
          />
        </div>
        <div class="todo-details">
          <div class="todo-header">
            <h3 class="todo-title" :class="{ 'completed': todo.completed }">
              {{ todo.title }}
            </h3>
            <div class="todo-meta">
              <el-tag 
                  v-if="todo.category" 
                  :type="getCategoryType(todo.category)"
                  size="small"
                >
                  {{ getCategoryName(todo.category) }}
                </el-tag>
                <el-tag 
                  v-if="todo.priority" 
                  :type="getPriorityType(todo.priority)"
                  size="small"
                >
                  {{ getPriorityName(todo.priority) }}
                </el-tag>
                <!-- 标签显示 -->
                <span 
                  v-for="tagId in (todo.tags || [])" 
                  :key="tagId"
                  class="todo-tag"
                  :style="{ backgroundColor: getTagById(tagId).color + '20', color: getTagById(tagId).color }"
                >
                  {{ getTagById(tagId).name }}
                </span>
            </div>
          </div>
          <p v-if="todo.description" class="todo-description" :class="{ 'completed': todo.completed }">
            {{ todo.description }}
          </p>
          
          <!-- 子任务管理 -->
          <div v-if="todo.subtasks" class="subtasks-section">
            <div class="subtasks-header">
              <span class="subtasks-count">
                子任务: {{ getCompletedSubtasksCount() }}/{{ todo.subtasks.length }}
              </span>
              <el-button 
                @click="toggleSubtasks" 
                size="small"
                type="primary"
                link
              >
                {{ showSubtasks ? '收起' : '展开' }}
              </el-button>
            </div>
            
            <!-- 子任务列表 -->
            <div v-if="showSubtasks" class="subtasks-list">
              <div 
                v-for="subtask in todo.subtasks" 
                :key="subtask.id"
                class="subtask-item"
              >
                <el-checkbox 
                  :checked="subtask.completed" 
                  @change="toggleSubtaskCompleted(subtask.id)"
                  size="small"
                />
                <span class="subtask-title" :class="{ 'completed': subtask.completed }">
                  {{ subtask.title }}
                </span>
                <el-button 
                  @click="deleteSubtask(subtask.id)"
                  size="small"
                  type="danger"
                  link
                >
                  ×
                </el-button>
              </div>
              
              <!-- 添加子任务表单 -->
              <div class="add-subtask-form">
                <el-input 
                  v-model="newSubtaskTitle"
                  placeholder="添加子任务..."
                  @keyup.enter="addSubtask"
                  size="small"
                  clearable
                />
                <el-button 
                  @click="addSubtask" 
                  size="small"
                  type="primary"
                >
                  添加
                </el-button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="todo-actions">
        <el-button @click="startEditing" size="small" type="primary" plain>
          编辑
        </el-button>
        <el-button @click="deleteTodo" size="small" type="danger" plain>
          删除
        </el-button>
      </div>
    </template>
    
    <!-- 编辑模式 -->
    <template v-else>
      <div class="todo-edit-form">
        <el-input 
          v-model="editForm.title"
          placeholder="待办事项标题"
          size="large"
        />
        <el-input 
          v-model="editForm.description"
          type="textarea"
          placeholder="待办事项描述（可选）"
          rows="3"
          size="large"
        />
        
        <el-row :gutter="15" style="margin: 15px 0;">
          <el-col :span="12">
            <div class="form-group">
                <label>分类</label>
                <el-select v-model="editForm.category" placeholder="请选择分类" size="large" clearable>
                  <el-option 
                    v-for="category in customCategories" 
                    :key="category.id"
                    :label="category.name" 
                    :value="category.id" 
                  />
                </el-select>
              </div>
          </el-col>
          <el-col :span="12">
            <div class="form-group">
                <label>优先级</label>
                <el-select v-model="editForm.priority" placeholder="请选择优先级" size="large" clearable>
                  <el-option 
                    v-for="priority in customPriorities" 
                    :key="priority.id"
                    :label="priority.name" 
                    :value="priority.id" 
                  />
                </el-select>
              </div>
          </el-col>
        </el-row>
        
        <!-- 标签选择 -->
        <div class="form-group">
          <label>标签</label>
          <el-select 
            v-model="editForm.tags"
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
        </div>
        
        <!-- 操作按钮 -->
        <div class="edit-actions">
          <el-button @click="saveEdit" type="primary">保存</el-button>
          <el-button @click="cancelEdit">取消</el-button>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { defineProps, defineEmits, ref } from 'vue'

const props = defineProps({
  todo: {
    type: Object,
    required: true
  },
  isSelected: {
    type: Boolean,
    default: false
  },
  customCategories: {
    type: Array,
    default: () => [
      { id: 1, name: '工作', color: 'primary' },
      { id: 2, name: '学习', color: 'info' },
      { id: 3, name: '生活', color: 'success' },
      { id: 4, name: '其他', color: 'warning' }
    ]
  },
  customPriorities: {
    type: Array,
    default: () => [
      { id: 1, name: '高', color: 'danger' },
      { id: 2, name: '中', color: 'warning' },
      { id: 3, name: '低', color: 'success' }
    ]
  },
  tags: {
    type: Array,
    default: () => [
      { id: 1, name: '重要', color: '#ef4444' },
      { id: 2, name: '紧急', color: '#f59e0b' },
      { id: 3, name: '日常', color: '#10b981' }
    ]
  }
})

const emit = defineEmits(['toggle', 'delete', 'update', 'toggle-select'])

const isEditing = ref(false)
const showSubtasks = ref(false)
const newSubtaskTitle = ref('')

const editForm = ref({
  title: '',
  description: '',
  category: '',
  priority: '',
  tags: []
})

// 获取分类名称
const getCategoryName = (categoryId) => {
  const category = props.customCategories.find(c => c.id === categoryId)
  return category ? category.name : ''
}

// 获取标签信息
const getTagById = (tagId) => {
  return props.tags.find(tag => tag.id === tagId) || { name: '未知', color: '#6b7280' }
}

// 获取分类对应的标签类型
const getCategoryType = (categoryId) => {
  const category = props.customCategories.find(c => c.id === categoryId)
  return category ? category.color : 'info'
}

// 获取优先级名称
const getPriorityName = (priorityId) => {
  const priority = props.customPriorities.find(p => p.id === priorityId)
  return priority ? priority.name : ''
}

// 获取优先级对应的标签类型
const getPriorityType = (priorityId) => {
  const priority = props.customPriorities.find(p => p.id === priorityId)
  return priority ? priority.color : 'info'
}

// 开始编辑
const startEditing = () => {
  // 复制当前待办事项数据到编辑表单
  editForm.value = {
    title: props.todo.title,
    description: props.todo.description || '',
    category: props.todo.category || '',
    priority: props.todo.priority || '',
    tags: [...(props.todo.tags || [])]
  }
  isEditing.value = true
}

// 保存编辑
const saveEdit = () => {
  if (!editForm.value.title.trim()) return
  
  emit('update', {
    id: props.todo.id,
    ...editForm.value
  })
  isEditing.value = false
}

// 取消编辑
const cancelEdit = () => {
  isEditing.value = false
}

// 切换完成状态
const toggleCompleted = () => {
  emit('toggle', props.todo.id)
}

// 切换选择状态
const toggleSelect = () => {
  emit('toggle-select', props.todo.id)
}

// 删除待办事项
const deleteTodo = () => {
  emit('delete', props.todo.id)
}

// 切换子任务显示/隐藏
const toggleSubtasks = () => {
  showSubtasks.value = !showSubtasks.value
}

// 添加子任务
const addSubtask = () => {
  if (!newSubtaskTitle.value.trim()) return
  
  const newSubtask = {
    id: Date.now(),
    title: newSubtaskTitle.value,
    completed: false
  }
  
  const updatedTodo = {
    ...props.todo,
    subtasks: [...(props.todo.subtasks || []), newSubtask]
  }
  
  emit('update', updatedTodo)
  newSubtaskTitle.value = ''
}

// 切换子任务完成状态
const toggleSubtaskCompleted = (subtaskId) => {
  const updatedSubtasks = props.todo.subtasks.map(subtask => {
    if (subtask.id === subtaskId) {
      return { ...subtask, completed: !subtask.completed }
    }
    return subtask
  })
  
  const updatedTodo = {
    ...props.todo,
    subtasks: updatedSubtasks
  }
  
  emit('update', updatedTodo)
}

// 删除子任务
const deleteSubtask = (subtaskId) => {
  const updatedSubtasks = props.todo.subtasks.filter(subtask => subtask.id !== subtaskId)
  
  const updatedTodo = {
    ...props.todo,
    subtasks: updatedSubtasks
  }
  
  emit('update', updatedTodo)
}

// 获取已完成子任务数量
const getCompletedSubtasksCount = () => {
  if (!props.todo.subtasks) return 0
  return props.todo.subtasks.filter(subtask => subtask.completed).length
}
</script>

<style scoped>
.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 18px;
  border-bottom: 1px solid var(--border-color);
  background-color: var(--card-color);
  border-radius: 12px;
  margin-bottom: 12px;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  gap: 15px;
}

.todo-item:hover {
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  transform: translateY(-2px);
}

.todo-content {
  display: flex;
  align-items: flex-start;
  gap: 15px;
  flex: 1;
}

.checkboxes {
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: center;
  margin-top: 4px;
}

.select-checkbox {
  margin-bottom: 8px;
}

.complete-checkbox {
  margin-top: 8px;
}

.todo-checkbox {
  margin-top: 8px;
  cursor: pointer;
  width: 20px;
  height: 20px;
  accent-color: var(--primary-color);
}

.todo-details {
  flex: 1;
}

.todo-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 10px;
}

.todo-title {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--text-primary);
  transition: all 0.3s ease;
  flex: 1;
  line-height: 1.4;
}

.todo-title.completed {
  text-decoration: line-through;
  color: var(--text-secondary);
  opacity: 0.7;
}

.todo-meta {
  display: flex;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap;
}

.todo-category {
  padding: 4px 12px;
  border-radius: 16px;
  font-size: 0.8rem;
  font-weight: 500;
  color: white;
  transition: all 0.3s ease;
}

.category-work {
  background-color: var(--category-work);
}

.category-study {
  background-color: var(--category-study);
}

.category-life {
  background-color: var(--category-life);
}

.category-other {
  background-color: var(--category-other);
}

.todo-priority {
  padding: 4px 12px;
  border-radius: 16px;
  font-size: 0.8rem;
  font-weight: 500;
  color: white;
  transition: all 0.3s ease;
}

/* 标签样式 */
.todo-tag {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 16px;
  font-size: 0.75rem;
  font-weight: 500;
  margin-left: 5px;
  transition: all 0.3s ease;
  border: 1px solid transparent;
}

.todo-tag:hover {
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* 标签管理相关样式 */
.tag-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.tag-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: var(--bg-color);
  border-radius: 8px;
  border: 1px solid var(--border-color);
}

.tag-info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.tag-color {
  width: 16px;
  height: 16px;
  border-radius: 50%;
}

.tag-name {
  font-size: 0.9rem;
  font-weight: 500;
  color: var(--text-primary);
}

.priority-high {
  background-color: var(--priority-high);
}

.priority-medium {
  background-color: var(--priority-medium);
}

.priority-low {
  background-color: var(--priority-low);
}

.todo-description {
  margin: 0 0 10px 0;
  font-size: 0.95rem;
  color: var(--text-secondary);
  line-height: 1.5;
  transition: all 0.3s ease;
}

.todo-description.completed {
  text-decoration: line-through;
  color: var(--text-secondary);
  opacity: 0.7;
}

.todo-actions {
  display: flex;
  gap: 10px;
  align-items: flex-start;
}

.todo-edit {
  background-color: var(--info-color);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(59, 130, 246, 0.2);
}

.todo-edit:hover {
  background-color: #2563eb;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(59, 130, 246, 0.3);
}

.todo-delete {
  background-color: var(--danger-color);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(239, 68, 68, 0.2);
}

.todo-delete:hover {
  background-color: #dc2626;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(239, 68, 68, 0.3);
}

/* 编辑表单样式 */
.todo-edit-form {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 15px;
  width: 100%;
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

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group label {
  font-size: 0.85rem;
  font-weight: 500;
  color: var(--text-secondary);
}

.edit-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 10px;
}

.save-btn {
  background-color: var(--success-color);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(16, 185, 129, 0.2);
}

.save-btn:hover {
  background-color: #059669;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(16, 185, 129, 0.3);
}

.cancel-btn {
  background-color: var(--text-secondary);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(100, 116, 139, 0.2);
}

.cancel-btn:hover {
  background-color: #475569;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(100, 116, 139, 0.3);
}

/* 子任务样式 */
.subtasks-section {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid var(--border-color);
}

.subtasks-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  font-size: 0.9rem;
}

.subtasks-count {
  color: var(--text-secondary);
  font-weight: 500;
}

.toggle-subtasks-btn {
  background: none;
  border: 2px solid var(--primary-color);
  color: var(--primary-color);
  cursor: pointer;
  font-size: 0.8rem;
  font-weight: 500;
  padding: 5px 12px;
  border-radius: 16px;
  transition: all 0.3s ease;
}

.toggle-subtasks-btn:hover {
  background-color: rgba(139, 92, 246, 0.1);
  transform: translateY(-1px);
}

.subtasks-list {
  margin-left: 25px;
}

.subtask-item {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
  padding: 8px 12px;
  background-color: var(--bg-color);
  border-radius: 8px;
  transition: all 0.3s ease;
  border: 1px solid var(--border-color);
}

.subtask-item:hover {
  border-color: var(--primary-color);
  box-shadow: 0 2px 4px rgba(139, 92, 246, 0.1);
}

.subtask-checkbox {
  cursor: pointer;
  width: 18px;
  height: 18px;
  accent-color: var(--success-color);
}

.subtask-title {
  flex: 1;
  font-size: 0.9rem;
  color: var(--text-primary);
  transition: all 0.3s ease;
  line-height: 1.4;
}

.subtask-title.completed {
  text-decoration: line-through;
  color: var(--text-secondary);
  opacity: 0.7;
}

.subtask-delete {
  background: none;
  border: 2px solid var(--danger-color);
  color: var(--danger-color);
  cursor: pointer;
  font-size: 1.1rem;
  font-weight: bold;
  padding: 2px 6px;
  border-radius: 50%;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 26px;
  height: 26px;
  line-height: 1;
}

.subtask-delete:hover {
  background-color: rgba(239, 68, 68, 0.1);
  transform: scale(1.1);
}

.add-subtask-form {
  display: flex;
  gap: 10px;
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px dashed var(--border-color);
}

.subtask-input {
  flex: 1;
  padding: 10px 14px;
  border: 2px solid var(--border-color);
  border-radius: 8px;
  font-size: 0.9rem;
  transition: all 0.3s ease;
}

.subtask-input:focus {
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(139, 92, 246, 0.1);
  outline: none;
}

.add-subtask-btn {
  background-color: var(--primary-color);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(139, 92, 246, 0.2);
}

.add-subtask-btn:hover {
  background-color: #7c3aed;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(139, 92, 246, 0.3);
}
</style>