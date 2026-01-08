<template>
  <div class="chart-container">
    <h2 class="chart-title">待办事项统计图表</h2>
    
    <div class="chart-grid">
      <!-- 饼图：分类分布 -->
      <div class="chart-item">
        <h3 class="chart-subtitle">分类分布</h3>
        <div ref="categoryChartRef" class="chart" id="category-chart"></div>
      </div>
      
      <!-- 环形图：完成情况 -->
      <div class="chart-item">
        <h3 class="chart-subtitle">完成情况</h3>
        <div ref="completionChartRef" class="chart" id="completion-chart"></div>
      </div>
      
      <!-- 柱状图：优先级分布 -->
      <div class="chart-item full-width">
        <h3 class="chart-subtitle">优先级分布</h3>
        <div ref="priorityChartRef" class="chart" id="priority-chart"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick } from 'vue'
import * as echarts from 'echarts'

// 接收父组件传递的统计数据
const props = defineProps({
  stats: {
    type: Object,
    required: true
  },
  todos: {
    type: Array,
    default: () => []
  }
})

// 图表实例引用
const categoryChartRef = ref(null)
const completionChartRef = ref(null)
const priorityChartRef = ref(null)

// 图表实例
let categoryChart = null
let completionChart = null
let priorityChart = null

// 初始化分类分布饼图
const initCategoryChart = () => {
  if (!categoryChartRef.value) return
  
  // 先销毁已存在的实例
  if (categoryChart) {
    categoryChart.dispose()
    categoryChart = null
  }
  
  // 初始化图表实例
  categoryChart = echarts.init(categoryChartRef.value)
  
  // 检查实例是否成功创建
  if (!categoryChart) return
  
  // 获取容器宽度，动态调整配置
  const containerWidth = categoryChartRef.value.clientWidth
  const isSmallScreen = containerWidth < 350
  
  const option = {
    tooltip: {
      trigger: 'item',
      formatter: '{a} <br/>{b}: {c} ({d}%)',
      backgroundColor: 'rgba(255, 255, 255, 0.95)',
      borderColor: '#e2e8f0',
      borderWidth: 1,
      textStyle: {
        color: '#1e293b',
        fontSize: 12
      },
      padding: 10,
      borderRadius: 8,
      boxShadow: '0 2px 4px rgba(0, 0, 0, 0.1)'
    },
    legend: {
      orient: isSmallScreen ? 'horizontal' : 'vertical',
      left: isSmallScreen ? 'center' : 'left',
      top: isSmallScreen ? 'top' : 'center',
      bottom: isSmallScreen ? 'auto' : 'auto',
      textStyle: {
        color: '#64748b',
        fontSize: 11,
        fontWeight: 500
      },
      itemGap: 8,
      itemWidth: 8,
      itemHeight: 8
    },
    series: [
      {
        name: '分类分布',
        type: 'pie',
        radius: isSmallScreen ? ['40%', '75%'] : ['35%', '75%'],
        center: isSmallScreen ? ['50%', '65%'] : ['65%', '50%'],
        avoidLabelOverlap: true,
        itemStyle: {
          borderRadius: 10,
          borderColor: '#fff',
          borderWidth: 2,
          shadowBlur: 5,
          shadowOffsetX: 0,
          shadowColor: 'rgba(0, 0, 0, 0.1)'
        },
        data: [
          { value: props.stats.categories.work, name: '工作', itemStyle: { color: '#3b82f6' } },
          { value: props.stats.categories.study, name: '学习', itemStyle: { color: '#8b5cf6' } },
          { value: props.stats.categories.life, name: '生活', itemStyle: { color: '#ec4899' } },
          { value: props.stats.categories.other, name: '其他', itemStyle: { color: '#6b7280' } }
        ],
        emphasis: {
          scale: true,
          itemStyle: {
            shadowBlur: 15,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.3)'
          }
        },
        label: {
          show: !isSmallScreen,
          formatter: '{b}\n{d}%',
          fontSize: 12,
          fontWeight: 'bold',
          color: '#1e293b'
        },
        labelLine: {
          show: !isSmallScreen,
          length: 10,
          length2: 10,
          lineStyle: {
            color: '#cbd5e1'
          }
        }
      }
    ]
  }
  
  // 确保实例有效后再调用setOption
  if (categoryChart) {
    categoryChart.setOption(option)
  }
}

// 初始化完成情况环形图
const initCompletionChart = () => {
  if (!completionChartRef.value) return
  
  // 先销毁已存在的实例
  if (completionChart) {
    completionChart.dispose()
    completionChart = null
  }
  
  completionChart = echarts.init(completionChartRef.value)
  
  // 检查实例是否成功创建
  if (!completionChart) return
  
  // 获取容器宽度，动态调整配置
  const containerWidth = completionChartRef.value.clientWidth
  const isSmallScreen = containerWidth < 350
  
  const option = {
    tooltip: {
      trigger: 'item',
      formatter: '{a} <br/>{b}: {c} ({d}%)',
      backgroundColor: 'rgba(255, 255, 255, 0.95)',
      borderColor: '#e2e8f0',
      borderWidth: 1,
      textStyle: {
        color: '#1e293b',
        fontSize: 12
      },
      padding: 10,
      borderRadius: 8,
      boxShadow: '0 2px 4px rgba(0, 0, 0, 0.1)'
    },
    legend: {
      orient: isSmallScreen ? 'horizontal' : 'vertical',
      left: isSmallScreen ? 'center' : 'left',
      top: isSmallScreen ? 'top' : 'center',
      bottom: isSmallScreen ? 'auto' : 'auto',
      textStyle: {
        color: '#64748b',
        fontSize: 11,
        fontWeight: 500
      },
      itemGap: 8,
      itemWidth: 8,
      itemHeight: 8
    },
    series: [
      {
        name: '完成情况',
        type: 'pie',
        radius: isSmallScreen ? ['40%', '75%'] : ['45%', '75%'],
        center: isSmallScreen ? ['50%', '65%'] : ['65%', '50%'],
        avoidLabelOverlap: false,
        itemStyle: {
          borderRadius: 15,
          borderColor: '#fff',
          borderWidth: 2,
          shadowBlur: 5,
          shadowOffsetX: 0,
          shadowColor: 'rgba(0, 0, 0, 0.1)'
        },
        data: [
          { value: props.stats.completed, name: '已完成', itemStyle: { color: '#10b981' } },
          { value: props.stats.active, name: '进行中', itemStyle: { color: '#f59e0b' } }
        ],
        emphasis: {
          scale: true,
          itemStyle: {
            shadowBlur: 15,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.3)'
          }
        },
        label: {
          show: false
        },
        labelLine: {
          show: false
        }
      }
    ]
  }
  
  // 确保实例有效后再调用setOption
  if (completionChart) {
    completionChart.setOption(option)
  }
}

// 初始化优先级分布柱状图
const initPriorityChart = () => {
  if (!priorityChartRef.value) return
  
  // 先销毁已存在的实例
  if (priorityChart) {
    priorityChart.dispose()
    priorityChart = null
  }
  
  priorityChart = echarts.init(priorityChartRef.value)
  
  // 检查实例是否成功创建
  if (!priorityChart) return
  
  // 计算优先级分布数据
  const priorityData = [
    { name: '高', value: 0, itemStyle: { color: '#ef4444' } },
    { name: '中', value: 0, itemStyle: { color: '#f59e0b' } },
    { name: '低', value: 0, itemStyle: { color: '#10b981' } }
  ]
  
  // 从待办事项中计算优先级分布
  if (Array.isArray(props.todos)) {
    props.todos.forEach(todo => {
      if (todo.priority === 'high') priorityData[0].value++
      if (todo.priority === 'medium') priorityData[1].value++
      if (todo.priority === 'low') priorityData[2].value++
    })
  }
  
  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow',
        shadowStyle: {
          color: 'rgba(0, 0, 0, 0.05)'
        }
      },
      backgroundColor: 'rgba(255, 255, 255, 0.95)',
      borderColor: '#e2e8f0',
      borderWidth: 1,
      textStyle: {
        color: '#1e293b',
        fontSize: 12
      },
      padding: 10,
      borderRadius: 8,
      boxShadow: '0 2px 4px rgba(0, 0, 0, 0.1)'
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: priorityData.map(item => item.name),
      axisLabel: {
        color: '#64748b',
        fontSize: 12,
        fontWeight: 500
      },
      axisLine: {
        lineStyle: {
          color: '#e2e8f0'
        }
      },
      axisTick: {
        show: false
      }
    },
    yAxis: {
      type: 'value',
      axisLabel: {
        color: '#64748b',
        fontSize: 11
      },
      axisLine: {
        lineStyle: {
          color: '#e2e8f0'
        }
      },
      splitLine: {
        lineStyle: {
          color: '#f1f5f9',
          type: 'dashed'
        }
      }
    },
    series: [
      {
        name: '优先级分布',
        type: 'bar',
        data: priorityData.map(item => ({
          value: item.value,
          itemStyle: item.itemStyle
        })),
        barWidth: '40%',
        emphasis: {
          scale: true,
          itemStyle: {
            shadowBlur: 15,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.3)'
          }
        },
        label: {
          show: true,
          position: 'top',
          color: '#1e293b',
          fontSize: 11,
          fontWeight: 500,
          formatter: '{c}'
        },
        animationDelay: function (idx) {
          return idx * 100;
        },
        animationEasing: 'elasticOut',
        animationDuration: 1000
      }
    ]
  }
  
  // 确保实例有效后再调用setOption
  if (priorityChart) {
    priorityChart.setOption(option)
  }
}

// 初始化所有图表
const initCharts = () => {
  nextTick(() => {
    initCategoryChart()
    initCompletionChart()
    initPriorityChart()
  })
}

// 监听窗口大小变化，自适应调整图表大小
const handleResize = () => {
  categoryChart && categoryChart.resize()
  completionChart && completionChart.resize()
  priorityChart && priorityChart.resize()
}

// 监听数据变化，更新图表
watch(
  () => props.stats,
  () => {
    initCharts()
  },
  { deep: true }
)

// 监听待办事项列表变化，更新图表
watch(
  () => props.todos,
  () => {
    initCharts()
  },
  { deep: true }
)

// 组件挂载时初始化图表
onMounted(() => {
  initCharts()
  window.addEventListener('resize', handleResize)
})

// 组件卸载时销毁图表实例
onUnmounted(() => {
  categoryChart && categoryChart.dispose()
  completionChart && completionChart.dispose()
  priorityChart && priorityChart.dispose()
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.chart-container {
  background-color: var(--card-color);
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  margin-bottom: 25px;
}

.chart-title {
  margin-top: 0;
  margin-bottom: 25px;
  color: var(--text-primary);
  font-size: 1.5rem;
  font-weight: 600;
  text-align: center;
}

.chart-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 25px;
}

/* 响应式设计，在小屏幕上确保图表完整显示 */
@media (max-width: 600px) {
  .chart-grid {
    grid-template-columns: 1fr;
  }
  
  .chart-item {
    padding: 15px;
  }
  
  .chart {
    height: 250px;
  }
}

.chart-item {
  background-color: var(--bg-color);
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.chart-item.full-width {
  grid-column: 1 / -1;
}

.chart-subtitle {
  margin-top: 0;
  margin-bottom: 15px;
  color: var(--text-primary);
  font-size: 1.1rem;
  font-weight: 500;
  text-align: center;
}

.chart {
  width: 100%;
  height: 300px;
}
</style>