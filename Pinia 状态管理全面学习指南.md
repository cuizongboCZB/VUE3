# Pinia 状态管理全面学习指南

Pinia 是 Vue 官方推荐的状态管理库，相比 Vuex 更简单、类型安全且功能强大。以下是 Pinia 的完整学习路径。

## 1. Pinia 核心概念

### 与 Vuex 对比
| 特性 | Pinia | Vuex |
|------|-------|------|
| 版本 | Vue 2/3 | Vue 2/3 |
| 复杂度 | 简单 | 较复杂 |
| 类型支持 | 一流 | 需要额外配置 |
| 模块化 | 自动 | 需要手动 |
| 代码组织 | 组合式API风格 | 选项式API风格 |
| 开发体验 | 更现代 | 传统 |

## 2. 安装与基础使用

### 安装
```bash
npm install pinia
# 或
yarn add pinia
```

### 创建 Pinia 实例
```javascript
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)
app.use(createPinia())
app.mount('#app')
```

## 3. 定义 Store

### 选项式 Store (类似 Vuex)
```javascript
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0
  }),
  getters: {
    doubleCount: (state) => state.count * 2
  },
  actions: {
    increment() {
      this.count++
    }
  }
})
```

### 组合式 Store (推荐)
```javascript
// stores/user.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
  const user = ref(null)
  const isLoggedIn = computed(() => user.value !== null)
  
  function login(name) {
    user.value = { name }
  }
  
  function logout() {
    user.value = null
  }
  
  return { user, isLoggedIn, login, logout }
})
```

## 4. 使用 Store

### 组件中使用
```vue
<script setup>
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()
</script>

<template>
  <div>
    <p>Count: {{ counter.count }}</p>
    <p>Double: {{ counter.doubleCount }}</p>
    <button @click="counter.increment()">+1</button>
  </div>
</template>
```

### 状态解构保持响应式
```javascript
import { storeToRefs } from 'pinia'

const counter = useCounterStore()
const { count, doubleCount } = storeToRefs(counter) // 保持响应式
const { increment } = counter // 方法直接解构
```

## 5. 核心功能详解

### State
```javascript
// 重置状态
counter.$reset()

// 修改多个状态
counter.$patch({
  count: counter.count + 1,
  name: 'newName'
})

// 函数式修改
counter.$patch((state) => {
  state.items.push({ id: 1 })
  state.hasChanged = true
})

// 替换整个state
counter.$state = { count: 999 }
```

### Getters
```javascript
defineStore('user', {
  state: () => ({ firstName: 'John', lastName: 'Doe' }),
  getters: {
    fullName: (state) => `${state.firstName} ${state.lastName}`,
    // 使用其他getter
    greeting: (state) => {
      return `Hello, ${this.fullName}!`
    }
  }
})
```

### Actions
```javascript
defineStore('posts', {
  actions: {
    async fetchPosts() {
      this.posts = await fetch('/api/posts').then(r => r.json())
    }
  }
})

// 使用
posts.fetchPosts()
```

## 6. 插件系统

### 创建插件
```javascript
// pinia-plugin.js
export function myPiniaPlugin(context) {
  const { store } = context
  // 添加全局属性
  store.$axios = inject('axios')
  
  // 响应state变化
  store.$subscribe((mutation, state) => {
    console.log(`[${mutation.storeId}] ${mutation.type}`)
  })
  
  // 添加action
  return {
    reset() {
      store.$reset()
    }
  }
}

// 使用插件
const pinia = createPinia()
pinia.use(myPiniaPlugin)
```

## 7. 服务端渲染 (SSR)

### 基本配置
```javascript
// app.js (服务器端)
import { createPinia } from 'pinia'

export default function createApp() {
  const app = createApp(App)
  const pinia = createPinia()
  app.use(pinia)
  
  return { app, pinia }
}

// 客户端激活
pinia.state.value = window.__PINIA_STATE__
```

## 8. 最佳实践

1. **Store 组织**
   - 按功能模块划分 store
   - 避免超大 store
   - 命名规范：`useXxxStore`

2. **TypeScript 支持**
   ```typescript
   interface UserState {
     name: string | null
     age?: number
   }
   
   export const useUserStore = defineStore('user', {
     state: (): UserState => ({
       name: null
     })
   })
   ```

3. **性能优化**
   - 避免在 getter 中进行重计算
   - 对大数组使用浅响应
   - 合理使用 `$patch` 批量更新

4. **测试策略**
   ```javascript
   import { setActivePinia, createPinia } from 'pinia'
   import { useCounterStore } from '@/stores/counter'
   
   beforeEach(() => {
     setActivePinia(createPinia())
   })
   
   test('increments', () => {
     const counter = useCounterStore()
     counter.increment()
     expect(counter.count).toBe(1)
   })
   ```

## 9. 完整示例：购物车 Store

```typescript
// stores/cart.ts
import { defineStore } from 'pinia'

type CartItem = {
  id: number
  name: string
  price: number
  quantity: number
}

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [] as CartItem[],
    discount: 0
  }),
  
  getters: {
    total: (state) => {
      const subtotal = state.items.reduce(
        (sum, item) => sum + item.price * item.quantity, 0)
      return subtotal * (1 - state.discount)
    },
    itemCount: (state) => state.items.length
  },
  
  actions: {
    addItem(item: Omit<CartItem, 'quantity'>) {
      const existing = this.items.find(i => i.id === item.id)
      if (existing) {
        existing.quantity++
      } else {
        this.items.push({ ...item, quantity: 1 })
      }
    },
    
    removeItem(id: number) {
      this.items = this.items.filter(item => item.id !== id)
    },
    
    applyDiscount(percent: number) {
      this.discount = Math.min(1, Math.max(0, percent / 100))
    },
    
    async checkout() {
      const response = await fetch('/api/checkout', {
        method: 'POST',
        body: JSON.stringify(this.items)
      })
      this.$reset()
      return response.json()
    }
  }
})
```

Pinia 提供了比 Vuex 更简洁直观的 API，同时保持了强大的状态管理能力。通过合理的 Store 设计和组织，可以构建出可维护性高且类型安全的大型应用状态管理系统。
