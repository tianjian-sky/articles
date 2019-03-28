# SSR
[TOC]

## 关于SSR
### SSR是什么(Server Side Rendering)
* SSR: 服务器端渲染: 
> 简单来说就是在服务器上把数据和模板拼接好以后发送给客户端显示。

## CSR（客户端渲染）
* CSR: 客户端渲染
> html 仅仅作为静态文件，客户端端在请求时，服务端不做任何处理，直接以原文件的形式返回给客户端客户端，然后根据 html 上的 JavaScript，生成 DOM 插入 html。


[服务端渲染 vs 客户端渲染](https://www.jianshu.com/p/656a1666a1c5?utm_source=oschina-app)

## SSR到CSR的演变

* asp,jsp,php,..（前端负责切图，js动效果，静态页面）
* 前后端分离 （ajax）

	> Ajax提供数据接口，就此复杂度从JSP里已到了浏览器的JavaScript，浏览器端变得很复杂，
	> 
* 单页应用（SPA）
* Node

[前端工程化之路—前后端协作发展史](https://www.jianshu.com/p/b06091f53aa5)


## CSR 模式的缺点
* 对 SEO 不利。往往还需要服务端做同步渲染的降级方案。
* 首屏渲染

## SSR 的优点：
* 对seo有利，对搜索引擎友好
* 渲染路径较短，更快的内容到达时间

[为什么使用服务器端渲染 (SSR)](https://ssr.vuejs.org/zh/#%E4%BB%80%E4%B9%88%E6%98%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E6%B8%B2%E6%9F%93-ssr-%EF%BC%9F)

![avatar](https://s2.ax1x.com/2019/03/26/AUdIEt.png)
![avatar](https://s2.ax1x.com/2019/03/26/AU0VSS.png)



* 快手直播
* 淘宝


## 新时代下的SSR
虽然ssr的做法又被提上了桌面，看似又回到了起点，但是与原先的jsp，asp等服务端渲染方法相比，目前前端的ssr做法更加高大上一点，强调同构应用，同一套代码技能在浏览器上跑，也能在node服务器上跑。

### 同构应用

* 得益于node.js的技术支持，一套代码前后端都搞定
![avatar](https://s2.ax1x.com/2019/03/26/AUD7Zj.png)


## 搭建vue ssr demo

### 准备工作
首先在开始做之前，再大致梳理一下ssr模式与普通csr模式工作流程的异同。
* csr模式
![avatar](https://s2.ax1x.com/2019/03/26/AUrW79.png)
* ssr模式
而在服务端渲染模式下，后端在获取到组件后只需将其转换为字符串，写入responnse返回给前端即可，因此ssr模式下后台的流程简化为：
![avatar](
https://s2.ax1x.com/2019/03/26/AUrvtI.png)


![avatar](https://s2.ax1x.com/2019/03/27/Aas1dH.png)

因此需要安装一个核心处理插件: vue-server-renderer
``` js
npm install vue vue-server-renderer --save
```
其次,为了达到代码同构的目的，我们的服务端必须使用node服务器，在这里选择express
``` js
npm install express --save
```


但是光把html字符串吐给前端还不够，那样只是一个静态页面，之前组件中定义的事件交互，与vnode之间的响应关系等都在字符串化后被抹去了。所以在服务端渲染内容到达客户端后，我们还需要在客户端对其进行“激活”。这次相当于我们有了最终产物：组件的html节点，但是需要为节点注入必要的信息。
客户端的流程如下：
![avatar](https://s2.ax1x.com/2019/03/27/Aas3od.png)




服务端与浏览器端的底层组件创建过程代码均可复用，只是实例化组件后的用途不同，因此服务端和浏览器端分别需要两个入口文件server-entry.js及client-entry.js，以及相应的两个打包文件webpack.server.config.js及webpack.client.config.js.底层的具体业务逻辑实现则完全可以复用。
![avatar](https://s2.ax1x.com/2019/03/27/AasGFA.png)


### 打包配置
#### webpack.client.js
* 引入vueSSRClientPlugin
``` js
var configBase = require('./webpack.base')
var merge = require('webpack-merge')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var path = require('path')
var VueSSRClientPlugin = require('vue-server-renderer/client-plugin')

function resolve (dir) {
    return path.join(__dirname, '..', dir)
}

var configClient = {
    entry: {
        app: './src/client-entry.js'
    },
    output: {
        path: path.resolve(__dirname, 'dist/vueSSR'),
        filename: 'clientEntry.[chunkhash].js'
    },
    plugins: [
        // new HtmlWebpackPlugin({
        //     template: path.resolve(__dirname, 'src/template/index.html'),
        // }),
        new VueSSRClientPlugin()
    ]
}

var output = merge([configClient, configBase]);

module.exports = output


```
** 引入改插件的目的是为我们客户端打包出来的资源文件生成一个名称固定的json，当在服务器端进行渲染时传入这个json文件，json文件里列出的client-entry.js等文件都会被检索并写如html中，等到浏览器加载到页面后，客户端代码便会执行了。
#### webpack.server.js
* 引入 VueSSRServerPlugin
* webpack 配置为node环境
* vue-style-loader

``` js
var configBase = require('./webpack.base')
var merge = require('webpack-merge')
var path = require('path')
var ssrPlugin = require('vue-ssr-webpack-plugin')
var VueSSRServerPlugin = require('vue-server-renderer/server-plugin')

function resolve (dir) {
    return path.join(__dirname, '..', dir)
}

var configServer = {
    target: 'node',
    entry: {
        serverEntry: './src/server-entry.js'
    },
    output: {
        path: path.resolve(__dirname, 'dist/vueSSR'),
        filename: '[name].[chunkhash].js',
        libraryTarget: 'commonjs2'
    },
    node: {
        fs: 'empty',
        net: 'empty',
        module: 'empty'
    },
    plugins: [ new VueSSRServerPlugin()]
}

var output = merge([configServer, configBase]);

module.exports = output
```



### 编写入口文件
#### server-entry

``` js
import { createApp } from './app'

let func = context => {
  const { app } = createApp()
  return app
}

export default func
```

#### server.js

``` js
const server = require('express')()
const path = require('path')
const renderer = require('vue-server-renderer').createRenderer({
    template: require('fs').readFileSync(path.join(__dirname, './src/template/index.html'), 'utf-8')
})

const createApp = require(path.join(__dirname, './dist/vueSSR/serverEntry.1245ffa10e859f5b6fde')).default

server.get('*', (req, res) => {
  console.log(2, createApp)
  const app = createApp()
  renderer.renderToString(app, (err, html) => {
    if (err) {
      res.status(500).end('Internal Server Error')
      return
    }
    res.end(html)
  })
})

server.listen(8080)
```
#### client-entry

vue框架内部封装了激活ssr组件的实现，因此与常规非ssr写法一毛一样） 
``` js
import { createApp } from './app'

const { app, store } = createApp()

// 这里假定 App.vue 模板中根元素具有 `id="app"`
app.$mount('#app')
```



### 数据预获取
通常我们的页面都不是纯静态的，页面里需要调用一些接口去获取数据。显而易见在ssr模式下，获取页面数据的任务必须要在服务端吐html给浏览器之前完成。具体如何实现，这里采用官网介绍的方法：
* 根据页面的url地址匹配出涉及到的组件
* 在涉及的组件中提供异步数据获取接口asyncData
* 执行所有的asyncdata方法，请求成功后将数据存储进store
* 服务端将store写入html字符串返回响应
* 客户端加载页面时取得服务端写入的store，并且替换客户端的store

``` js

// entry-server.js
 const matchedComponents = router.getMatchedComponents()
      if (!matchedComponents.length) {
        return reject({ code: 404 })
      }
      // 对所有匹配的路由组件调用 `asyncData()`
      Promise.all(matchedComponents.map(Component => {
        if (Component.asyncData) {
          return Component.asyncData({
            store,
            route: router.currentRoute
          })
        }
      })).then(() => {
        context.state = store.state
        resolve(app)
```

``` js
// entry-client.js
const { app, router, store } = createApp()
if (window.__INITIAL_STATE__) {
  store.replaceState(window.__INITIAL_STATE__)
}
```



### 客户端激活原理（hydrate）
> 所谓客户端激活，指的是 Vue 在浏览器端接管由服务端发送的静态 HTML，使其变为由 Vue 管理的动态 DOM 的过程。
> 由于服务器已经渲染好了 HTML，我们显然无需将其丢弃再重新创建所有的 DOM 元素。相反，我们需要"激活"这些静态的 HTML，然后使他们成为动态的（能够响应后续的数据变化）。
> 如果你检查服务器渲染的输出结果，你会注意到应用程序的根元素上添加了一个特殊的属性：

``` js
<div id="app" data-server-rendered="true">
```

> data-server-rendered 特殊属性，让客户端 Vue 知道这部分 HTML 是由 Vue 在服务端渲染的，并且应该以激活模式进行挂载。注意，这里并没有添加 id="app"，而是添加 data-server-rendered 属性：你需要自行添加 ID 或其他能够选取到应用程序根元素的选择器，否则应用程序将无法正常激活。
> 在开发模式下，Vue 将推断客户端生成的虚拟 DOM 树 (virtual DOM tree)，是否与从服务器渲染的 DOM 结构 (DOM structure) 匹配。如果无法匹配，它将退出混合模式，丢弃现有的 DOM 并从头开始渲染。在生产模式下，此检测会被跳过，以避免性能损耗。


![09f135ffb04d790c19cc3b142a358f90.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p124)

![4542eab9709a9743511112464bb5e597.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p125)


* vue中hydrate方法


## 优化措施
### 缓存
* 页面级别缓存
* 组件级别缓存
> vue-server-renderer 内置支持组件级别缓存 (component-level caching)。要启用组件级别缓存，你需要在创建 renderer 时提供具体缓存实现方式(cache implementation)。典型做法是传入 lru-cache

``` js
const LRU = require('lru-cache')
const renderer = createRenderer({
  cache: LRU({
    max: 10000,
    maxAge: ...
  })
})
```
> 每个组件上提供 serverCacheKey 函数
``` js
export default {
  name: 'item', // 必填选项
  props: ['item'],
  serverCacheKey: props => props.item.id,
  render (h) {
    return h('div', this.item.id)
  }
}
```

## 其他需要考虑的问题
* 代码写法注意node环境与浏览器环境兼容
* 服务端压力承载
* 监控
    * 性能监控
    * 错误监控
    * 灰度策略


)
