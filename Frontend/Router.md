## 路由配置

在 React 项目中，路由配置通常有几种常见的方式：

### **集中式路由配置** - 所有路由都在 `src/App.jsx` 中定义

> **优点：** 所有路由都在一个地方，容易理解和维护 
>
> **缺点：** 文件可能变得很长，路由和组件耦合

```jsx
// src/App.jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { TaskProvider } from '@/contexts/TaskContext'
import { Toaster } from 'sonner'
import Layout from '@/layouts/Layout'
import SearchPage from '@/pages/SearchPage'
import BatchProcessPage from '@/pages/BatchProcessPage'
import ResultsPage from '@/pages/ResultsPage'
import HistoryPage from '@/pages/HistoryPage'
import SettingsPage from '@/pages/SettingsPage'

function App() {
  return (
    <BrowserRouter>
      <TaskProvider>
        <Routes>
          <Route path="/" element={<Layout />}>
            {/* 主页：SearchPage */}
            <Route index element={<SearchPage />} />
            
            {/* 批量处理流程（不在侧边栏显示，程序自动跳转） */}
            <Route path="batch/:taskId" element={<BatchProcessPage />} />
            <Route path="results/:taskId" element={<ResultsPage />} />
            
            {/* 工具页面（侧边栏导航） */}
            <Route path="history" element={<HistoryPage />} />
            <Route path="settings" element={<SettingsPage />} />
          </Route>
        </Routes>
        
        {/* 全局Toast通知 */}
        <Toaster position="top-right" />
      </TaskProvider>
    </BrowserRouter>
  )
}

export default App

```



### **分散式路由配置** - 每个页面组件自己定义子路由

> **优点：** 路由和组件紧耦合，每个组件管理自己的子路由 
>
> **缺点：** 路由结构分散，不容易全局查看

```jsx
// src/pages/Dashboard.jsx - 父页面组件
import { Outlet } from 'react-router-dom'

function Dashboard() {
  return (
    <div className="dashboard">
      <h1>仪表板</h1>
      {/* 子路由会在这里渲染 */}
      <Outlet />
    </div>
  )
}

// src/pages/Dashboard/Home.jsx - 子页面组件
function Home() {
  return <div>仪表板首页</div>
}

// src/App.jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import Dashboard from '@/pages/Dashboard'
import Home from '@/pages/Dashboard/Home'
import Settings from '@/pages/Dashboard/Settings'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Dashboard 组件管理自己的子路由 */}
        <Route path="/dashboard" element={<Dashboard />}>
          <Route index element={<Home />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Routes>
    </BrowserRouter>
  )
}
```

### **路由配置文件** - 创建单独的 `routes.js` 文件来管理路由配置

> **优点：** 路由和组件完全分离，易于维护和测试 
>
> **缺点：** 需要额外的配置文件

```javascript
// src/router/routes.js - 路由配置文件
import Layout from '@/layouts/Layout'
import SearchPage from '@/pages/SearchPage'
import BatchProcessPage from '@/pages/BatchProcessPage'
import ResultsPage from '@/pages/ResultsPage'
import HistoryPage from '@/pages/HistoryPage'
import SettingsPage from '@/pages/SettingsPage'

// 路由配置数组
export const routes = [
  {
    path: "/",
    element: <Layout />,
    children: [
      // index 路由 - 默认显示的子路由
      { index: true, element: <SearchPage /> },
      
      // 批量处理流程（程序自动跳转）
      { path: "batch/:taskId", element: <BatchProcessPage /> },
      { path: "results/:taskId", element: <ResultsPage /> },
      
      // 工具页面（侧边栏导航）
      { path: "history", element: <HistoryPage /> },
      { path: "settings", element: <SettingsPage /> },
    ]
  }
]

export default routes
```







