# Vue 3 响应式核心：ref、reactive、watch、computed 全面学习指南

Vue 3 的响应式系统是其核心特性，理解 `ref`、`reactive`、`watch` 和 `computed` 是掌握 Vue 3 开发的关键。下面我将详细介绍这些 API 的用法和区别。

## 1. `ref` - 基础响应式引用

### 基本用法
```javascript
import { ref } from 'vue'

const count = ref(0) // 创建一个响应式引用，初始值为0

// 在模板中使用
// <div>{{ count }}</div>

// 在JS中访问和修改
console.log(count.value) // 访问值需要使用 .value
count.value++ // 修改值
```

### 特点
- 适用于基本类型（string, number, boolean等）
- 通过 `.value` 访问和修改值
- 在模板中自动解包（不需要 .value）
- 可以用于对象，但更推荐 `reactive` 处理对象

### 示例：计数器
```javascript
const counter = ref(0)

function increment() {
  counter.value++
}
```

## 2. `reactive` - 响应式对象

### 基本用法
```javascript
import { reactive } from 'vue'

const state = reactive({
  count: 0,
  user: {
    name: 'John',
    age: 30
  }
})

// 访问和修改
state.count++
console.log(state.user.name)
```

### 特点
- 适用于对象和数组
- 直接访问属性，不需要 .value
- 嵌套对象也是响应式的
- 不能解构，否则会失去响应性

### 示例：表单绑定
```javascript
const form = reactive({
  username: '',
  password: '',
  remember: false
})
```

## 3. `computed` - 计算属性

### 基本用法
```javascript
import { ref, computed } from 'vue'

const count = ref(0)
const doubleCount = computed(() => count.value * 2)

console.log(doubleCount.value) // 0 (初始值)
count.value = 5
console.log(doubleCount.value) // 10
```

### 特点
- 基于依赖的响应式值自动计算
- 缓存计算结果，只有依赖变化时才重新计算
- 只读（如果需要可写，需提供 getter 和 setter）

### 可写计算属性
```javascript
const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(newValue) {
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
```

## 4. `watch` - 侦听器

### 基本用法
```javascript
import { ref, watch } from 'vue'

const count = ref(0)

// 侦听单个ref
watch(count, (newValue, oldValue) => {
  console.log(`count changed from ${oldValue} to ${newValue}`)
})

// 立即执行
watch(count, (newVal) => {
  console.log(newVal)
}, { immediate: true })
```

### 侦听多个源
```javascript
const x = ref(0)
const y = ref(0)

watch([x, y], ([newX, newY], [oldX, oldY]) => {
  console.log(`x: ${newX}, y: ${newY}`)
})
```

### 深度侦听对象
```javascript
const state = reactive({ 
  user: { 
    name: 'John' 
  } 
})

watch(
  () => state.user,
  (newVal, oldVal) => {
    // 注意：newVal和oldVal相同
    // 因为它们是同一个对象
  },
  { deep: true }
)
```

### 特点
- 可以侦听一个或多个响应式源
- 提供新旧值
- 可以配置 immediate, deep 等选项
- 返回一个停止函数，可以手动停止侦听

## 5. 核心API对比

| 特性 | ref | reactive | computed | watch |
|------|-----|----------|----------|-------|
| 用途 | 基本类型响应式 | 对象响应式 | 派生值 | 响应变化执行副作用 |
| 访问方式 | .value | 直接访问 | .value | - |
| 是否缓存 | - | - | 是 | - |
| 主要优势 | 简单值响应 | 复杂对象响应 | 高效计算 | 响应变化执行操作 |

## 6. 最佳实践

1. **选择正确的API**：
   - 基本类型用 `ref`
   - 对象/数组用 `reactive`
   - 派生数据用 `computed`
   - 副作用操作用 `watch`

2. **组合式函数**：
   ```javascript
   // useCounter.js
   export function useCounter() {
     const count = ref(0)
     
     function increment() {
       count.value++
     }
     
     const double = computed(() => count.value * 2)
     
     return { count, increment, double }
   }
   ```

3. **避免响应式丢失**：
   - 不要解构 `reactive` 对象，使用 `toRefs`
   ```javascript
   const state = reactive({ x: 1, y: 2 })
   const { x, y } = toRefs(state) // 现在x和y是ref
   ```

4. **性能优化**：
   - 避免不必要的 `watch` 和 `computed`
   - 对于大型列表，考虑使用 `shallowRef` 或 `shallowReactive`

## 7. 综合示例

```javascript
import { ref, reactive, computed, watch } from 'vue'

export function useUserProfile() {
  const userId = ref(1)
  const user = reactive({
    name: '',
    age: 0,
    posts: []
  })
  
  const isAdult = computed(() => user.age >= 18)
  
  watch(userId, async (newId) => {
    const response = await fetch(`/api/users/${newId}`)
    const data = await response.json()
    Object.assign(user, data)
  }, { immediate: true })
  
  watch(
    () => user.posts.length,
    (newLength) => {
      console.log(`User now has ${newLength} posts`)
    }
  )
  
  return {
    userId,
    user,
    isAdult
  }
}
```

掌握这些核心响应式API是成为Vue 3开发高手的关键。建议通过实际项目练习，逐步深入理解它们的特性和适用场景。
