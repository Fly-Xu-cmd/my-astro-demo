---
title: React学习笔记
author: 徐飞
description: "学习了一些 React 后，我根本停不下来！"
image:
    url: "https://docs.astro.build/assets/arc.webp"
    alt: "The Astro logo on a dark background with a purple gradient arc."
pubDate: 2022-07-08
tags: ["react", "blogging", "learning in public", "successes"]
---

### 打包优化
#### 路由懒加载
##### 定义
路由懒加载是指路由的JS资源只有在被访问时才会动态获取，目的是为了优化项目首次打开的时间
##### 配置方法
1. 把路由修改为由React提供的lazy函数进行动态导入
2. 使用React内置的Suspense组件包裹路由中的element选项对应的组件
```JavaScript
// 假设拥有一个Home组件
// 导入相关包
import { createBrowserRouter} from 'react-router-dom'
import {lazy,Suspense} from 'react'
// 1. lazy函数对组件进行导入
const Home = lazy(()=>import('@/pages/Home'))

// 配置路由实例
const router = createBrowserRouter([
	{
		path:"/home",
		// 异步渲染<Home />
		element:<Suspense fallback={'加载中'}><Home /></Suspense>
	}
])
```
#### 包体积分析
```bash
// npm下载第三方插件
$ npm i source-map-explorer
// 使用插件命令指定分析的文件（以build/static/js路径下的所有js文件为例）
$ source-map-explorer 'build/static/js/*.js'
```
#### CDN优化
##### 什么是CDN？
CDN是一种内容分发网络服务，当用户请求网站内容时，由离用户最近的服务器将缓存的资源内容传递给用户

##### 哪些资源可以放到CDN服务器？
体积较大的非业务JS文件，比如react、react-dom
1. 体积较大，需要利用CDN文件在浏览器的缓存特性，加快加载时间
2. 非业务JS文件，不需要经常做变动，CDN不用频繁更新缓存

##### 项目中怎么做？
1. 把需要做CDN缓存的文件排除在打包之外（react、react-dom）
2. 以CDN的方式重新引入资源（react、react-dom）

**分析说明**：通过 craco 来修改 webpack 配置，从而实现 CDN 优化

**核心代码** `craco.config.js`
```js
// 添加自定义对于webpack的配置  
​  
const path = require('path')  
const { whenProd, getPlugin, pluginByName } = require('@craco/craco')  
​  
module.exports = {  
  // webpack 配置  
  webpack: {  
    // 配置别名  
    alias: {  
      // 约定：使用 @ 表示 src 文件所在路径  
      '@': path.resolve(__dirname, 'src')  
    },  
    // 配置webpack  
    // 配置CDN  
    configure: (webpackConfig) => {  
      let cdn = {  
        js:[]  
      }  
      whenProd(() => {  
        // key: 不参与打包的包(由dependencies依赖项中的key决定)  
        // value: cdn文件中 挂载于全局的变量名称 为了替换之前在开发环境下  
        webpackConfig.externals = {  
          react: 'React',  
          'react-dom': 'ReactDOM'  
        }  
        // 配置现成的cdn资源地址  
        // 实际开发的时候 用公司自己花钱买的cdn服务器  
        cdn = {  
          js: [  
            'https://cdnjs.cloudflare.com/ajax/libs/react/18.1.0/umd/react.production.min.js',  
            'https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.1.0/umd/react-dom.production.min.js',  
          ]  
        }  
      })  
​  
      // 通过 htmlWebpackPlugin插件 在public/index.html注入cdn资源url  
      const { isFound, match } = getPlugin(  
        webpackConfig,  
        pluginByName('HtmlWebpackPlugin')  
      )  
​  
      if (isFound) {  
        // 找到了HtmlWebpackPlugin的插件  
        match.userOptions.files = cdn  
      }  
​  
      return webpackConfig  
    }  
  }  
}
```

```js
// public/index.html

<body>  
  <div id="root"></div>  
  <!-- 加载第三发包的 CDN 链接 -->  
  <% htmlWebpackPlugin.options.files.js.forEach(cdnURL => { %>  
    <script src="<%= cdnURL %>"></script>  
  <% }) %>  
</body>
```


### useState
#### 更新方式
##### 基本更新方法
```js
// 例：
const [count,setCount] = useState(0);
setCount(1);
```
##### 函数式更新
```js
// 例：当新得state需要依赖先前得state时，可以传入一个函数
const [count,setCount] = useState(0);
setCount(prevCount => prevCount + 1);
```

##### 惰性初始 state

```js
// 例：如果初始state需要通过复杂计算获得，可以传入一个函数
const [state,setState] = useState(() => {
	const initialState = someExpensiveComputation(props);
	return initialState;
})
```
### useReducer
#### 基础用法
1. 定义一个redcucer函数（根据不同的action返回不同的新状态）
2. 在组件中调用useReducer，并传入reducer函数和状态的初始值
3. 事件发生时，通过dispatch函数分派一个action对象（通知reducer要返回那个新状态并渲染UI）
**加减器为例**
```js
function reducer(state,action){
	// 根据不同的action type 返回新的state
	switch(action.type){
		case 'INC':
			return state + 1
		case 'DEC':
			return state - 1
		case 'SET':
			return action.payload
		default:
			return state
	}
}

const [state,dispatch] = useReducer(reducer,0);

dispatch({type:'INC'})
```
##### 分派action时传参
```js
dispatch({
	type: 'SET',
	payload: 100
})
```
### useMemo
#### 作用
在组件每次重新渲染的时候缓存计算的结果
#### 基础语法
```js
// 使用useMemo做缓存之后可以保证只有count1依赖项发生变化时才会重新计算
useMemo(()=>{
// 根据cout1返回计算的结果

},[count1])
```
### React.memo
#### 作用
允许组件在**Props没有改变**的情况下跳过渲染
React组件默认的渲染机制：只要父组件重新渲染子组件就会重新渲染
#### 基础语法
```js
// 经过memo函数包裹生成的缓存组件只有在props发生变化的时候才会重新渲染
const MemoComponent = memo(function SomeComponent(props){
	// ..
})
```
#### props的比较机制
**机制**：在使用memo缓存组件之后，React会对每一个prop使用Object.js比较新值和老值，返回true，表示没有变化

**prop是简单类型**

Object.is(3,3)=>true 没有变化

**prop是引用类型（对象/数组）**

Object([],[])=>false 有变化，React只关心引用是否变化
### useCallback
#### 作用
在组件多次重新渲染的时候缓存函数
#### 基础语法
```js
// 使用useCallback包裹函数之后，函数可以保证在App重新渲染的时候保持引用稳定
useCallback((value)=>console.log(value),[])
```
### React.forwardRef
#### 作用
使用ref暴露DOM节点给父组件
#### 语法实现
```js
// 子组件
const Input = forwardRef((props,ref)=>{
	return <input type="text" ref={ref} />
})

// 父组件
function App(){
	const inputRef = useRef(null)
	return (
		<>
			<Input ref={inputRef} />
			<button onClick={focusHandler}>focus</button>
		</>
	)
}
```

### useInperativeHandle
#### 作用
通过ref暴露子组件中的方法
#### 语法实现
```js
// 子组件
const Son = forwardRef((props,ref)=>{
	// 实现聚焦逻辑
	const inputRef = useRef(null)
	const focusHandler = () => {
		inputRef.current.focus()
	}
	// 把聚焦方法暴露出去
	useImperativeHandle(ref,() => {
		return {
			// 暴露的方法
			focusHandler
		}
	})
	return <input type="text" ref={inputRef} />
})

// 父组件
function App(){
	const sonRef = useRef(null)
	const focusHandler = () => {
		sonRef.current.focusHandler()
	}
	return (
		<>
			<Son ref={sonRef} />
			<button onClick={focusHandler}>focus</button>
		</>
	)
}
```
### React.Fragment
#### 作用
`React.Fragment` 是 React 提供的一个特殊组件，允许你将多个子元素分组，而无需在 DOM 中添加额外的节点。

#### 核心特性

1. ​**无 DOM 节点**​：不会在最终渲染的 HTML 中创建实际元素
2. ​**轻量级**​：比常规 DOM 元素更高效
3. ​**满足 JSX 单根要求**​：JSX 必须返回单个根元素，Fragment 提供了解决方案
4. **简写**：`<></>`
5. **属性**：只支持**key**属性
### zustand
#### 快速上手

##### Step 1: 安装

```bash
npm install zustand # or yarn add zustand
```

##### Step 2: Store 初始化

创建的 store 是一个 `hook`，你可以放任何东西到里面：基础变量，对象、函数，状态必须不可改变地更新，`set` 函数合并状态以实现状态更新。

```jsx
import { create } from 'zustand'

const useBearStore = create((set) => ({
	bears: 0,  
	increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),  
	removeAllBears: () => set({ bears: 0 }),
}))
```

##### Step 3: Store 绑定组件，就完成了!

可以在任何地方使用钩子，不需要提供 `provider`。  
基于 `selector` 获取您的目标状态，组件将在状态更改时重新渲染。

###### 选择目标状态：bears

```jsx
function BearCounter() {
	const bears = useBearStore((state) => state.bears)  
	return <h1>{bears} around here ...</h1>
}
```

###### 更新目标状态：bears

```jsx
function Controls() {  
	const increasePopulation = useBearStore((state) => state.increasePopulation)  
	return <button onClick={increasePopulation}>one up</button>
}
```


### SWR
#### SWR 是什么
SWR 是一个用于 React 的数据获取库，由 Vercel 团队开发。SWR 这个名字来自于 stale-while-revalidate ，这是一种 HTTP 缓存失效策略。

#### 核心特性
##### 1. Stale-While-Revalidate 策略
- 首先返回缓存中的数据（stale）
- 然后在后台发送请求验证数据（revalidate）
- 最后用最新数据更新缓存
##### 2. 自动重新验证
- 窗口重新获得焦点时
- 网络重新连接时
- 组件重新挂载时
- 定时轮询
##### 3. 内置功能
- 请求去重
- 错误重试
- 分页支持
- 本地缓存
- 乐观更新
#### 基本用法
```js
import useSWR from 'swr'

function Profile() {
  const { data, error, isLoading } = 
  useSWR('/api/user', fetcher)

  if (error) return <div>加载失败</div>
  if (isLoading) return <div>加载中...</
  div>
  return <div>你好 {data.name}!</div>
}
```