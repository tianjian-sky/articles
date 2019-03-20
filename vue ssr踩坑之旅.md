# SSR
[TOC]

## 关于SSR
### SSR是什么(Server Side Rendering)
> 服务器端渲染: 
> 简单来说就是在服务器上把数据和模板拼接好以后发送给客户端显示。

### SSR 与 CSR（客户端渲染）
> 客户端渲染
> html 仅仅作为静态文件，客户端端在请求时，服务端不做任何处理，直接以原文件的形式返回给客户端客户端，然后根据 html 上的 JavaScript，生成 DOM 插入 html。


[服务端渲染 vs 客户端渲染](https://www.jianshu.com/p/656a1666a1c5?utm_source=oschina-app)

## 开发模式的演变

* asp,jsp,php,..（前端负责切图，js动效果，静态页面）
* 前后端分离 （ajax）

	> Ajax提供数据接口，就此复杂度从JSP里已到了浏览器的JavaScript，浏览器端变得很复杂，
	> 
* 单页应用（SPA）
* Node
> 在没有Grunt、Gulp、webpack等便捷且强大的工具的时候，静态资源的压缩合并，都是需要像Apache Ant这用工具完成的，但Node的出现，带来了Grunt、Gulp、webpack，结合JavaScript的灵活性与Node.js提供的API，前端工程师可以编写各种工具满足项目的开发需求。

[前端工程化之路—前后端协作发展史](https://www.jianshu.com/p/b06091f53aa5)


## CSR 模式的缺点
* 全异步，对 SEO 不利。往往还需要服务端做同步渲染的降级方案。
* 性能并非最佳，特别是移动互联网环境下。
	* 关键渲染路径
	* SSR： 解析html，生成节点树 解析css，生成css树 css树与节点树合并形成渲染树 布局 绘制 
	* 显然CSR模式下关键渲染路径更长，要有一个异步取数据并渲染的过程


## 新时代下的SSR
> 恩格斯在《自然辩证法》中说：“由矛盾引起的发展或否定的否定——发展的螺旋形式
> 
> 列宁指出：“发展似乎是在重复以往的阶段，但它是以另一种方式重复，是在更高的基础上重复。” [1] 
> 
### 代码同构

* 同构
* 得益于node.js的技术支持，一套代码前后端都搞定


## 搭建vue ssr demo

### 准备工作
首先，通过阅读官网文档，为了使vue项目能变成ssr模式，我们需要安装一个核心处理插件: vue-server-renderer

``` js
npm install vue vue-server-renderer --save
```
其次,为了达到代码重构的目的，我们的服务端必须使用node服务器，在这里选择express

``` js
npm install express --save
```
最后，开始做之前，再大致梳理一下ssr模式与普通csr模式工作流程的异同。
* csr模式
![1eb86a508a234c1fc0078e4250d29849.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p119)

* ssr模式
而在服务端渲染模式下，后端在获取到组件后只需将其转换为字符串，写入responnse返回给前端即可，因此ssr模式下后台的流程简化为：

![b0149db32128b6fd10b3f720885efb84.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p120)


![6d25ff458b6c7b9b466b3b126bae00fc.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p126)


但是光把html字符串吐给前端还不够，那样只是一个静态页面，之前组件中定义的事件交互，状态响应式渲染等都在字符串化后被抹去了。所以在服务端渲染内容到达客户端后，我们还需要在客户端对其进行“激活”。这时的流程又与csr模式有点区别，这次相当于我们有了最终产物：组件的html节点，但是缺少其他的环节。
客户端的流程如下：

### 开始搭建

* 服务端代码
![3ee27e85274e74975974ad9603079430.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p121)
* 客户端代码 （vue框架内部封装了激活ssr组件的实现，因此与常规非ssr写法一毛一样） 
![1665e4ed6fdde4c48bf15d357d54908b.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p122)

#### Bundle Renderer
在上面那种情况下组件内部的逻辑一旦发生变动，重新编译后就会生成一个hash值不同的js文件，而此时服务器必须更改文件url地址以获取更新的文件。为此可以采用bundleRenderer，即把具体业务代码写入一个json文件，这样无论业务逻辑如何改动，服务器脚本都不用动，因为都读取的是同一个名称的json文件。
![34764afc240eb568333056b6d441058b.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p123)

* 内置的 source map 支持
* 在开发环境甚至部署过程中热重载
* 关键 CSS(critical CSS) 注入
* 使用 clientManifest 进行资源注入：自动推断出最佳的预加载(preload)和预取(prefetch)指令，以及初始渲染所需的代码分割 chunk。

### 数据预获取
通常我们的页面都不是春泥静态的，页面里需要调用一些接口去获取数据。显而易见在ssr模式下，获取页面数据的任务必须要在服务端吐html给浏览器之前完成。具体如何实现，这里采用官网介绍的方法：
* 利用状态管理工具vuex， 所有页面数据统一在vuex中进行存储、处理。
* 根据页面的url地址匹配出涉及到的组件
* 在涉及的组件中提供异步数据获取接口asyncData
* 执行所有的asyncdata方法，请求成功后将数据存储进vuex
* 服务端将store写入html字符串
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
![09f135ffb04d790c19cc3b142a358f90.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p124)

![4542eab9709a9743511112464bb5e597.png](evernotecid://81FF9D19-9E7B-436D-BBB7-27841F7DFCAB/appyinxiangcom/21477520/ENResource/p125)


* vue中hydrte方法









