# Vue 组件系统全面学习指南

Vue 的组件系统是其核心特性，理解 `props`、`emit`、`slots` 和生命周期钩子是构建可维护、可复用组件的基础。下面我将详细介绍这些核心概念。

## 1. Props - 父组件向子组件传递数据

### 基本用法
```vue
<!-- 父组件 -->
<ChildComponent title="欢迎信息" :count="totalCount" />

<!-- 子组件 -->
<script setup>
const props = defineProps({
  title: String,
  count: {
    type: Number,
    default: 0,
    required: true,
    validator: (value) => value >= 0
  }
})
</script>

<template>
  <h2>{{ title }}</h2>
  <p>计数: {{ count }}</p>
</template>
```

### 最佳实践
1. **明确 prop 类型**：使用 TypeScript 或详细验证
2. **避免直接修改 prop**：应该通过 emit 通知父组件修改
3. **命名规范**：使用 camelCase 定义，kebab-case 传递

## 2. Emit - 子组件向父组件通信

### 基本用法
```vue
<!-- 子组件 -->
<script setup>
const emit = defineEmits(['updateCount', 'submit'])

function handleClick() {
  emit('updateCount', newValue)
}
</script>

<template>
  <button @click="handleClick">增加</button>
</template>

<!-- 父组件 -->
<ChildComponent @update-count="handleUpdate" />
```

### 带验证的 Emit
```javascript
const emit = defineEmits({
  'update-count': (value) => {
    if (value >= 0) return true
    console.warn('值必须大于等于0')
    return false
  }
})
```

## 3. Slots - 内容分发

### 默认插槽
```vue
<!-- 子组件 -->
<div class="card">
  <slot>默认内容（当父组件不提供内容时显示）</slot>
</div>

<!-- 父组件 -->
<CardComponent>
  <p>这是插入到插槽的内容</p>
</CardComponent>
```

### 具名插槽
```vue
<!-- 子组件 -->
<div class="layout">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot> <!-- 默认插槽 -->
  </main>
</div>

<!-- 父组件 -->
<LayoutComponent>
  <template v-slot:header>
    <h1>页面标题</h1>
  </template>
  
  <p>主要内容</p> <!-- 自动放入默认插槽 -->
</LayoutComponent>
```

### 作用域插槽
```vue
<!-- 子组件 -->
<ul>
  <li v-for="item in items" :key="item.id">
    <slot :item="item" :index="index"></slot>
  </li>
</ul>

<!-- 父组件 -->
<ItemList :items="items">
  <template v-slot="{ item, index }">
    {{ index + 1 }}. {{ item.name }}
  </template>
</ItemList>
```

## 4. 生命周期钩子

### 组合式 API 生命周期
```vue
<script setup>
import { onMounted, onUpdated, onUnmounted } from 'vue'

onMounted(() => {
  console.log('组件挂载完成')
  // 初始化操作，如获取数据
})

onUpdated(() => {
  console.log('组件更新完成')
})

onUnmounted(() => {
  console.log('组件卸载')
  // 清理操作，如取消事件监听
})
</script>
```

### 主要生命周期钩子
| 钩子 | 调用时机 |
|------|----------|
| `onBeforeMount` | 挂载开始之前 |
| `onMounted` | 挂载完成后 |
| `onBeforeUpdate` | 响应式数据变化，DOM 更新前 |
| `onUpdated` | DOM 更新后 |
| `onBeforeUnmount` | 组件卸载前 |
| `onUnmounted` | 组件卸载后 |
| `onErrorCaptured` | 捕获子组件错误 |

### 异步数据获取示例
```vue
<script setup>
import { ref, onMounted } from 'vue'

const data = ref(null)
const loading = ref(false)
const error = ref(null)

onMounted(async () => {
  try {
    loading.value = true
    const response = await fetch('/api/data')
    data.value = await response.json()
  } catch (err) {
    error.value = err
  } finally {
    loading.value = false
  }
})
</script>
```

## 5. 组件通信综合示例

### 父组件
```vue
<template>
  <div>
    <ChildComponent
      :user="currentUser"
      @update-user="handleUpdate"
    >
      <template #header>
        <h2>用户资料</h2>
      </template>
      
      <template #default="{ user }">
        <p>用户名: {{ user.name }}</p>
      </template>
    </ChildComponent>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const currentUser = ref({ name: '张三', age: 30 })

function handleUpdate(newUser) {
  currentUser.value = newUser
}
</script>
```

### 子组件
```vue
<template>
  <div class="user-card">
    <slot name="header"></slot>
    
    <div class="content">
      <slot :user="user"></slot>
      <p>年龄: {{ user.age }}</p>
    </div>
    
    <button @click="updateAge">增加年龄</button>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

const props = defineProps({
  user: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['update-user'])

function updateAge() {
  const newUser = { ...props.user, age: props.user.age + 1 }
  emit('update-user', newUser)
}
</script>
```

## 6. 最佳实践

1. **Props 设计原则**：
   - 保持单向数据流
   - 复杂对象使用 `v-bind` 传递属性
   - 使用 TypeScript 定义接口

2. **Emit 规范**：
   - 使用 kebab-case 事件名
   - 复杂数据应该作为对象传递
   - 考虑使用 `v-model` 简化双向绑定

3. **Slot 高级用法**：
   - 动态插槽名：`<template #[dynamicSlotName]>`
   - 无渲染组件：完全基于插槽的组件

4. **生命周期注意事项**：
   - `onMounted` 不保证所有子组件都已挂载
   - 副作用清理应在 `onUnmounted` 中进行
   - 避免在 `onUpdated` 中修改状态，可能导致无限循环

5. **组件设计模式**：
   - 容器组件 vs 展示组件
   - 复合组件（如 tabs + tab-item）
   - 高阶组件（HOC）

掌握这些组件系统核心概念后，你将能够构建出结构清晰、可维护性高的 Vue 应用组件架构。
