# Vue Router 全面学习指南

Vue Router 是 Vue.js 官方的路由管理器，用于构建单页面应用（SPA）。以下是 Vue Router 的核心概念和使用方法。

## 1. 安装与基础配置

### 安装
```bash
npm install vue-router@4
# 或
yarn add vue-router@4
```

### 基本配置
```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: About
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

### 在 main.js 中使用
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(router)
app.mount('#app')
```

## 2. 路由视图与导航

### 路由出口
```vue
<!-- App.vue -->
<template>
  <nav>
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>
  </nav>
  <router-view/>
</template>
```

### 编程式导航
```javascript
// 在组件方法中
this.$router.push('/about') // 跳转到关于页
this.$router.push({ name: 'About' }) // 使用命名路由
this.$router.replace('/login') // 替换当前历史记录
this.$router.go(-1) // 后退一页
```

### 组合式 API 中使用路由
```javascript
import { useRouter } from 'vue-router'

export default {
  setup() {
    const router = useRouter()
    
    const navigate = () => {
      router.push('/about')
    }
    
    return { navigate }
  }
}
```

## 3. 动态路由与参数

### 带参数的路由
```javascript
// router/index.js
{
  path: '/user/:id',
  name: 'User',
  component: User
}
```

### 获取路由参数
```vue
<!-- User.vue -->
<template>
  <div>User ID: {{ $route.params.id }}</div>
</template>

<script>
export default {
  created() {
    console.log(this.$route.params.id)
  }
}
</script>
```

### 组合式 API 获取参数
```javascript
import { useRoute } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    console.log(route.params.id)
  }
}
```

## 4. 嵌套路由

### 配置嵌套路由
```javascript
{
  path: '/dashboard',
  component: Dashboard,
  children: [
    {
      path: '',
      component: DashboardDefault
    },
    {
      path: 'settings',
      component: DashboardSettings
    }
  ]
}
```

### 嵌套路由视图
```vue
<!-- Dashboard.vue -->
<template>
  <div>
    <h1>Dashboard</h1>
    <router-view/> <!-- 子路由将在这里渲染 -->
  </div>
</template>
```

## 5. 路由守卫

### 全局前置守卫
```javascript
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth && !isAuthenticated()) {
    next('/login')
  } else {
    next()
  }
})
```

### 路由独享守卫
```javascript
{
  path: '/admin',
  component: Admin,
  beforeEnter: (to, from, next) => {
    // 登录检查逻辑
  }
}
```

### 组件内守卫
```javascript
export default {
  beforeRouteEnter(to, from, next) {
    // 在渲染该组件的对应路由被验证前调用
  },
  beforeRouteUpdate(to, from, next) {
    // 当前路由改变，但该组件被复用时调用
  },
  beforeRouteLeave(to, from, next) {
    // 导航离开该组件的对应路由时调用
  }
}
```

## 6. 路由元信息

### 定义路由元信息
```javascript
{
  path: '/admin',
  meta: {
    requiresAuth: true,
    title: 'Admin Dashboard'
  }
}
```

### 使用元信息
```javascript
router.beforeEach((to, from, next) => {
  document.title = to.meta.title || 'My App'
  next()
})
```

## 7. 路由懒加载

### 动态导入组件
```javascript
{
  path: '/about',
  name: 'About',
  component: () => import('../views/About.vue')
}
```

### 分组和命名 chunk
```javascript
component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
```

## 8. 滚动行为

### 自定义滚动行为
```javascript
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else {
      return { top: 0 }
    }
  }
})
```

## 9. 路由模式

### Hash 模式
```javascript
createWebHashHistory() // URL 中有 # 符号
```

### HTML5 模式 (需要服务器支持)
```javascript
createWebHistory() // 干净的 URL
```

## 10. 完整示例

### 路由配置
```javascript
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('../views/Home.vue'),
    meta: {
      title: 'Home Page'
    }
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('../views/Login.vue'),
    meta: {
      title: 'Login',
      guest: true
    }
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('../views/Dashboard.vue'),
    meta: {
      title: 'Dashboard',
      requiresAuth: true
    },
    children: [
      {
        path: '',
        component: () => import('../views/Dashboard/Index.vue')
      },
      {
        path: 'settings',
        component: () => import('../views/Dashboard/Settings.vue')
      }
    ]
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: () => import('../views/NotFound.vue'),
    meta: {
      title: '404 Not Found'
    }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

router.beforeEach((to, from, next) => {
  document.title = to.meta.title || 'My App'
  
  const isAuthenticated = checkAuth() // 你的认证检查逻辑
  
  if (to.meta.requiresAuth && !isAuthenticated) {
    next({ name: 'Login' })
  } else if (to.meta.guest && isAuthenticated) {
    next({ name: 'Dashboard' })
  } else {
    next()
  }
})

export default router
```

### 导航守卫实用示例
```javascript
// 检查用户权限
router.beforeEach(async (to, from, next) => {
  const requiresAuth = to.matched.some(record => record.meta.requiresAuth)
  const user = await getCurrentUser() // 获取当前用户
  
  if (requiresAuth && !user) {
    next('/login')
  } else if (to.path === '/login' && user) {
    next('/dashboard')
  } else {
    next()
  }
})
```

## 11. 最佳实践

1. **路由组织**
   - 按功能模块拆分路由文件
   - 使用命名路由代替硬编码路径
   - 为路由添加有意义的名称

2. **性能优化**
   - 使用路由懒加载
   - 合理使用路由组件缓存 (`<keep-alive>`)
   - 避免在全局守卫中进行重操作

3. **安全考虑**
   - 在前端和后端都进行权限验证
   - 敏感路由添加认证检查
   - 考虑使用路由元信息进行权限控制

4. **开发体验**
   - 使用路由元信息管理页面标题
   - 实现良好的404处理
   - 添加路由过渡动画提升用户体验

Vue Router 提供了强大的功能来管理单页面应用的路由和导航。通过合理配置和使用高级功能，可以构建出结构清晰、用户体验良好的应用。
