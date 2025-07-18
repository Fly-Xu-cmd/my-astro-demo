---
title: webpack学习笔记
author: 徐飞
description: "我遇到了一些问题，但是在社区里面提问真的很有帮助！"
image:
    url: "https://docs.astro.build/assets/rays.webp"
    alt: "The Astro logo on a dark background with rainbow rays."
pubDate: 2022-07-15
tags: ["webpack", "learning in public", "setbacks", "community"]
---
## 解决的问题

- 将开发流程中的模块化开发打包成一个js文件供浏览器执行
- 将开发流程中的ts，es6和vue等新语法翻译为浏览器可识别的js语法

## 配置项

- **entry**：必须项，以哪个文件为开始
- **output**：必须项，最终产出js配置
- **mode**：webpack4以后必填
- **devServer**：非必须，开发模式配置
- **module**：非必须，loader编写的地方
- **plugins**：非必须，插件
- **optimization**：非必须，优化相关
- **resolve**：非必须，提供一些简化功能

```js
// commonjs
module.exports={
	//entry:["./app.js","./app2.js"]
	entry:{
		app:"./app.js",
	},
	output:{
		path:__dirname+"/dist",// 要求绝对路径
		filename:"[name].[hash:4].bundle.js"// name为入口文件的名字，hash:4表示取哈希值的4位
	}
}
```

## 处理js

### 流程
- **es6转化**：babel-loader
- **代码规范**：eslint
- **代码的分割和打包**：Webpack的自身核心功能

#### babel-loader
##### 安装

`npm install babel-loader @babel/core --save-dev`
>[!Tip]
>babel-loader只是提供一个调用@babel/core的接口，真正发挥作用的是@babel/core

##### loader的使用方法

```js
module:{
	rules:[
		{
			test: /\.js$/,//表示对所有js文件进行处理
			// 写法1:
			// loader:'babel-loader',
			// 写法2：
			// use:['babel-loader',]
			// 写法3：
			use:{
				loader: "babel-loader",
				// preset->@babel/preset-env（preset相当于编译规则集）
				options: {
					// 对babel-loader的一些常用配置
					presets:[
						[
							'@babel/preset-env',
							{
								targets:{
									browsers:[
										">1%",// 所有占有率大于1%的浏览器
										"last 2 versions",// 支持浏览器厂商最后两个版本 
										"not ie<=8",// 不支持IE8及之前的版本
									]
								}
							}
						]
					]
				}
			}
		}
	]
}
```

>[!Tip]
> - 写法1的loader：后面只能跟字符串
> - 写法2的数组里面可以写多个loader，处理顺序是从后往前，先用数组后面的loader处理文件
> - 写法3可以对loader进行一些配置
> - 如果options里面没有presets则babel-loader不起作用，因为它不知道怎么用什么规则去处理js文件
> - 安装常用的规则集->preset-env `npm install @babel/preset-env --save-dev`

##### .babelrc文件的作用
**用来书写babel-loader配置中的presets部分等效于直接在babel-loader的options中写presets**
```js
//使用.babelrc文件简化webpack.config.js文件的modul部分代码
// webpack.config.js
module.exports={
	//entry:["./app.js","./app2.js"]
	entry:{
		app:"./app.js",
	},
	output:{
		path:__dirname+"/dist",// 要求绝对路径
		filename:"[name].[hash:4].bundle.js"// name为入口文件的名字，hash:4表示取哈希值的4位
	},
	module:{
		rules:[
			{
				test: /\.js$/,//表示对所有js文件进行处理
				// 写法1:
				// loader:'babel-loader',
				// 写法2：
				// use:['babel-loader',]
				// 写法3：
				use:{
					loader: "babel-loader",
					// preset->@babel/preset-env（preset相当于编译规则集）
					options: {
						// 对babel-loader的一些常用配置
					}
				}
			}
		]
	}
}
// .babelrc
{
	 "presets":[
		[
			"@babel/preset-env",
				{
					"targets":{
					"browsers":[
						">1%",// 所有占有率大于1%的浏览器
						"last 2 versions",// 支持浏览器厂商最后两个版本 
						"not ie<=8",// 不支持IE8及之前的版本
					]
				}
			}
		]
	]
}

```
>[!Tip]
>.babelrc文件必须为JSON格式

#### eslint
##### 安装插件
`npm install eslint eslint-webpack-plugin --save-dev`
##### 使用方法
```js
// 下载完eslint插件后在这里注册一下
const eslintplugin=require("eslint-webpack-plugin");
module.exports={
	//entry:["./app.js","./app2.js"]
	entry:{
		app:"./app.js",
	},
	output:{
		path:__dirname+"/dist",// 要求绝对路径
		filename:"[name].[hash:4].bundle.js"// name为入口文件的名字，hash:4表示取哈希值的4位
	},
	plugins:[
		new eslintplugin();//可以在括号里面直接写配置对象,但是推荐在.eslintrc.js文件中编写规则
	]
}
```
##### .eslintrc.js文件
```js
module.exports={
	env:{
		browser:true,//浏览器环境，可使用window全局对象
		es2021:true
	},
	// 两个常用的eslint规范包
	// eslint-config-standard
	// eslint-config-airbnb
	// extends继承别的规范包，节省自己写规则的时间
	extends:[
		"standard",// 继承eslint-config-standard规范包,安装语句:npm install eslint-config-standard --save-dev
		"plugin:vue/strongly-recommended"// 继承vue插件包里面的规范包
	],
	// vue
	// 额外的rules+提供一套现成的规范
	plugins: [
		"vue",// 安装语句：npm install eslint-plugin-vue --save-dev
	],
	parserOptions:{
		ecmaVersion:6,
		sourceType: "module",// 支持import
		ecmaFeature:{
			jsx: true,// 支持检查jsx语法
		}
	},
	rules: {
		// 书写具体规则，可以去官网查看
	}
}
```

>[!Tip]
>- eslint本身不包括规范，需要向里面填充具体的规则才能发挥作用
>- 继承的规则和插件可以被自己的rules覆盖

## css处理
##### 安装
`npm install css-loader style-loader mini-css-extract-plugin --save-dev`
>[!Tip]
>- css-loader让webpack认识css文件
>- style-loader规定了css的处理方式：加入js中最后作为style标签插入html文件中
>- mini-css-extract-plugin规定了css的处理方式：生成单独的css文件，适用于大项目

##### 配置插件
```js
const minicss = require("mini-css-extract-plugin")
const minimizer = require("css-minimizer-webpack-plugin");
module.exports={
	//entry:["./app.js","./app2.js"]
	entry:{
		app:"./app.js",
	},
	output:{
		path:__dirname+"/dist",// 要求绝对路径
		filename:"[name].[hash:4].bundle.js"// name为入口文件的名字，hash:4表示取哈希值的4位
	},
	module:{
		rules:[
			{
				test:/\.css/,
				use:[minicss.loader,"css-loader"]
			},
			{
				test:/\.less/,
				use:[minicss.loader,"css-loader","less-loader"]// less-loader安装：npm install less less-loader --save-dev
			}
		]
	},
	plugins:[
		new minicss({
			filename:"test.bundle.css",//指定打包后的css文件出口
		}),
		new minimizer(),// css压缩插件安装：npm install css-minimizer-webpack-plugin --save-dev
	]
}
```

>[!Tip]
>- 解析less文件的流程：less-loader先将less文件解析为css文件，css-loader让webpack认识css文件，minicss.loader将css文件打包为一个单独的css文件

## 资源处理
##### 加载器

| **版本**     | webpack5之前                 | webpack5           |
| ---------- | -------------------------- | ------------------ |
| **loader** | 需要下载file-loader和url-loader | 使用内置asset加载器不用第三方包 |
##### 用法
```js
module.exports={
	//entry:["./app.js","./app2.js"]
	entry:{
		app:"./app.js",
	},
	output:{
		path:__dirname+"/dist",// 要求绝对路径
		filename:"[name].[hash:4].bundle.js"// name为入口文件的名字，hash:4表示取哈希值的4位
	},
	module:{
		rules:[
			/*// webpack5之前的写法
			{
				test:/\.(jpg|jpeg|png|gif|svg|mp3|ttf)$/,
				loader: "url-loader",
				options:{
					limit: 5000,
					name: "[name].[hash].[ext]"
				}
			},*/
			// webpack5写法（推荐）
			{
				test:/\.(jpg|jpeg|png|gif|svg|mp3|ttf)$/,
				type: "asset", // asset/inline:所有资源都打包为Base64；asset/resource:所有资源都大包为单独文件
				parser: {
					dataUrlCondition: {
						maxSize: 5000
					}
				},
				generator: {
					filename:"[name].[hash][ext]"
				}
			}
		]
	},

}
```
>[!Tip]
>- file-loader只是让webpack可以认识资源文件可以打包，但是不做其他处理，所以如果要处理资源文件webpack5之前使用的url-loader

## Loader
### Loader的本质
> Loader本质是一个方法，该方法接受到要处理的资源的内容，处理完后给出内容，作为打包结果
### 创建一个loader
```js
// 在这里我们创建一个简单的css处理loader用于演示
// css-loader/index.js
module.exports = function(cssContent){
	console.log(cssContent);
	cssContent = cssContent.replace("1","1px")
	return cssContent;
}

// 打包结果会让css的内容中所有1替换成1px
```

## 处理HTML
### 步骤
- 提供一个HTML模板，复用固定内容
- 打包成一个HTML文件
- 打包后的html自动引入打包后的js文件
### 插件实现
#### 安装
> 安装html-webpack-plugin插件

#### 使用
```js
const htmlwebpackplugin = require("html-webpack-plugin")

module.exports={
	//entry:["./app.js","./app2.js"]
	entry:{
		app:"./app.js",
	},
	output:{
		path:__dirname+"/dist",// 要求绝对路径
		filename:"[name].[hash:4].bundle.js"// name为入口文件的名字，hash:4表示取哈希值的4位
	},
	plugins:[
		/*
		// 单入口
		new htmlwebpackplugin（{
			template: "./index.html",// 假设你已经有了一个index.html模板页面
			filename: "index.html"
		}）*/
		// 多入口对应多个插件配置，每个插件配置需要指定要引用的js文件
		new htmlwebpackplugin({
			template: "./index.html",
			filename: "index.html",
			chunks: ["app"] // 指定js
			// 指定压缩策略
			minify:{
				collapseWhitespace: false,// 不删除空格和换行
				removeComments: false,// 不删除注释
				removeAttributeQuotes: false,// 不删除单词间不必要的空格
			}
			// 指定插入script的位置
			inject:"body"// body|true:插入body中，head:插入head中,false:不插入
		}),
		new htmlwebpackplugin({
			template: "./index.html",
			filename: "index2.html",
			chunks:["app2"] // 指定js
		})
	]

}
```

>[!Tip]
>- 在模板html中可以使用模板引擎写法嵌入内容

**EJS模板引擎**
```js
// EJS 模板引擎
<!-- 服务端渲染动态数据 -->
<h1>欢迎, <%= username %>!</h1>
<ul>
  <% todos.forEach(todo => { %>
    <li><%= todo.text %></li>
  <% }) %>
</ul>
```

## 分割代码
### 主要解决问题
- 单入口打包后的文件过大
- 多入口打包后，每个入口的重复文件下载多次
### 写法
```js
module.exports={
	//entry:["./app.js","./app2.js"]
	entry:{
		app:"./app.js",
	},
	output:{
		path:__dirname+"/dist",// 要求绝对路径
		filename:"[name].[hash:4].bundle.js"// name为入口文件的名字，hash:4表示取哈希值的4位
	},
	// 优化打包配置
	optimization:{
		// 通用分割配置
		splitChunks:{
			chunks: "all",// all:分割全部代码，async：分割异步代码，initial：分割同步代码
			cacheGroups:{
				verdor: {
					test: /[\\/]node_modules[\\/]/,
					filename: "vendor.js",
					chunks:"all",
					minChunks:1,// 引用1次就执行该打包规则
				},
				common: {
					filename: "common.js",
					chunks: "all",
					minChunks: 2,
					minSize:0,// 大小>minSize的文件执行该打包规则
				}
			}
		},
		runtimeChunk: {
			name: "runtime",// webpack运行代码打包到runtime.js文件中
		}
	},
	plugins:[
		/*
		// 单入口
		new htmlwebpackplugin（{
			template: "./index.html",// 假设你已经有了一个index.html模板页面
			filename: "index.html"
		}）*/
		// 多入口对应多个插件配置，每个插件配置需要指定要引用的js文件
		new htmlwebpackplugin({
			template: "./index.html",
			filename: "index.html",
			chunks: ["app"] // 指定js
			// 指定压缩策略
			minify:{
				collapseWhitespace: false,// 不删除空格和换行
				removeComments: false,// 不删除注释
				removeAttributeQuotes: false,// 不删除单词间不必要的空格
			}
			// 指定插入script的位置
			inject:"body"// body|true:插入body中，head:插入head中,false:不插入
		}),
		new htmlwebpackplugin({
			template: "./index.html",
			filename: "index2.html",
			chunks:["app2"] // 指定js
		})
	]

}
```

## 小技巧
### hash值得意义
>[!Tip]
>- 浏览器加载文件时有缓存的，如果你的文件名一样，即使内容改变了，浏览器还是会使用缓存的文件而不是上传的新文件
>- 如果使用[hash:4]的写法，每次打包生成的文件都共用相同的hash值这就导致有些内容没有改变的文件打包后，由于hash值的改变浏览器也要重新下载
>- 使用[chunkhash:4]就可以解决这个问题，因为这样每个打包后的文件有自己的hash值，只有自身内容变化的时候hash值才会改变

### resoleve
>[!Tip]
>- **alias**-别名，提供路径的简写
>- **Extensions**-拓展省略，定义可省略的拓展名

### require.context
>[!Tip]
>- 批量引入指定文件夹下的所有文件

```js
const r = require.context("./mode",false,/.js/);// 第一个参数指定引入哪个目录；第二个参数true代表子目录的文件也要引入，false相反；第三个参数指定要引入的符合一定规则的文件

// r.key()得到r所有的键。r(item).default得到某个文件默认导出的内容
r.key().forEach((item)=>{
	console.log(r(item).default);
})
```

### publicPath
>[!Tip]
>- 相当于路径前缀，常用于需要发布在cdn上的文件


## 开发

### 配置项
```js
devtool:"eval-cheap-source-map",// 开发常用配置，可定位到源代码；如果没有这个配置，只能定位到打包后的代码
devServer:{
	prot:1000,// 运行在哪个端口
	hot:true,// 是否开启热更新：默认针对资源文件，如果是js文件仍然是强制更新
	// 代理：解决浏览器跨域问题（除了proxy外后端也可开启cors解决跨域问题）
	proxy:{
		// 可写多个代理规则
		"/": {
			target: "http://localhost:3000/",
			pathRewrite:{
				"^/num1":"/api/getNum1",
				"^/num2":"/api/getNum2"
			},
			headers:{
				
			}
		},
	}
}
```
### webpack-dev-serve
#### 工作原理
![[Pasted image 20250711183148.png]]

#### 热更新和强制更新
##### 热更新
**在不刷新浏览器的情况下更新页面。可以保持页面的当前状态**
```js
// 让js文件热更新的方法(如果遇到必须热更新的情况可以这样写否则不推荐这样写)
if(module.hot){
	module.hot.accept();
}
```

##### 强制更新
**自动刷新页面来更新界面，会重置页面状态**

### proxy
>Proxy说白了就是由我们的webpack-dev-server开启的node服务来代替我们请求接口，因为如果后端没有开启cors，我们直接从前端请求会跨域。我们可以利用proxy。让请求从node服务发，这样就完全绕过了浏览器的同源策略

### source-map
>出现错误，或者输出内容的时候，source-map能够帮助我们定位到它来自哪个代码