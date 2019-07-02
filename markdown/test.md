# 【架构拾集】移动应用的自动化测试（BDD 方式）


我的上一篇关于自动化测试的文章，大抵已经在一年以前——一篇关于前端自动化测试的 BDD 框架对比。这么长的时间里，没有相关的文章，总得给自己找一个合适的理由。

编写测试是开发人员日常工作的一小部分，但并非是全部。即使是专业的测试人员，自动化测试也并非是全部的工作。与此同时呢，有了单元测试之外，对于自动化测试的需求，并不是那么强烈。速度，是我们不考虑自动化测试的主要原因，运行起来慢了，进一步地导致编写测试也慢了。UI 上一旦有一丁点的修改，那怕是得引起多少的不愉快，不得不噗呲噗呲地去修改测试。我想说的无非就是，**避免编写 “可怕” 地自动化测试**，尽量用你的单元测试来保障质量。要是你们有 KPI 的限制，请将以上的文字当成废话。

似乎在有些公司里，自动化测试、单元测试并不是技术负责人要考虑的问题，可在我司并非如此，测试也在技术的范围里。每每开始一个项目时，就不得不去考虑自动化测试的问题，选用什么框架合适、需要前后端如何配合、怎样去替换第三方的服务。这些内容完全交给测试人员吧，怕是会遇到一些不顺。测试人员有自己的测试技术栈，拥有自己的 “银弹” —— 过去的经验和代码库。哪怕经验再丰富的测试人员，有时遇到一些新的项目、技术栈，这些东西可能就用不上了，又或者是使用某些框架可能会更加便利。因此呢，要成为更好的技术人，测试也是要考虑的范畴。

说到 Web 方面的自动化测试，我算是个有经验的老手。从顶层的 DSL 到底层的 Web Driver，到底来说，还是颇有经验的。可是说到 APP 的自动化测试，在项目上尝试过，但也不敢说经验丰富。而最近的项目，正在实施相应的移动应用自动化测试。看了看方案，与之前的 Web 或者是 React Native 的自动化测试，从底层对比架构吧，相似、差距也不是太大。便想着写篇文章来记录一下，相应的架构设计。

## 技术远景

作为一个团队的技术负责人，我希望：拥有一个移动应用测试架构，它能快速让测试人员快速上手——阅读、编写测试用例。与此同时，我希望这些测试用例是能让非技术人员阅读，诸如业务分析人员，并且符合真实的用户使用场景。

## 架构设计

当我们谈到业务分析人员也能编写的测试，我们说的只有 BDD（Behavior Driven Development，行为驱动开发）。它是一种敏捷软件开发的技术，它鼓励软件项目中的开发者、QA（测试人员）和非技术人员或商业参与者之间的协作。

BDD  在这一种上相当的迷人——能让非技术人员编写测试。而其核心的部分在于：创建了一个环境隔离的 DSL，仿人类语言的 DSL。咳咳，这个 DSL 实现起来，可不轻松。关注顶层 DSL 的同时，开发人员还要努力实现好底层的实现。举个简单的例子，如下是之前在 BDD 一文中的 DSL 示例，这是顶层的设计：

```markdown
功能: 失败的登录

  场景大纲: 失败的登录
    假设 当我在网站的首页
```

对应的，开发人员需要编写实现：

```javascript
...
Given('当我在网站的首页', function() {
  return this.driver.get('http://0.0.0.0:7272/');
});
..
```

从上述的代码中，一眼就可以看出复杂的地方，实现一个领域特定（业务特定）的 DSL 语言。

我们要完成的 DSL 实现，上层是提供一个 DSL，下层则是对接 driver 的 Agent 层。在 Web 领域里，这个 driver 的 Agent 层负责对接不同的浏览器，诸如 Selenium，driver 则视不同的浏览器而有所不同，如 ChromeDriver、FirefoxDriver、PhantomJSDriver 等等。Selenium 这样的测试框架，除了通过 driver 直接操作了浏览器，还提供了不同语言的编程接口。

相似的，在 APP 领域也有这样的方案，它要通常这个 agent 来连接物理设备，并提供一系列的编程接口。

## 架构设计方案

对整个架构有了一个基本的认识之后，让我们继续往下移动，来重新发掘一下：我们究竟需要哪些基本元素？
 
 - BDD 测试框架，为开发人员提供可创建 DSL 的接口。
 - 移动设备的测试编程接口，提供一个操作移动应用的接口。
 - 连接移动设备的操作库，即移动端的 WebDriver。
 - 用于编写测试时的 UI 检查工具。

从这一点上来看，它与 Web 应用的 BDD 架构差不多。为此，我们需要准备如下的一些框架：

 - Robot Framework，一个支持 BDD 的、基于 Python 编写的功能自动化测试软件框架。
 - Appium，是一个开源测试自动化框架，用于原生，混合和移动 Web 应用程序。它使用 WebDriver 协议来驱动 iOS、Android 和 Windows 应用程序。
 -  XCUITest Driver，基于 Apple 官方的界面自动化测试 XCUITest 封装的测试接口，可以直接执行 iOS 的自动化测试。
 -  UiAutomator2 Driver，则是 Google 官方提供的用于 Android 系统的测试接口，可以直接执行 Android 的自动化测试。
 -  Appium Inspector，用于查找 iOS/Android 上的元素
 -  UiAutomator Viewer，由 Android SDK 自带的元素查找工具。

由于我们计划的顶层是由 DSL 来实现，而对应的 BDD 层实现是由 Robot Framework 来完成的。Robot Framework 使用的是 Python 语言，我们就需要找到对应的 Python 主要依赖有：

 - robotframework，即 Robot Framework 本身
 - robotframework-appiumlibrary，用于为 Robot Framework 提供 Appium 相应的接口封装
 - robotframework-ride，用于 Robot Framework 的测试数据编辑器

有了这些主要的库，我们就可以编写我们的 DSL？不，我们还需要配置好，对应的移动端 Driver。

**Android Driver 依赖**

比较简单，通过 ``appium-uiautomator2-driver`` 库就拥有了 driver。

**iOS Driver 依赖** 

为了实现对 iOS 设备的自动化测试，需要安装 XCUITest，为此我们需要下面的这一系列工具：

 - ``libimobiledevice``，是一个跨平台的用于与 iOS 设备通讯的协议库，它可以让 Windows、macOS、GNU/Linux 系统连接 iPhone/iPod Touch 等iOS 设备。
 - ``carthage`` 是一个简单的、去中心化的依赖管理工具。
 - ``ios-deploy`` 是一个使用命令行安装 iOS 应用程序到连接设备的工具
 - ``xcodebuild``，是苹果发布自动构建的工具。
 - ``xcpretty``，用于对 xcodebuild 的输出进行格式化，包含输出 report 功能。

看，有了这一系列的知识，我们几乎知道怎么做搭建移动应用的自动化测试。

## 结论

还是 Web 大法好。