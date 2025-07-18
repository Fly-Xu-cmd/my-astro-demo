---
title: 'next.js学习笔记'
pubDate: 2024-07-018
description: '这是我next.js的学习笔记。'
author: '徐飞'
image:
    url: 'https://docs.astro.build/assets/rose.webp'
    alt: 'The Astro logo on a dark background with a pink glow.'
tags: ["astro", "blogging", "learning in public"]
---

## 构建Next.js项目
Next.js 提供了一个官方的脚手架工具 create-next-app，用于快速搭建项目。

create-react-app 是来自于 Facebook，通过该命令我们无需配置就能快速构建 Next.js 开发环境。

create-react-app 自动创建的项目是基于 Webpack + ES6 。

执行以下命令创建项目：

```
npx create-next-app@latest my-next-app
```

安装时，您将看到以下提示，一路回车即可：
```
Would you like to use TypeScript? No / Yes
Would you like to use ESLint? No / Yes
Would you like to use Tailwind CSS? No / Yes
Would you like your code inside a `src/` directory? No / Yes
Would you like to use App Router? (recommended) No / Yes
Would you like to use Turbopack for `next dev`?  No / Yes
Would you like to customize the import alias (`@/*` by default)? No / Yes
What import alias would you like configured? @/*
```


npx 是一个 Node.js 工具，用于运行包中的二进制文件，而无需全局安装。

- 使用 `create-next-app` 脚手架工具创建一个新的 Next.js 项目。
- `my-next-app` 是项目的名字，你可以根据需要修改为任何名称。
- 它会自动安装项目所需的依赖。

运行该命令后，create-next-app 会自动完成以下操作：

- 创建一个名为 my-next-app 的文件夹。
- 初始化项目，安装必要的依赖。
- 创建基本的项目结构，包括 app 文件夹、public 文件夹、package.json 文件等。

![项目目录|250](https://www.runoob.com/wp-content/uploads/2025/02/646a5876-b3cd-4c98-bd34-936f2dd6cb49.png)

创建完成后，进入项目目录：
```
cd my-next-app
```


在项目目录中，运行以下命令启动开发服务器：
```
npm run dev
```

## 简介（与React关系）
React 是一个用于构建用户界面的 JavaScript 库，而 Next.js 是在 React 上构建的框架。

React 关注于构建 UI 组件，而 Next.js 提供了更多的功能和结构，帮助开发者解决一些在 React 中较为繁琐的开发问题，如路由、数据获取、页面渲染等。

### 主要特性
- **文件系统路由：** Next.js 使用文件系统来自动化路由的创建。你只需要在 `pages`（老版）|`app`（Next.js13后推荐使用） 目录下创建文件，它就会自动映射为相应的路由，不需要额外的路由配置。
- **静态生成（SSG）与服务端渲染（SSR）：** Next.js 支持这两种渲染方式，可以根据需要灵活选择。静态生成适用于大多数情况，尤其是内容不会频繁变化的页面，而 SSR 适用于需要动态获取数据的页面。
- **API 路由：** Next.js 允许你在应用中直接创建 API 路由，可以在 `pages/api`|`app/api` 目录下轻松创建后端 API 端点，处理前后端逻辑。
- **自动代码拆分：** 每个页面只会加载它所需的 JavaScript 代码，确保应用启动速度更快，减少不必要的资源消耗。
- **优化图片：** Next.js 内置了图片优化功能，使用 `next/image` 组件可以自动为图像选择最佳格式、压缩、懒加载等，以提升页面加载性能。
- **支持 TypeScript：** Next.js 默认支持 TypeScript，可以让开发者在开发过程中享受更强的类型检查。

## app Router VS pages Router
#### 路由规则
##### app Router规则
- app Router按照文件夹名称构建路由
- 约定每一个路由下面必须有page.js文件
- layout.js文件用于定制布局，如果没有这个文件和父级路由的布局相同，通过layout.js文件可实现嵌套布局
- 动态路由参数约定文件夹名称[params],在该动态路由文件夹page.js文件中通过params参数获取
- 约定@filename文件夹时插槽文件，不生成路由，一般用于实现平行路由 
##### pages Router规则
- pages Router按照js文件名字构建路由，每一个js组件文件都是一个路由
- 通过getStaticProps，getServerSideProps，getStaticPaths获取数据

#### 客户端和服务端组件（Client and server components）
##### 客户端组件：

- 浏览器API
- 事件监听器
- 所有 React 钩子
- 非常适合在客户端生成一堆 HTML

##### 服务器组件：

- 非常适合隐藏代码和秘密
- 不要传送大部分依赖项
- 直接访问后台
- 完全集成服务器操作
- 有利于seo

Next.js默认使用服务器组件，如果要使用客户端组件需要在文件开头显示声明(`"use Client"`)

#### 布局更简单
我已经提到了layout.js 文件，它可以位于每个路径的目录中。 该组件使布局变得简单，因为路径组件会自动应用于提供的布局。 让我们看一个例子。

在我们选择的路径目录中，我们创建一个 layout.js：

```tsx
// layout.js

export default function LoginLayout({ children }) {
  return <div className='login-area'>{children}</div>
}
```

它所需要做的就是渲染一个自动传递的子组件 - 该子组件是 page.js 组件。  
page.js 完全取决于我们。 由于布局是自动应用的，因此我们不需要在此文件中指定引用任何内容。