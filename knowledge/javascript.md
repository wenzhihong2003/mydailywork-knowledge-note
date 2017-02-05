### 前端blog

[徐飞blog, 民工叔](https://github.com/xufei/blog)  
[taobao云谦, dvajs发起人](https://github.com/sorrycc/blog/issues)  
[[dwqs-blog](https://github.com/dwqs/blog)  
[JavaScript风格指南](https://github.com/alivebao/clean-code-js/blob/master/README.md)  
[复杂单页应用的数据层设计](https://github.com/xufei/blog/issues/42  "spa")  

[关于Promise：你可能不知道的6件事](https://github.com/dwqs/blog/issues/1)  
[不再彷徨：完全弄懂JavaScript中的this](https://segmentfault.com/a/1190000006076637)  

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
[流动的数据——使用 RxJS 构造复杂单页应用的数据逻辑](https://github.com/xufei/blog/issues/38)  
[探秘 vue-rx 2.0](https://github.com/xufei/blog/issues/39)  
[数据的关联计算](https://github.com/xufei/blog/issues/36)  
[对当前单页应用的技术栈思考](https://github.com/xufei/blog/issues/37)

### angularjs

[Angular1.x + ES6 开发风格指南](https://github.com/kuitos/kuitos.github.io/issues/34)  
[Angular 1.x和ES6的结合](https://github.com/xufei/blog/issues/29)  
[AngularJS 入门教学视频, 由codeschool出品](http://blog.jobbole.com/87995/)  
[AngularJS最佳编码实践指南](http://blog.jobbole.com/80634/)  
[七步走 Angular.js 从菜鸟到专家](http://justcode.ikeepstudying.com/2016/05/%E4%B8%83%E6%AD%A5%E8%B5%B0-angular-js-%E4%BB%8E%E8%8F%9C%E9%B8%9F%E5%88%B0%E4%B8%93%E5%AE%B6/)  
[How do I integrate Angular.js with RxJS?](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/howdoi/angular.md)  
[Using RxJS Observables with AngularJS 1](http://cvuorinen.net/2016/05/using-rxjs-observables-with-angularjs-1/)  
[AngularJs seed](https://github.com/kuitos/angular-seed)  
[Angular最佳实践之$http-麻雀虽小 五脏俱全](https://github.com/kuitos/kuitos.github.io/issues/12)  
[Javascript异步流程控制之Promise(1)-Angular $q简介](https://github.com/kuitos/kuitos.github.io/issues/15)  
[Javascript异步流程控制之Promise(2)-Angular $q源码解读](https://github.com/kuitos/kuitos.github.io/issues/16)  
[Angular黑科技之transclude](https://github.com/kuitos/kuitos.github.io/issues/17)  
[基于ui-router的非侵入式angular按需加载方案](https://github.com/kuitos/kuitos.github.io/issues/31)  
[MVVM 模式](https://github.com/kuitos/kuitos.github.io/issues/35)  
[构建单页Web应用](https://github.com/xufei/blog/issues/5)  
[Angular沉思录（一）数据绑定](https://github.com/xufei/blog/issues/10)  
[Angular沉思录（二）视图模型的层次](https://github.com/xufei/blog/issues/11)  
[Angular沉思录（三）Angular中的模块机制 ](https://github.com/xufei/blog/issues/17)  
[AngularJS实例教程（一）——数据绑定与监控](https://github.com/xufei/blog/issues/14)  
[AngularJS实例教程（二）——作用域与事件](https://github.com/xufei/blog/issues/18)  
[Angular的问题](https://github.com/xufei/blog/issues/15)  
[后Angular时代二三事](https://github.com/xufei/blog/issues/21)  
[Angular的变革](https://github.com/xufei/blog/issues/25)  
[AngularJS入门教程——AngularJS中文社区提供](https://github.com/zensh/AngularjsTutorial_cn)  
[AngularJS入门教程](http://www.ituring.com.cn/book/1206)  
[AngularJS 教程](http://www.runoob.com/angularjs/angularjs-tutorial.html)  
[angularjs guide](http://docs.ngnice.com/guide)  
[angularjs 中文api说明](http://www.angularjsapi.cn/)  
[angularjs-style-guide](https://github.com/mgechev/angularjs-style-guide/blob/master/README-zh-cn.md)  
[AngularJS-Learning](https://github.com/jmcunningham/AngularJS-Learning/blob/master/ZH-CN.md)  
[angulartics Analytics for AngularJS applications 统计, 支持baidu](https://github.com/angulartics/angulartics)
[angular-styleguide es6](https://github.com/toddmotto/angular-styleguide)  
[Make-Your-Own-AngularJS(https://github.com/xufei/Make-Your-Own-AngularJS)  
[前端如何更好的实现接口的缓存和更新?](https://www.zhihu.com/question/40035517)  
[傅里叶分析之掐死教程（完整版）](https://zhuanlan.zhihu.com/p/19763358?columnSlug=wille)
  
### webpack

[Webpack 2 快速入门](https://github.com/dwqs/blog/issues/46)  
[webpack2 终极优化](http://imweb.io/topic/5868e1abb3ce6d8e3f9f99bb)  
[多页为王：webpack多页应用架构专题系列](http://array_huang.coding.me/webpack-book/)  
[Webpack的dll功能](https://segmentfault.com/a/1190000005969643)  
[经典webpack入门](https://github.com/starduliang/blog/blob/master/2016.6/webpack%20your%20bags.md)  
[Webpack——令人困惑的地方](https://segmentfault.com/a/1190000005089993)  
[入门 Webpack，看这篇就够了](https://segmentfault.com/a/1190000006178770)  
