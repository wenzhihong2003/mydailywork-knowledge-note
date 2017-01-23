### 前端blog

[徐飞blog, 民工叔](https://github.com/xufei/blog)
[taobao云谦, dvajs发起人](https://github.com/sorrycc/blog/issues)

[JavaScript风格指南](https://github.com/alivebao/clean-code-js/blob/master/README.md)

[复杂单页应用的数据层设计](https://github.com/xufei/blog/issues/42  "spa")
[流动的数据——使用 RxJS 构造复杂单页应用的数据逻辑](https://github.com/xufei/blog/issues/38)

#### 电子书
[Chrome 开发者工具中文手册](https://github.com/CN-Chrome-DevTools/CN-Chrome-DevTools)
[Chrome扩展开发文档](http://open.chrome.360.cn/extension_dev/overview.html)
[浏览器开发工具的秘密](http://jinlong.github.io/2013/08/29/devtoolsecrets/)
[前端代码规范 及 最佳实践](http://coderlmn.github.io/code-standards/)

[Growth: 全栈增长工程师指南](https://github.com/phodal/growth-ebook)

[HTTP 接口设计指北](https://github.com/bolasblack/http-api-guide)
[HTTP2.0 中文翻译](http://yuedu.baidu.com/ebook/478d1a62376baf1ffc4fad99?pn=1)
[http2讲解](https://www.gitbook.com/book/ye11ow/http2-explained/details)

[免费的HTML5相关技术书籍](https://github.com/wangleihd/freeBook-H5)

[JavaScript 标准参考教程（alpha）](http://javascript.ruanyifeng.com/)
[ECMAScript 6 入门](http://es6.ruanyifeng.com/)
[Javascript 半知半解](http://www.kancloud.cn/dennis/tgjavascript/241800)


### electron

[electron 中文翻译文档](https://github.com/electron/electron/tree/master/docs-translations/zh-CN)
[Electron 中文文档](http://www.kancloud.cn/wizardforcel/electron-doc)
[Electron 应用实战 (架构篇)](https://github.com/sorrycc/blog/issues/13)
[Electron 实践之自动更新](https://github.com/sorrycc/blog/issues/26)

#### [在electron中出现 jQuery is not defined](http://stackoverflow.com/questions/32621988/electron-jquery-is-not-defined)
这个主要是因为electron默认使用的nodejs的module的加载模式起的. 解决方法

- 在electron里设置不集成nodejs, 在main进程里设置
```javascript
var win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: false //使用这个参数就会不使用module的方式引入js, 符合浏览器的习惯. 这个值默认为true
    }
  });
```

- 使用electron的默认设置BrowserWindow, 但是在网页(渲染进程)的最开始处插入
```html
<!-- Insert this line above script imports  -->
<script>if (typeof module === 'object') {window.module = module; module = undefined;}</script>
```
其核心也就是把`module`移动到`window.module`下面, 然后把`module`重新定义
这样在渲染进程里, 我们就可以使用与main进程的通讯方式了
```javascript
// 在主进程中
global.sharedObject = {
  someProperty: 'default value'
}
```
```javascript
// 在第一个页面中
require('electron').remote.getGlobal('sharedObject').someProperty = 'new value'
```
```javascript
// 在第二个页面中
console.log(require('electron').remote.getGlobal('sharedObject').someProperty)
```
也就是在 [Electron 应用实战 (架构篇)](https://github.com/sorrycc/blog/issues/13) 中提到的方式


### 知识点

[白话 JS 的原型链](https://wx.angular.cn/library/article/simple-prototype-chain)  
[白话 JS 数值的可变与不可变](https://wx.angular.cn/library/article/simple-js-mutability)  
[白话Javascript的Event Loop和Async](https://wx.angular.cn/library/article/simple-javascript-event-loop-and-async)

### rxjs

[《RxJS Overview》阅读笔记](http://mp.weixin.qq.com/s?__biz=MzI5MDM2NjY5Nw==&mid=2247483777&idx=1&sn=034b9caf38d1c7dfe871cf8c82ea9ec5&chksm=ec21b407db563d115c396f675027e734a90c527effd16822b15e160206e849f26b56a0669756&mpshare=1&scene=23&srcid=01180jTMKEpLmaWrqC2l5XVO#rd)  
[Async！JS中的异步！](http://mp.weixin.qq.com/s?__biz=MzI5MDM2NjY5Nw==&mid=2247483853&idx=1&sn=5345f2eb99b893873bfa88641cddfadc&chksm=ec21b44bdb563d5d5dbf11f09b9abcb0dae9f135312c64c8c7d6c226a06203c471f7b6b55e3f&mpshare=1&scene=23&srcid=0118ZrWJ9BVYwKaxQsMVU39n#rd)  
[如何理解 RxJS？](https://wx.angular.cn/library/article/%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3RxJS)  
[RxJS Overview阅读笔记](https://wx.angular.cn/library/article/RxJS%20Overview%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0)  
[一个简单的RxJS Test Spec](https://wx.angular.cn/library/article/%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84RxJS%20Test%20Spec)  
[应用RxJS模拟redux](https://wx.angular.cn/library/article/%E5%BA%94%E7%94%A8RxJS%E6%A8%A1%E6%8B%9Fredux)  
[应用RxJS模拟redux - 第二集 - Todo App](https://wx.angular.cn/library/article/%E5%BA%94%E7%94%A8RxJS%E6%A8%A1%E6%8B%9Fredux-%E7%AC%AC%E4%BA%8C%E9%9B%86-Todo-App)  
[白话RxJS](https://wx.angular.cn/library/article/simple-rxjs)  
[构建流式应用—RxJS详解](https://wx.angular.cn/library/article/%E6%9E%84%E5%BB%BA%E6%B5%81%E5%BC%8F%E5%BA%94%E7%94%A8%E2%80%94RxJS%E8%AF%A6%E8%A7%A3)  
[用 RxJS 连接世界](http://mp.weixin.qq.com/s?__biz=MzI5MDM2NjY5Nw==&mid=2247484036&idx=1&sn=032feca5f526d66ae91c018017f07870&chksm=ec21b702db563e144555a7a299ce023cd0f6890124f62aeb9c860c09fdac77b32d01951cba1e&mpshare=1&scene=23&srcid=0118vp6lbTNBY2kd5Qwk6O7g#rd)  
[RxJS 入坑教程](http://mp.weixin.qq.com/s?__biz=MzI5MDM2NjY5Nw==&mid=2247483966&idx=1&sn=201ca7dcfd54a14a315715b29a4608cb&chksm=ec21b7b8db563eae362f62d6430a61a87a11df0edcbfe63adb21e2865eeeb540e88879d693f2&mpshare=1&scene=23&srcid=0118HqgW2AuyCb48rYN5BqBV#rd)
