## 为什么选择

Vite 是一个现代化的前端构建工具，由 Vue.js 的作者 Evan You 开发，主要用于快速构建和开发前端项目。与传统的打包工具（如 Webpack）不同，Vite 专注于开发体验的优化。

**Vite 的主要特点：**
1. **快速冷启动**：使用 ES 模块原生支持，无需打包即可启动开发服务器
2. **热模块替换 (HMR)**：修改代码后瞬间更新浏览器，无需刷新页面
3. **按需编译**：只编译当前使用的模块，提高构建速度
4. **多框架支持**：不仅支持 Vue，还支持 React、Svelte、Vanilla JS 等

## **为什么要引用 Vite 的客户端类型定义？**

> `/// <reference types="vite/client" />`

在前端项目中使用 TypeScript 时，需要为代码中的变量、函数等提供类型信息，以确保类型安全和更好的开发体验。

Vite 提供了一些特殊的全局 API，这些不是标准 JavaScript/TypeScript 的一部分：

- `import.meta.env`：访问环境变量（如 `VITE_API_URL`）
- `import.meta.hot`：热重载相关的 API
- `import.meta.glob`：动态导入多个模块
- `import.meta.url`：获取当前模块的 URL

这些 API 的类型定义包含在 Vite 的 `@types/vite/client` 包中。通过在 `vite-env.d.ts` 文件中添加 `/// <reference types="vite/client" />`，TypeScript 编译器就能识别这些 Vite 特有的 API 的类型，避免类型错误并提供代码提示。

没有这个引用，TypeScript 会把 `import.meta.env` 等当作 `any` 类型，无法提供类型检查和智能提示。

----------------------------------------------------------------------------------------------------

## 环境配置

> 目的：让前端应用能够无缝适配不同的运行环境（本地/生产）

**1. `import.meta.env` - Vite 环境变量 API**

- Vite 特有的全局对象，用于访问环境变量
- 替代了传统 webpack 的 `process.env`
- 只在构建时注入实际值，开发时可访问所有变量

**2. 环境变量命名约定**
- `VITE_` 前缀：只有以 `VITE_` 开头的环境变量才会被 Vite 暴露给客户端代码
- 安全考虑：防止敏感信息意外暴露
- 示例：`VITE_API_BASE_URL`、`VITE_ENABLE_DEBUG`

**3. 运行时环境检测**
- `typeof window !== 'undefined'`：检查是否在浏览器环境中运行
- 兼容性处理：Vite 支持 SSR（服务端渲染），`import.meta.env` 在服务器端不存在
- 条件执行：确保代码只在客户端执行

**4. 测试环境兼容性**
- `try/catch` 块：处理 Jest 等测试环境
- Jest 不支持 `import.meta`，需要特殊配置或 mock
- 降级策略：在测试环境中使用默认配置

**5. TypeScript 类型处理**
- `@ts-ignore`：临时忽略 TypeScript 类型检查
- 原因：`import.meta.env` 的类型定义可能不完整或测试环境兼容性问题

**完整工作流程：**
1. 开发时：从 `.env` 文件读取 `VITE_*` 变量
2. 构建时：Vite 将变量注入 `import.meta.env`
3. 运行时：代码检查环境并读取配置
4. 测试时：降级到默认值，避免测试失败

这种模式确保了配置的灵活性和环境兼容性。