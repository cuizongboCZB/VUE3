# Vue 模板语法全面学习指南

Vue 的模板语法是连接组件逻辑与视图的桥梁，以下是核心模板指令的详细解析。

## 1. 插值表达式 `{{ }}`

### 基础文本插值
```html
<p>Message: {{ msg }}</p>
```
- 显示组件中 `msg` 属性的值
- 自动响应数据变化更新
- **不支持** HTML 解析（使用 `v-html` 指令解析 HTML）

### JavaScript 表达式
```html
<p>{{ number + 1 }}</p>
<p>{{ ok ? 'YES' : 'NO' }}</p>
<p>{{ message.split('').reverse().join('') }}</p>
```
- 支持单一表达式（不能是语句）
- 可以访问组件作用域内的属性

## 2. 属性绑定 `v-bind` 或 `:`

### 基础属性绑定
```html
<img v-bind:src="imageSrc">
<!-- 简写 -->
<img :src="imageSrc">
```

### 动态属性名
```html
<button :[dynamicAttr]="value">按钮</button>
```
```javascript
data() {
  return {
    dynamicAttr: 'disabled',
    value: true
  }
}
```

### 绑定对象
```html
<div v-bind="{ id: containerId, class: wrapperClass }"></div>
```
等价于：
```html
<div :id="containerId" :class="wrapperClass"></div>
```

### 特殊场景
```html
<!-- 类绑定 -->
<div :class="{ active: isActive, 'text-danger': hasError }"></div>

<!-- 样式绑定 -->
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

## 3. 双向绑定 `v-model`

### 基础用法
```html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

### 修饰符
| 修饰符 | 作用 | 示例 |
|--------|------|------|
| `.lazy` | 改为 change 事件触发 | `v-model.lazy` |
| `.number` | 自动转为数字 | `v-model.number` |
| `.trim` | 自动去除首尾空格 | `v-model.trim` |

### 组件上使用
```html
<CustomInput v-model="searchText" />
```
等价于：
```html
<CustomInput
  :modelValue="searchText"
  @update:modelValue="newValue => searchText = newValue"
/>
```

## 4. 条件渲染 `v-if` vs `v-show`

### `v-if` 指令
```html
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else-if="ok">Okay</h1>
<h1 v-else>Oh no 😢</h1>
```
- **真正**的条件渲染，元素存在/不存在于 DOM
- 适合运行时条件很少改变的场景

### `v-show` 指令
```html
<h1 v-show="isVisible">Hello!</h1>
```
- 总是渲染，只是切换 `display` CSS 属性
- 适合频繁切换的场景

## 5. 列表渲染 `v-for`

### 基础列表
```html
<ul>
  <li v-for="item in items" :key="item.id">
    {{ item.text }}
  </li>
</ul>
```

### 带索引
```html
<li v-for="(item, index) in items"></li>
```

### 遍历对象
```html
<li v-for="(value, key, index) in myObject"></li>
```

### 重要实践
1. **始终提供 `key`**：  
   ```html
   <div v-for="item in items" :key="item.id"></div>
   ```
2. **避免 `v-for` 和 `v-if` 一起用**（`v-for` 优先级更高）

## 6. 事件绑定 `v-on` 或 `@`

### 基础事件
```html
<button v-on:click="count++">Add 1</button>
<!-- 简写 -->
<button @click="count++">Add 1</button>
```

### 方法处理
```html
<button @click="greet">Greet</button>
```
```javascript
methods: {
  greet(event) {
    // event 是原生 DOM 事件
    alert('Hello!')
  }
}
```

### 事件修饰符
| 修饰符 | 作用 |
|--------|------|
| `.stop` | 阻止事件冒泡 |
| `.prevent` | 阻止默认行为 |
| `.capture` | 使用捕获模式 |
| `.self` | 仅当 event.target 是元素自身时触发 |
| `.once` | 只触发一次 |
| `.passive` | 改善滚动性能 |

```html
<form @submit.prevent="onSubmit"></form>
<a @click.stop.prevent="doThat"></a>
```

### 按键修饰符
```html
<input @keyup.enter="submit">
<input @keyup.page-down="onPageDown">
```
常用按键别名：`.enter`, `.tab`, `.delete`, `.esc`, `.space`, `.up`, `.down`, `.left`, `.right`

## 7. 综合应用示例

```html
<template>
  <div>
    <!-- 条件渲染 -->
    <div v-if="loading">Loading...</div>
    
    <!-- 列表渲染 -->
    <ul v-else>
      <li 
        v-for="user in filteredUsers" 
        :key="user.id"
        :class="{ active: selectedUser === user.id }"
        @click="selectUser(user.id)"
      >
        {{ user.name }} ({{ user.age }})
      </li>
    </ul>
    
    <!-- 表单绑定 -->
    <input 
      v-model.trim="searchQuery" 
      @keyup.enter="search"
      placeholder="Search users..."
    >
    
    <!-- 自定义组件 -->
    <UserDetail 
      v-model:user="currentUser" 
      @update="handleUpdate"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      loading: false,
      users: [],
      selectedUser: null,
      searchQuery: '',
      currentUser: null
    }
  },
  computed: {
    filteredUsers() {
      return this.users.filter(user => 
        user.name.includes(this.searchQuery)
      )
    }
  },
  methods: {
    selectUser(id) {
      this.selectedUser = id
    },
    search() {
      // 搜索逻辑
    },
    handleUpdate(userData) {
      // 处理更新
    }
  }
}
</script>
```

## 8. 最佳实践

1. **保持模板简洁**：复杂逻辑应移到计算属性或方法中
2. **合理使用指令**：
   - 频繁切换用 `v-show`，很少变化用 `v-if`
   - 列表始终提供唯一的 `key`
3. **事件处理**：
   - 简单表达式可直接写在模板中
   - 复杂逻辑应调用方法
4. **组件通信**：
   - 父传子用 `props`
   - 子传父用 `$emit`
   - 复杂场景考虑 Vuex/Pinia

掌握这些模板语法后，你将能够高效地构建 Vue 应用的视图层。建议通过实际项目练习来巩固这些概念。
