# 以记账本应用为例分析React前端架构

### 项目功能

- 添加/删除账目
- 查看账目列表
- 计算总收入/支出
- 数据本地存储

### Step 1: 定义服务 (Services)

> 服务 = 纯功能模块，提供具体功能，包括业务逻辑和UI表现服务，不关心谁用，怎么用

```javascript
class 存储服务 {
  保存数据(key, data) { /* 存到localStorage */ }
  读取数据(key) { /* 从localStorage读 */ }
}

class 通知服务 {
  显示成功(message) { /* 显示绿色的成功提示 */ }
  显示错误(message) { /* 显示红色的错误提示 */ }
}

```

```javascript
// services/StorageService.js - 存储服务
class StorageService {
  save(key, data) {
    localStorage.setItem(key, JSON.stringify(data))
  }
  
  load(key) {
    const data = localStorage.getItem(key)
    return data ? JSON.parse(data) : []
  }
}

// services/NotificationService.js - 通知服务
class NotificationService {
  showSuccess(message) {
    console.log(`✅ ${message}`)
  }
  
  showError(message) {
    console.log(`❌ ${message}`)
  }
}
```

### Step 2: 创建容器 (Containers)

> 容器 = 管理服务的使用，协调多个服务之间的关系

```js
function 账目管理者容器({ children, 存储服务, 通知服务 }) {
  // 容器内部使用这些服务
  const 添加账目 = (新账目) => {
    存储服务.保存数据('账目们', [...旧账目们, 新账目])
    通知服务.显示成功('添加成功！')
  }
  
  // 容器把"添加账目"功能提供给子组件使用
  return (
    <账目上下文.Provider value={{ 添加账目 }}>
      {children}
    </账目上下文.Provider>
  )
}

```

```jsx
// contexts/ExpenseContext.js - 账目管理容器
import { createContext, useContext, useState, useCallback } from 'react'

const ExpenseContext = createContext()

export function ExpenseProvider({ children, storageService, notificationService }) {
  // 注入的服务
  const storage = storageService
  const notification = notificationService
  
  const [expenses, setExpenses] = useState(() => 
    storage.load('expenses') // 使用注入的存储服务
  )
  
  // 添加账目
  const addExpense = useCallback((expense) => {
    const newExpenses = [...expenses, { ...expense, id: Date.now() }]
    setExpenses(newExpenses)
    storage.save('expenses', newExpenses) // 使用注入的存储服务
    
    notification.showSuccess('账目添加成功') // 使用注入的通知服务
  }, [expenses, storage, notification])
  
  // 删除账目
  const deleteExpense = useCallback((id) => {
    const newExpenses = expenses.filter(exp => exp.id !== id)
    setExpenses(newExpenses)
    storage.save('expenses', newExpenses)
    
    notification.showError('账目已删除') // 这里用error样式但表示删除操作
  }, [expenses, storage, notification])
  
  // 计算总计
  const total = expenses.reduce((sum, exp) => 
    exp.type === 'income' ? sum + exp.amount : sum - exp.amount, 0
  )
  
  return (
    <ExpenseContext.Provider value={{
      expenses,
      total,
      addExpense,
      deleteExpense
    }}>
      {children}
    </ExpenseContext.Provider>
  )
}

export const useExpenses = () => useContext(ExpenseContext)
```

### Step 3: 页面组件使用注入的服务

> 页面组件 = 使用功能的消费者，使用容器提供的功能，负责UI交互

```js
// 页面组件只需要调用"添加账目"，不需要知道怎么存数据、怎么显示通知
function 添加账目页面() {
  const { 添加账目 } = use账目() // 从容器获取功能
  
  const 提交表单 = () => {
    添加账目(表单数据) // 直接调用，容器会处理存储和通知
  }
}

```

```jsx
// pages/AddExpense.js - 添加账目页面
import { useState } from 'react'
import { useExpenses } from '../contexts/ExpenseContext'

function AddExpense() {
  const { addExpense } = useExpenses() // 获取注入的任务服务
  const [formData, setFormData] = useState({
    description: '',
    amount: '',
    type: 'expense'
  })
  
  const handleSubmit = (e) => {
    e.preventDefault()
    
    // 使用注入的 addExpense 服务
    addExpense({
      description: formData.description,
      amount: parseFloat(formData.amount),
      type: formData.type,
      date: new Date().toISOString()
    })
    
    // 清空表单
    setFormData({ description: '', amount: '', type: 'expense' })
  }
  
  return (
    <div>
      <h1>添加账目</h1>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="描述"
          value={formData.description}
          onChange={(e) => setFormData({...formData, description: e.target.value})}
        />
        <input
          type="number"
          placeholder="金额"
          value={formData.amount}
          onChange={(e) => setFormData({...formData, amount: e.target.value})}
        />
        <select
          value={formData.type}
          onChange={(e) => setFormData({...formData, type: e.target.value})}
        >
          <option value="expense">支出</option>
          <option value="income">收入</option>
        </select>
        <button type="submit">添加</button>
      </form>
    </div>
  )
}

export default AddExpense
```

```jsx
// pages/ExpenseList.js - 账目列表页面
import { useExpenses } from '../contexts/ExpenseContext'

function ExpenseList() {
  const { expenses, deleteExpense } = useExpenses() // 获取注入的服务
  
  return (
    <div>
      <h1>账目列表</h1>
      <ul>
        {expenses.map(expense => (
          <li key={expense.id}>
            <span>{expense.description}</span>
            <span>{expense.type === 'income' ? '+' : '-'}${expense.amount}</span>
            <button onClick={() => deleteExpense(expense.id)}>
              删除
            </button>
          </li>
        ))}
      </ul>
    </div>
  )
}

export default ExpenseList
```

```jsx
// pages/Dashboard.js - 仪表板页面
import { useExpenses } from '../contexts/ExpenseContext'

function Dashboard() {
  const { total, expenses } = useExpenses() // 获取注入的服务
  
  const income = expenses
    .filter(exp => exp.type === 'income')
    .reduce((sum, exp) => sum + exp.amount, 0)
    
  const expense = expenses
    .filter(exp => exp.type === 'expense')
    .reduce((sum, exp) => sum + exp.amount, 0)
  
  return (
    <div>
      <h1>记账本仪表板</h1>
      <div>
        <p>总余额: ${total}</p>
        <p>总收入: ${income}</p>
        <p>总支出: ${expense}</p>
      </div>
    </div>
  )
}

export default Dashboard
```

### Step 4: 路由逻辑 (Logic Layer)

> **`BrowserRouter` (路由容器 - 全局)** → 提供全局路由服务，管理整个应用的URL导航和历史记录
>
> **`TaskProvider` (任务容器 - 业务功能)** → 管理任务相关的服务，协调任务状态、存储和通知服务之间的关系
>
> **`Routes` (路由逻辑 - 决定显示什么)** → 根据当前URL路径，决定渲染哪个组件，处理路由匹配逻辑

```jsx
// App.js - 应用主组件，包含路由逻辑
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { ExpenseProvider } from './contexts/ExpenseContext'
import { StorageService } from './services/StorageService'
import { NotificationService } from './services/NotificationService'
import ExpenseList from './pages/ExpenseList'
import AddExpense from './pages/AddExpense'
import Dashboard from './pages/Dashboard'
import NotificationContainer from './components/NotificationContainer'

function App() {
  // 创建服务实例
  const storageService = new StorageService()
  const notificationService = new NotificationService()
  
  return (
    <BrowserRouter>  {/* Step 3.1: 路由容器 */}
      <ExpenseProvider 
        storageService={storageService}    {/* 注入存储服务 */}
        notificationService={notificationService}  {/* 注入通知服务 */}
      >
        <Routes>  {/* Step 3.2: 路由逻辑 */}
          <Route path="/" element={<Dashboard />} />
          <Route path="/expenses" element={<ExpenseList />} />
          <Route path="/add" element={<AddExpense />} />
        </Routes>
        
        <NotificationContainer />  {/* Step 3.3: UI表现层 */}
      </ExpenseProvider>
    </BrowserRouter>
  )
}
```



### 完整的依赖注入流程图

```
App (应用入口)
├── 创建服务实例
│   ├── storageService (存储服务)
│   └── notificationService (通知服务)
│
├── BrowserRouter (路由容器)
│   ├── ExpenseProvider (账目管理容器)
│   │   ├── 注入 storageService 和 notificationService
│   │   ├── 提供 addExpense, deleteExpense 等服务
│   │   │
│   │   ├── Routes (路由逻辑)
│   │   │   ├── Dashboard - 使用 total, expenses 服务
│   │   │   ├── ExpenseList - 使用 expenses, deleteExpense 服务
│   │   │   └── AddExpense - 使用 addExpense 服务
│   │   │
│   │   └── NotificationContainer (UI表现层)
│   │       └── 显示 notificationService 的消息
```



通过这个记账本例子，你能看到：

1. **路由功能的容器: **`BrowserRouter`

2. **业务功能的容器**： `ExpenseProvider`

3. **具体服务**：`StorageService`, `NotificationService`

4. **逻辑**：`Routes`, `addExpense函数`

5. **表现层**：各个页面组件, `NotificationContainer`

6. **依赖注入**：服务从外部注入到组件中

   

## 包裹层次关系（从外到内）：

### 1. **路由功能的容器** (`BrowserRouter`) - 最外层

**职责**：全局路由管理，提供路由上下文给整个应用 

**为什么在外层**：所有组件都需要路由功能

### 2. **业务功能的容器** (`ExpenseProvider/TaskProvider`) - 中间层

**职责**：提供业务逻辑服务，管理特定功能域的状态 

**为什么在中间**：需要路由功能，但只给相关页面提供业务服务

### 3. **逻辑** (`Routes`)/ 业务函数(如`addExpense`) - 协调层

**职责**：Routes决定显示什么组件，业务函数协调服务调用 

**位置**：在对应容器内部

### 4. **表现层** (页面组件, `NotificationContainer`) - 最内层

**职责**：UI展示和用户交互 

**位置**：Routes决定显示哪个，NotificationContainer根据作用域放置

### 5. **具体服务** - 注入到容器中

**职责**：提供基础功能 

**位置**：不渲染在组件树中，而是作为依赖注入

### 6. **依赖注入** - 贯穿整个层次

**职责**：服务如何传递给使用者 

**方式**：通过props传入容器，通过Context提供给组件

## 标准的包裹顺序：

```txt
BrowserRouter (路由容器)                    // 最外层 - 全局路由
├── ExpenseProvider (业务容器)              // 中间层 - 业务功能  
│   ├── Routes (路由逻辑)                   // 协调层 - 决定显示
│   │   ├── Dashboard (页面组件)            // 最内层 - UI表现
│   │   ├── ExpenseList (页面组件)
│   │   └── AddExpense (页面组件)
│   └── NotificationContainer (UI服务)      // 根据作用域决定位置
├── 其他业务容器...
└── 全局UI服务 (如全局通知)
```



## 设计原则：

**外层** → **内层** = **通用** → **特定**

- 路由容器：所有页面都需要
- 业务容器：特定功能页面需要
- 页面组件：具体某个页面需要

**服务注入方向**：自上而下

> React Context 的工作原理是**向下传递**，外层组件提供的数据，内层组件都能访问

- 应用创建服务实例
- 传入业务容器
- 容器提供给页面组件使用



