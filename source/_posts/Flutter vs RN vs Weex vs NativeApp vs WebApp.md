---
title: Flutter vs RN vs Weex vs NativeApp vs WebApp
tags: [Flutter,React Native,NativeApp,Weex,WebApp,跨平台,Android,iOS,Web]
grammar_cjkRuby: true
categories: [技术选型]
date: 2018-08-27
---

### App开发的四种方式
#### NativeApp

Apple iOS SDK于2008年发布，2009年发布Google Android SDK。这两个SDK分别基于不同的语言：Objective-C和Java。应用构建步骤：

![enter description here][1]

应用程序会与平台进行交互，以创建小部件或访问相机等服务。小部件呈现给屏幕画布，并且事件被传回给小部件。这是一个简单的架构，但是你需要为每个不同的平台创建单独的应用程序。由于我们开发的程序最终都是需要运行在设备提供商的平台和设备上的，所以NativeApp可以实现与硬件最高效率的交互。

#### WebApp

第一个跨平台框架基于JavaScript和WebView。例如Titanium和一系列相关的框架：PhoneGap，Apache Cordova，Ionic等等。在苹果发布iOS SDK之前，他们鼓励第三方开发者为iPhone构建webapps，所以用web技术构建跨平台的应用程序是一个显而易见的步骤。应用构建步骤：

![enter description here][2]

使用WebView作为NativeApp的载体来加载网页程序，中间使用一个“桥”来与平台Rom进行交互，由于不能直接操作硬件平台，再加上所有的视图都是由前端页面构建，所以在使用本地硬件时会有一定不便和性能问题，在UI展示上也不能很好的实现原生的流程效果。

#### Reactive（RN、Weex）
最近几年，React变得越来越流行，相比大家对ReactJS等其他的反应视图都有了一定的了解，在跨平台领域借助于ReactJS可以很方便的简化Web视图的创建，React Native的诞生更是在很大程度上方便了跨平台开发者。应用构建步骤：

![enter description here][3]

从构建步骤不难看出，React Native使用了和WebView不同的方式，使用“桥”很好地让ReactJS与Native进行沟通，而非与webView沟通。在React Native中所有的View基本都是继承于原生平台的View，这使得React Native在原生体验上基本可以达到原生应用的层次，但是在动画高帧率方面还是有很大的不足。

#### Flutter
像React Native一样，Flutter也提供反应式风格的视图。Flutter采用不同的方法来避免由于使用编译的编程语言而导致的性能问题。Flutter使用AOT编译成多个平台的本地代码。这使得Flutter可以与平台进行通信，而无需通过执行上下文切换的JavaScript桥。编译为本机代码也可以提高应用程序的启动时间。应用构建步骤：

![enter description here][4]

在Flutter中通过使用本地内部的View来构建View树，然后使用AOT技术在原生平台编译为原生View，中间少了JavaScript桥，在性能上可以说是有了很大的提高，再加上使用的完全都是原生界面的东西，所以在原生界面的还原上也是非常出色的。

### Flutter及其优缺点

#### Flutter是什么

Flutter是一款移动应用程序SDK，一份代码可以同时生成iOS和Android两个高性能、高保真的应用程序。

Flutter目标是使开发人员能够交付在不同平台上都感觉自然流畅的高性能应用程序。我们兼容滚动行为、排版、图标等方面的差异。

Flutter框架是一个分层的结构，每个层都建立在前一层之上。

![enter description here][5]



#### 优点
- HotReload机制，大大节约了编译、安装、调试的过程，开发体验非常好

- 标准的Material Design风格，界面大气、上档次

- 兼容性较好，在iOS和Android平台的差异性较少（但是针对两个平台的特殊处理仍然必不可少，所以也只是做到部分跨平台）

- 一切皆widget的设计，风格统一

#### 缺点

- 仅保留arm64-v8a的so包情况下，实际APK的大小仍然达到14M左右（网络上的说法是8M左右）

- APP启动过慢，用户体验明显（Hello World程序的白屏时长大于4-5s，虽然可以使用Splash引导页提升用户体验，但无法实质上提升启动速度）
- 界面布局方式过于复杂原始，无可视化且嵌套过深（虽然HotReload能够在一定程度上弥补这一问题，但是仍然无法接受） 

- 当前可用的第三方库的丰富性和稳定性有待提升，自定义控件和库，难度和成本较高。不太适合创新型应用

### React Native及其优缺点

#### React Native是什么

React Native使你只使用JavaScript也能编写原生移动应用。 它在设计原理上和React一致，通过声明式的组件机制来搭建丰富多彩的用户界面。RN使用Javascript语言，类似于HTML的JSX，以及CSS来开发移动应用，因此熟悉Web前端开发的技术人员只需很少的学习就可以进入移动应用开发领域。

![enter description here][6]

RN需要一个JS的运行环境， 在IOS上直接使用内置的javascriptcore， 在Android 则使用webkit.org官方开源的jsc.so。 此外还集成了其他开源组件，如fresco图片组件，okhttp网络组件等。

#### 优点

- 它对比原生开发更为灵活，对比H5体验更为高效。（JSX和HTML5还是有差异，有学习成本）

- 替代传统的WebView，打开效率更高，和原生之间的交互更方便。

- 多个版本迭代后的今天，它已经拥有了丰富第三方插件支持。

- 更方便的热更新

#### 缺点

- 尽管是跨平台，但是不同平台Api的特性与显示并不一定一致，只有部分代码可重用。

- 相对增大了app的体积。

- 调试’相对‘麻烦。

- Android上的兼容性问题。

### Weex及其优缺点

#### Weex是什么
Weex 是一个使用 Web 开发体验来开发高性能原生应用的框架。

Weex 致力于使开发者能基于当代先进的 Web 开发技术，使用同一套代码来构建 Android、iOS 和 Web 应用。具体来讲，在集成了 WeexSDK 之后，你可以使用 JavaScript 和现代流行的前端框架来开发移动应用。Weex 的结构是解耦的，渲染引擎与语法层是分开的，也不依赖任何特定的前端框架，目前主要支持 Vue.js 和 Rax 这两个前端框架。Weex 的另一个主要目标是跟进当代先进的 Web 开发和原生开发的技术，使生产力和性能共存。在开发 Weex 页面就像开发普通网页一样；在渲染 Weex 页面时和渲染原生页面一样。


Weex 表面上是一个客户端技术，但实际上它串联起了从本地开发、云端部署到分发的整个链路。开发者首先可在本地像编写 web 页面一样编写一个 app 的界面，然后通过命令行工具将之编译成一段 JavaScript 代码，生成一个 Weex 的 JS bundle；同时，开发者可以将生成的 JS bundle 部署至云端，然后通过网络请求或预下发的方式加载至用户的移动应用客户端；在移动应用客户端里，Weex SDK 会准备好一个 JavaScript 执行环境，并且在用户打开一个 Weex 页面时在这个执行环境中执行相应的 JS bundle，并将执行过程中产生的各种命令发送到 native 端进行界面渲染、数据存储、网络通信、调用设备功能及用户交互响应等功能；同时，如果用户希望使用浏览器访问这个界面，那么他可以在浏览器里打开一个相同的 web 页面，这个页面和移动应用使用相同的页面源代码，但被编译成适合Web展示的JS Bundle，通过浏览器里的 JavaScript 引擎及 Weex SDK 运行起来的。

![enter description here][7]


#### 优点
- 跨平台并支持web端（对微信小程序开发更方便复用）

- 学习成本更小（Vue）

- 更方便的热更新

#### 缺点
- 起步较晚，文档不全，资料少，社区几乎等于没有

- 不够稳定

### NativeApp及其优缺点

#### NativeApp是什么

采用Java或Kotlin编写的Android原生App，或采用Objective-C或Swift编写的iOS原生App

#### 优点
- 性能最优

- 用户体验最好

- 稳定性好

- 功能最全面
 
#### 缺点

- 无法跨平台

- 应用升级复杂

### WebApp及其优缺点

#### WebApp是什么

基于HTML和JS开发Web页面，并在App中使用WebView或Webkit进行加载展示

#### 优点

- 上手简单

- 可维护性

- 升级方便

#### 缺点

- 性能差

- 用户体验差

### Flutter vs RN vs Weex vs NativeApp vs WebApp总结与选型建议

整个上来看，WebApp和NativeApp是两个极端，Flutter、RN和Weex处于过滤形态，这三都各有特色，可根据需要选用。

![enter description here][8]


  
  ### 参考资料
https://www.zhihu.com/question/50156415/answer/331761116
https://www.cnblogs.com/guyuehuanhuan/p/6847979.html


  [1]: ./images/1535541175605.jpg
  [2]: ./images/1535541272841.jpg
  [3]: ./images/1535541401692.jpg
  [4]: ./images/1535541498964.jpg
  [5]: ./images/1535534522691.jpg
  [6]: ./images/1535544222539.jpg
  [7]: ./images/1535542216337.jpg
  [8]: ./images/1535544028111.jpg