# 如何编写自定义 React Hook。

以 `TaskContext.tsx` 为例，分解如何创建类似的任务管理 Hook。

## 第一步：理解自定义 Hook 的概念

自定义 Hook 是 React 函数，用于封装可重用的逻辑。Hook 名称必须以 `use` 开头。

**基本结构：**
```tsx
function useCustomHook() {
  // Hook 逻辑
  return { /* 暴露的状态和方法 */ };
}
```

## 第二步：分析现有代码中的 Hook 使用

在 `TaskContext.tsx` 中，主要使用了：
- `useState`：状态管理
- `useCallback`：函数优化
- `useContext`：Context 访问

## 第三步：创建简单的任务管理 Hook

让我们从简化版本开始，创建一个 `useTasks` Hook：

### 步骤 1：定义类型和接口
```tsx
// 定义任务接口
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

// 定义 Hook 返回的接口
interface UseTasksReturn {
  tasks: Task[];
  addTask: (title: string) => void;
  toggleTask: (id: string) => void;
  removeTask: (id: string) => void;
}
```

> #### 1.理解 TypeScript 接口
>
> 接口（Interface）是 TypeScript 的核心特性，用于定义对象的形状和结构。它就像一个"合同"，规定了数据应该有什么属性和方法。
>
> #### 2.为什么需要 `Task` 接口
>
> ```tsx
> interface Task {
>   id: string;
>   title: string;
>   completed: boolean;
> }
> ```
>
> **设计逻辑：**
>
> 1. **`id: string`** - 唯一标识符
>    - **为什么需要？**：每个任务都需要唯一标识，用于查找、更新、删除操作
>    - **为什么是 string？**：字符串更灵活，可以使用时间戳、UUID 等
>
> 2. **`title: string`** - 任务标题
>    - **为什么需要？**：存储用户输入的任务内容
>    - **为什么是 string？**：任务标题通常是文本
>
> 3. **`completed: boolean`** - 完成状态
>    - **为什么需要？**：标记任务是否已完成，用于 UI 显示和过滤
>    - **为什么是 boolean？**：只有完成/未完成两种状态
>
> **这个接口的作用：**
> - 确保所有任务对象都有相同的结构
> - 提供类型检查，防止意外的属性访问
> - 提高代码可维护性
>
> #### 3.为什么需要 `UseTasksReturn` 接口
>
> ```tsx
> interface UseTasksReturn {
>   tasks: Task[];
>   addTask: (title: string) => void;
>   toggleTask: (id: string) => void;
>   removeTask: (id: string) => void;
> }
> ```
>
> **设计逻辑：**
>
> 1. **`tasks: Task[]`** - 任务数组
>    - **为什么需要？**：Hook 需要返回当前的所有任务状态
>    - **为什么是 Task[]？**：数组类型，包含多个 Task 对象
>
> 2. **`addTask: (title: string) => void`** - 添加任务方法
>    - **参数 title: string**：新任务的标题
>    - **返回值 void**：不需要返回值，任务会通过状态更新反映
>
> 3. **`toggleTask: (id: string) => void`** - 切换任务状态
>    - **参数 id: string**：要切换的任务 ID
>    - **为什么用 toggle？**：完成/未完成切换，更简洁的 API
>
> 4. **`removeTask: (id: string) => void`** - 删除任务
>    - **参数 id: string**：要删除的任务 ID
>
> **这个接口的作用：**
> - 定义 Hook 的"公共 API"
> - 确保使用 Hook 的组件知道可以调用什么方法
> - 提供类型提示，提高开发体验
>
> #### 4.接口的好处
>
> **类型安全：**
> ```tsx
> const { tasks, addTask } = useTasks();
> // TypeScript 知道 tasks 是 Task[] 类型
> // addTask 需要 string 参数，返回 void
> ```
>
> **防止错误：**
> ```tsx
> // 这会报错，因为 completed 应该是 boolean
> const task: Task = { id: '1', title: 'test', completed: 'yes' }; // ❌
> 
> // 这会报错，因为缺少必需属性
> const task: Task = { id: '1', title: 'test' }; // ❌
> ```
>
> **重构安全：**
> ```tsx
> // 如果改变 Task 接口，所有相关代码都会报错提醒你更新
> interface Task {
>   id: string;
>   title: string;
>   completed: boolean;
>   priority: 'low' | 'medium' | 'high'; // 新增属性
> }
> // 所有使用 Task 的地方都会报错，直到你更新它们
> ```
>
> #### 5.接口设计原则
>
> 1. **最小化原则**：只定义必需的属性
> 2. **扩展性**：使用联合类型支持未来扩展
> 3. **一致性**：保持命名和类型约定一致
> 4. **组合性**：可以通过扩展接口来创建变体
>
> **扩展示例：**
> ```tsx
> // 基础任务
> interface Task {
>   id: string;
>   title: string;
>   completed: boolean;
> }
> 
> // 带优先级的任务（扩展基础接口）
> interface PriorityTask extends Task {
>   priority: 'low' | 'medium' | 'high';
> }
> 
> // 带截止日期的任务
> interface DeadlineTask extends Task {
>   deadline: Date;
> }
> ```
>
> 通过精心设计的接口，你可以构建类型安全、可维护的代码结构。接口是 TypeScript 项目的基石，让你的代码更加健壮和可预测。

### 步骤 2：实现 Hook 主体

```tsx
import { useState, useCallback } from 'react';

export function useTasks(): UseTasksReturn {
  // 步骤 3：使用 useState 管理状态
  const [tasks, setTasks] = useState<Task[]>([]);

  // 步骤 4：使用 useCallback 优化函数性能
  const addTask = useCallback((title: string) => {
    const newTask: Task = {
      id: Date.now().toString(),
      title,
      completed: false
    };
    setTasks(prev => [...prev, newTask]);
  }, []);

  const toggleTask = useCallback((id: string) => {
    setTasks(prev => prev.map(task => 
      task.id === id ? { ...task, completed: !task.completed } : task
    ));
  }, []);

  const removeTask = useCallback((id: string) => {
    setTasks(prev => prev.filter(task => task.id !== id));
  }, []);

  // 步骤 5：返回状态和方法
  return {
    tasks,
    addTask,
    toggleTask,
    removeTask
  };
}
```

> #### 1.理解 `useState` 的作用
>
> ```tsx
> const [tasks, setTasks] = useState<Task[]>([]);
> ```
>
> **`useState` 是什么：**
> - React Hook，用于在函数组件中添加状态
> - 返回一个数组：`[当前状态值, 更新状态的函数]`
>
> **在这个 Hook 中的作用：**
> - **`tasks`**：存储所有任务的数组，是 Hook 的核心状态
> - **`setTasks`**：更新任务列表的函数
> - **初始值 `[]`**：空数组，因为应用启动时没有任务
>
> **为什么使用 `useState`：**
> - 任务列表会动态变化（添加、删除、修改）
> - 需要触发组件重新渲染来更新 UI
> - 状态必须在组件生命周期中保持
>
> #### 2.理解 `useCallback` 的作用
>
> ```tsx
> const addTask = useCallback((title: string) => { ... }, []);
> ```
>
> **`useCallback` 是什么：**
> - React Hook，用于缓存函数引用，避免不必要的重新创建
> - 语法：`useCallback(callback, deps)`
> - 当依赖数组不变时，返回相同的函数引用
>
> **在这个 Hook 中的作用：**
> - **性能优化**：防止传递给子组件的函数每次都重新创建
> - **引用稳定性**：确保函数在多次渲染中保持相同引用
> - **依赖数组 `[]`**：空数组表示函数永不重新创建
>
> #### 3.函数逻辑的确定过程
>
> ##### `addTask` 函数逻辑
>
> ```tsx
> const addTask = useCallback((title: string) => {
>   const newTask: Task = {
>     id: Date.now().toString(),    // 唯一 ID
>     title,                        // 用户输入的标题
>     completed: false              // 新任务默认未完成
>   };
>   setTasks(prev => [...prev, newTask]); // 添加到现有任务列表
> }, []);
> ```
>
> **逻辑推理：**
> 1. **ID 生成**：`Date.now().toString()` - 使用时间戳确保唯一性
> 2. **默认状态**：新任务应该是未完成的
> 3. **状态更新**：使用 `prev => [...prev, newTask]` 确保基于最新状态更新
> 4. **不可变更新**：不直接修改原数组，创建新数组
>
> ##### `toggleTask` 函数逻辑
>
> ```tsx
> const toggleTask = useCallback((id: string) => {
>   setTasks(prev => prev.map(task => 
>     task.id === id ? { ...task, completed: !task.completed } : task
>   ));
> }, []);
> ```
>
> **逻辑推理：**
> 1. **查找任务**：通过 `id` 找到要切换的任务
> 2. **切换逻辑**：`!task.completed` - 布尔值取反
> 3. **保持其他任务不变**：只修改匹配的任务
> 4. **对象展开**：`{ ...task, completed: !task.completed }` 更新完成状态
>
> ##### `removeTask` 函数逻辑
>
> ```tsx
> const removeTask = useCallback((id: string) => {
>   setTasks(prev => prev.filter(task => task.id !== id));
> }, []);
> ```
>
> **逻辑推理：**
> 1. **过滤逻辑**：保留 `id` 不等于要删除 ID 的任务
> 2. **返回新数组**：`filter` 创建新数组，不修改原数组
> 3. **简单直接**：删除操作不需要复杂逻辑
>
> #### 4.为什么使用这些特定的更新模式
>
> ##### 使用 `prev =>` 回调形式
>
> ```tsx
> setTasks(prev => [...prev, newTask])
> ```
>
> **原因：**
> - **并发安全**：确保基于最新状态更新
> - **避免竞态条件**：多个状态更新按顺序执行
> - **函数式更新**：符合 React 的不可变数据原则
>
> ##### 为什么不直接使用 `tasks`
>
> ```tsx
> // ❌ 不推荐
> setTasks([...tasks, newTask])
> ```
>
> **问题：**
> - `tasks` 可能不是最新的值（闭包陷阱）
> - 在异步操作中可能导致状态不一致
>
> #### 5.性能考虑
>
> **使用 `useCallback` 的好处：**
>
> - **减少重新渲染**：子组件接收相同函数引用，不会不必要重渲染
> - **依赖优化**：当 Hook 被其他组件使用时，避免传递新函数
> - **内存优化**：减少垃圾回收压力
>
> **空依赖数组 `[]` 的意义：**
>
> - 函数逻辑不依赖任何外部变量
> - 函数在整个组件生命周期中保持不变
> - 适合纯逻辑函数
>

## 第四步：在组件中使用自定义 Hook

```tsx
import React from 'react';
import { useTasks } from './useTasks'; // 导入自定义 Hook

function TaskApp() {
  const { tasks, addTask, toggleTask, removeTask } = useTasks();
  
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const title = (e.currentTarget.elements.namedItem('title') as HTMLInputElement).value;
    addTask(title);
    e.currentTarget.reset();
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input name="title" placeholder="添加新任务" />
        <button type="submit">添加</button>
      </form>
      
      <ul>
        {tasks.map(task => (
          <li key={task.id}>
            <input 
              type="checkbox" 
              checked={task.completed}
              onChange={() => toggleTask(task.id)}
            />
            <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
              {task.title}
            </span>
            <button onClick={() => removeTask(task.id)}>删除</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

> #### 1. 导入语句
>
> ```tsx
> import React from 'react';
> import { useTasks } from './useTasks'; // 导入自定义 Hook
> ```
>
> **逻辑解释：**
> - **`import React`**：在 JSX 中使用 React 语法
> - **`import { useTasks }`**：导入我们创建的自定义 Hook
> - **相对路径 `'./useTasks'`**：假设 Hook 在同一目录
>
> #### 2. 组件定义和 Hook 使用
>
> ```tsx
> function TaskApp() {
>   const { tasks, addTask, toggleTask, removeTask } = useTasks();
> ```
>
> **逻辑解释：**
> - **函数组件**：现代 React 推荐的组件写法
> - **解构赋值**：从 Hook 返回的对象中提取需要的属性
> - **命名清晰**：变量名直接反映功能
>
> ## 3. 表单提交处理
>
> ```tsx
> const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
>   e.preventDefault();
>   const title = (e.currentTarget.elements.namedItem('title') as HTMLInputElement).value;
>   addTask(title);
>   e.currentTarget.reset();
> };
> ```
>
> **逻辑分解：**
>
> 1. **事件类型注解**：
>    ```tsx
>    (e: React.FormEvent<HTMLFormElement>)
>    ```
>    - `React.FormEvent`：表单事件类型
>    - `<HTMLFormElement>`：泛型参数，指定事件目标是表单元素
>
> 2. **阻止默认行为**：
>    ```tsx
>    e.preventDefault();
>    ```
>    - 防止表单提交导致页面刷新
>    - SPA 应用中，表单提交通常由 JavaScript 处理
>
> 3. **获取输入值**：
>    ```tsx
>    const title = (e.currentTarget.elements.namedItem('title') as HTMLInputElement).value;
>    ```
>    - `e.currentTarget`：指向绑定事件的元素（表单）
>    - `elements.namedItem('title')`：通过 name 属性获取输入框
>    - `as HTMLInputElement`：TypeScript 类型断言
>    - `.value`：获取输入框的值
>
> 4. **调用 Hook 方法**：
>    ```tsx
>    addTask(title);
>    ```
>    - 传递用户输入的任务标题
>
> 5. **重置表单**：
>    ```tsx
>    e.currentTarget.reset();
>    ```
>    - 清空输入框，准备下一次输入
>
> #### 4. JSX 结构分析
>
> #### 表单部分
>
> ```tsx
> <form onSubmit={handleSubmit}>
>   <input name="title" placeholder="添加新任务" />
>   <button type="submit">添加</button>
> </form>
> ```
>
> **逻辑解释：**
>
> - **`onSubmit={handleSubmit}`**：绑定提交事件处理函数
> - **`name="title"`**：为输入框命名，便于在事件处理中访问
> - **`placeholder`**：提供用户提示
> - **`type="submit"`**：按钮类型，点击时触发表单提交
>
> ### 任务列表部分
>
> ```tsx
> <ul>
>   {tasks.map(task => (
>     <li key={task.id}>
>       <input 
>         type="checkbox" 
>         checked={task.completed}
>         onChange={() => toggleTask(task.id)}
>       />
>       <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
>         {task.title}
>       </span>
>       <button onClick={() => removeTask(task.id)}>删除</button>
>     </li>
>   ))}
> </ul>
> ```
>
> **逻辑分解：**
>
> 1. **列表渲染**：
>    ```tsx
>    {tasks.map(task => (...))}
>    ```
>    - 对任务数组进行遍历
>    - 为每个任务生成对应的 JSX
>
> 2. **Key 属性**：
>    ```tsx
>    <li key={task.id}>
>    ```
>    - React 要求列表项有唯一 key
>    - 用于优化渲染性能和状态管理
>
> 3. **复选框**：
>    ```tsx
>    <input 
>      type="checkbox" 
>      checked={task.completed}
>      onChange={() => toggleTask(task.id)}
>    />
>    ```
>    - `checked`：受控组件，与状态同步
>    - `onChange`：状态改变时调用 toggleTask
>
> 4. **任务标题**：
>    ```tsx
>    <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
>      {task.title}
>    </span>
>    ```
>    - 条件样式：完成的任务显示删除线
>    - 内联样式：简单条件渲染
>
> 5. **删除按钮**：
>    ```tsx
>    <button onClick={() => removeTask(task.id)}>删除</button>
>    ```
>    - 点击时调用 removeTask，传递任务 ID
>
> ## 5. React 最佳实践体现
>
> 1. **受控组件**：输入框和复选框都与状态同步
> 2. **事件处理**：使用箭头函数或绑定方法
> 3. **条件渲染**：基于状态显示不同样式
> 4. **不可变更新**：通过 Hook 管理状态变化
> 5. **类型安全**：TypeScript 提供类型检查
>
> ## 6. 可能的改进点
>
> ```tsx
> // 可以添加输入验证
> const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
>   e.preventDefault();
>   const title = (e.currentTarget.elements.namedItem('title') as HTMLInputElement).value.trim();
>   if (title) {  // 防止空任务
>     addTask(title);
>     e.currentTarget.reset();
>   }
> };
> 
> // 可以添加任务过滤
> const completedTasks = tasks.filter(task => task.completed);
> const pendingTasks = tasks.filter(task => !task.completed);
> ```
>
> 这个组件展示了 React 函数组件、Hook 使用、事件处理、条件渲染等核心概念的综合应用。

## 第五步：理解 Hook 规则

1. **只在 React 函数组件或自定义 Hook 中调用**
2. **只在顶层调用，不能在循环、条件或嵌套函数中**
3. **Hook 名称必须以 `use` 开头**

## 第六步：扩展到 Context 版本

如果你想要全局状态，可以结合 Context：

```tsx
// 创建 Context
const TasksContext = createContext<UseTasksReturn | undefined>(undefined);

// Provider 组件
export function TasksProvider({ children }: { children: React.ReactNode }) {
  const tasksLogic = useTasks(); // 使用自定义 Hook
  
  return (
    <TasksContext.Provider value={tasksLogic}>
      {children}
    </TasksContext.Provider>
  );
}

// 访问 Hook
export function useTasksContext() {
  const context = useContext(TasksContext);
  if (!context) {
    throw new Error('useTasksContext must be used within TasksProvider');
  }
  return context;
}
```

> ## 1. 理解 Context 的作用
>
> Context 是 React 提供的全局状态管理机制，用于在组件树中共享数据，避免"props drilling"（逐层传递 props）。
>
> ## 2. 创建 Context
>
> ```tsx
> const TasksContext = createContext<UseTasksReturn | undefined>(undefined);
> ```
>
> **逻辑解释：**
> - **`createContext`**：React 函数，创建一个 Context 对象
> - **泛型参数 `<UseTasksReturn | undefined>`**：
>   - `UseTasksReturn`：Context 中存储的数据类型
>   - `| undefined`：允许初始值为 undefined（未提供 Provider 时）
> - **初始值 `undefined`**：表示默认情况下没有提供 Context
>
> **为什么需要泛型：**
> - TypeScript 类型安全，确保 Context 中的数据类型正确
> - 开发时提供智能提示和错误检查
>
> #### 3. TasksProvider 组件
>
> ```tsx
> export function TasksProvider({ children }: { children: React.ReactNode }) {
>   const tasksLogic = useTasks(); // 使用自定义 Hook
>   
>   return (
>     <TasksContext.Provider value={tasksLogic}>
>       {children}
>     </TasksContext.Provider>
>   );
> }
> ```
>
> **逻辑分解：**
>
> 1. **组件签名**：
>    ```tsx
>    ({ children }: { children: React.ReactNode })
>    ```
>    - `children`：React 内置 prop，包含子组件
>    - `React.ReactNode`：可以是任何 React 可渲染内容
>
> 2. **使用自定义 Hook**：
>    ```tsx
>    const tasksLogic = useTasks();
>    ```
>    - 获取任务管理的所有逻辑和状态
>    - 这是"逻辑复用"的关键
>
> 3. **Provider 组件**：
>    ```tsx
>    <TasksContext.Provider value={tasksLogic}>
>    ```
>    - `Provider`：Context 的提供者组件
>    - `value`：要共享给子组件的数据
>
> 4. **渲染子组件**：
>    ```tsx
>    {children}
>    ```
>    - 渲染所有被包裹的子组件
>    - 这些子组件现在可以访问 Context
>
> #### 4. 访问 Context 的 Hook
>
> ```tsx
> export function useTasksContext() {
>   const context = useContext(TasksContext);
>   if (!context) {
>     throw new Error('useTasksContext must be used within TasksProvider');
>   }
>   return context;
> }
> ```
>
> **逻辑分解：**
>
> 1. **使用 useContext**：
>    ```tsx
>    const context = useContext(TasksContext);
>    ```
>    - `useContext`：React Hook，用于消费 Context
>    - 返回 Provider 提供的 value
>
> 2. **错误检查**：
>    ```tsx
>    if (!context) {
>      throw new Error('useTasksContext must be used within TasksProvider');
>    }
>    ```
>    - 检查是否在 Provider 内部使用
>    - 提供清晰的错误信息，帮助开发者调试
>
> 3. **返回数据**：
>    ```tsx
>    return context;
>    ```
>    - 返回完整的任务管理接口
>
> #### 5. 完整使用流程
>
> ##### 应用根组件
>
> ```tsx
> // App.tsx
> import { TasksProvider } from './contexts/TasksContext';
> 
> function App() {
>   return (
>     <TasksProvider>
>       <TaskApp />
>       <TaskStats />
>       <TaskFilters />
>     </TasksProvider>
>   );
> }
> ```
>
> ##### 子组件使用
>
> ```tsx
> // TaskApp.tsx
> import { useTasksContext } from './contexts/TasksContext';
> 
> function TaskApp() {
>   const { tasks, addTask, toggleTask, removeTask } = useTasksContext();
>   // 使用任务管理逻辑
> }
> 
> // TaskStats.tsx
> function TaskStats() {
>   const { tasks } = useTasksContext();
>   const completedCount = tasks.filter(t => t.completed).length;
>   // 显示统计信息
> }
> ```
>
> #### 6. 设计模式的好处
>
> ##### 1. 避免 Props Drilling
>
> **问题场景：**
> ```tsx
> // 嵌套多层传递 props
> <App>
>   <Page>
>     <TaskList tasks={tasks} onToggle={onToggle} />
>   </Page>
> </App>
> ```
>
> **Context 解决方案：**
> ```tsx
> // 任意层级直接访问
> function TaskList() {
>   const { tasks, toggleTask } = useTasksContext();
> }
> ```
>
> ##### 2. 逻辑复用
>
> - 自定义 Hook 封装业务逻辑
> - Context 提供全局访问
> - 组件只需关注 UI 渲染
>
> ##### 3. 类型安全
>
> - 泛型确保数据类型正确
> - 错误检查防止 misuse
> - IDE 提供完整的智能提示
>
> ##### 4. 测试友好
>
> ```tsx
> // 测试时可以单独测试 Hook
> const { result } = renderHook(() => useTasks(), {
>   wrapper: TasksProvider
> });
> ```
>
> #### 7. 最佳实践
>
> 1. **单一职责**：每个 Context 管理一个关注点
> 2. **错误边界**：考虑添加错误边界处理 Context 错误
> 3. **性能优化**：使用 `useMemo` 避免不必要的重新渲染
> 4. **组合使用**：可以嵌套多个 Context
>
> 通过这种模式，你创建了一个可扩展、类型安全、易于测试的全局状态管理系统。

## 第七步：测试你的 Hook

```tsx
import { renderHook, act } from '@testing-library/react';
import { useTasks } from './useTasks';

test('should add a task', () => {
  const { result } = renderHook(() => useTasks());
  
  act(() => {
    result.current.addTask('新任务');
  });
  
  expect(result.current.tasks).toHaveLength(1);
  expect(result.current.tasks[0].title).toBe('新任务');
});
```

现在你可以尝试创建自己的 Hook！从简单的开始，逐步添加复杂的功能。关键是要理解 Hook 的生命周期和依赖关系。