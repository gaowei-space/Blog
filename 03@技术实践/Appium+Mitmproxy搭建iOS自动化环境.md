#  Appium+Mitmproxy 搭建iOS自动化环境



## 一 安装 mitmproxy

1. 安装 brew [查看](https://brew.sh/)
2. 安装 mitmproxy [查看](https://mitmproxy.org/)
3. 验证安装是否成功 `mitmdump --version`
4. 手机和PC处于同一个局域网下，配置好了mitmproxy的CA证书
   执行`mitmproxy --listen-host 127.0.0.1 -p 8899`之后，浏览器访问`http://mitm.it/`， 安装证书并且信任证书（通用>VPN与设备管理 | 描述文件与设备管理>mitmproxy>）

5. 创建 python 脚本 `response-to-api.py`，编写拦截host并上报api的逻辑
6. 运行脚本 `mitmdump -m transparent -p 8899 -s response-to-api.py --no-ssl-insecure --no-http2 --showhost --rawtcp --ignore-hosts "(.*tangguomao.*)|(.*static.tangguomao.*) |(.*.apple.*)|(.*icloud.*)|(.*mzstatic.*)|(.*autonavi.*)|(.*nmzhuan.*)|(.*baidu.*)|(.*weixin.*)|(.*growingio.*)|(.*umengcloud.*)|(.*umeng.*)|(.*toutiao.com.*)|(.*taobao.com.*)|(.*59.109.100.97.*)"`
7. 打开手机访问指定APP，查看是否有捕捉
参考文档: https://ptorch.com/news/269.html



## 二 配置透明代理

> OSX Lion 之后的系统集成了OpenBSD项目中的pf数据包过滤器，mitmproxy使用它来在OSX上实现透明模式。请注意，这意味着我们不支持OSX早期版本的透明模式。

#### 1.启用IP转发。

```
sudo sysctl -w net.inet.ip.forwarding=1
```

#### 2.将下行内容放在一个名为pf.conf（/etc/pf.conf）的文件中。

```
rdr pass on en0 inet proto tcp to any port {80, 443} -> 127.0.0.1 port 9999
```

此规则告诉pf将发往端口80或443的所有流量重定向到在端口9999上运行的本地mitmproxy实例。您应该替换 `en0` 为将出现测试设备的网卡。

#### 3.使用规则配置pf。

```
sudo pfctl -f pf.conf
```

#### 4.现在启用它。

```
sudo pfctl -e
```

#### 5.配置sudoer以允许mitmproxy访问pfctl。

以根用户身份在系统上编辑文件/ etc / sudoers。将以下行添加到文件末尾：

```
ALL ALL=NOPASSWD: /sbin/pfctl -s state
```

请注意，这允许系统上的任何用户/sbin/pfctl -s state以root用户身份运行命令而无需输入密码。这仅允许检查状态表，因此不应有不适当的安全风险。如果您有特殊要求，请随时限制运行mitmproxy的用户。

#### 6.Fire up mitmproxy。

您可能想要这样的命令：

```
mitmproxy --mode transparent --showhost
```

该--mode transparent标志打开透明模式，该--showhost参数告诉mitmproxy使用Host标头的值显示URL。

#### 7.最后，配置您的测试设备。

设置测试设备以将运行mitmproxy的主机用作默认网关，并在测试设备上安装mitmproxy证书颁发机构。

> 请注意，上面给出的pf.conf中的 rdr 规则仅适用于入站流量。 这意味着它们将不会重定向来自运行pf本身的盒子的流量。我们无法区分非mitmproxy应用程序的出站连接和mitmproxy本身的出站连接。如果要拦截自己的macOS流量，请参阅下面的解决方法，或使用外部主机运行mitmproxy。实际上，PF可以灵活地满足各种创造性的可能性，例如拦截来自VM的流量。有关更多信息，请参见pf.conf手册页。



参考文档：https://ptorch.com/docs/10/howto-transparent



## 三 配置appium

### 目录

- IOS 自动化相关框架介绍
- 基础环境搭建
- 安装内容
- 利用Appium-Python-Client进行iOS的自动化测试

### 前言

以下内容参考了如下网站：

- appium英文官方：https://appium.io/docs/en/drivers/ios-xcuitest/index.html
- appium使用问题集锦： https://discuss.appium.io/ (需要：科学上网)
- facebook-wda源码：https://github.com/kwmgenius/facebook-wda
- TestHome社区：https://testerhome.com/topics/24227?locale=zh-CN
- WebDriverAgent for ios: https://docs.katalon.com/katalon-studio/docs/installing-webdriveragent-for-ios-devices.html (需要：科学上网)
- Appium XCUITest Driver Setup: https://www.vskills.in/certification/tutorial/appium-xcuitest-driver-real-device-setup/ (需要：科学上网)
- stackoverflow中文社区：https://www.soinside.com/
- 简书：mac下配置Appium和WebDriverAgent： https://www.jianshu.com/p/81899f2b64b0
- [ATX](https://testerhome.com/topics/node78) ATX 文档 - iOS 真机如何安装 WebDriverAgent ：https://testerhome.com/topics/7220
- Mac OS安装appium: https://www.techaheadcorp.com/blog/how-to-install-appium-on-mac/
- 初识 iOS 自动化测试框架 WebDriverAgent : https://www.cnblogs.com/zgq123456/p/9979280.html
- openatx/facebook-wda安装教程：https://github.com/openatx/facebook-wda/blob/master/README.md （需要科学上网）
- 如何设置和自定义WebDriverAgent服务器: https://github.com/appium/appium/blob/master/docs/en/advanced-concepts/wda-custom-server.md



### IOS 自动化相关框架介绍

自动化测试类工具 随着移动互联网的兴起，APP 测试的越来越被重视！Android 系统因为自己的开源性，测试工具和测试方法比较广为流传，但是 iOS 系统的私密性，导致很多测试的执行都有点麻烦。

为了帮助大家更好的执行 iOS APP 的测试，以下为大家收集了非常全面的 iOS 测试工具，涵盖各大领域，希望各位能有所认识

#### 自动化测试类工具



**1.UIAutomation**

UIAutomation 是苹果提供的 **UI 自动化测试框架**，使用 JavaScript 编写。

基于 UIAutomation 有扩展型的工具框架和驱动型的框架。扩展型框架以 JavaScript 扩展库方法提供了很多好用 js 工具，注入式的框架通常会提供一些 Lib 或者是 Framework，**要求测试人员在待测应用的代码工程中导入这些内容，框架可以通过他们完成对 app 的驱动。**驱动型 UI Automation 在自动化测试底层使用了 UI Automation 库，通过 TCP 通信的方式驱动 UI Automation 来完成自动化测试，通过这种方式，**编辑脚本的语言不再局限于 JavaScript。**

这个工具在 iOS UI 自动化测试中使用非常广泛。

具体参考资料：https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/UIAutomation.html



**2.XCTest**

XCTest 是苹果在 iOS 7 和 Xcode5 引入的一个简单而强大的测试框架，**集成在 Xcode 中，用来编写测试代码**。它提供了各个层次的测试。

XCTest 测试编写起来非常简单，并且遵循 xUnit 风格。

Xcode 在创建工程时，会默认使用 XCTest，并且默认创建了 **Unit Test（单元测试）**和 **UI Test（界面测试）**两个 Target，其中 Unit Test 主要用于测试代码的大部分基本功能，比如绝大多数 Model 的类和方法测试，业务逻辑测试，网络接口调用测试等等。

UI Test 一般会考虑到用户的交互流程，模拟用户的交互操作，利用 XCTest 的 UI 记录特性来获取界面上的一些列视图元素和操作事件，然后在测试方法中触发事件。

所以**这是一个可以提供各个层次的测试的框架**，比如单元测试，自动化测试，性能测试等。

具体参考资料：

https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/01-introduction.html



**3. KIF**

KIF 是 Keep It Functional 项目的缩写，是一款 **iOS app 功能性测试框架**，来自 Square，该测试框架只支持 iOS。

所有测试使用 Objective-C 语言编写，对测试人员来讲，需要熟练的掌握 Objective-C 语言 ， 对苹果开发者来说非常容易上手，更是一款开发者广为推荐的测试工具。

KIF 使用未公开的 Apple API（私有 API），这**对于测试目的而言是安全的**，基于第三方 iOS UI 的单元测试框架，所以可以做项目的单元测试，也可以做 UI 集成测试。但**缺点是运行较慢。**

具体参考资料：https://github.com/kif-framework/KIF



**4. Frank**

Frank 是 iOS 开发环境下一款**实现自动测试的工具**，Xcode 环境下开发完成后，通过 Frank 实现结构化的测试用例，其底层语言为 Ruby，作为一款开源的 iOS 测试工具，在国外已经有广泛的应用。

但是国内相关资料却比较少。其**最大的优点是允许我们用熟悉的自然语言实现实际的操作逻辑。**

它提供了针对 iOS 平台的功能测试能力，可以模拟用户的操作对应用程序进行黑盒测试，并且使用 Cucumber 编写测试用例，使测试用例如同自然语言一样描述功能需求，**让测试以“可执行的文档”的形式成为业务客户与交付团队之间的桥梁。**

**优点：** 测试场景是在 Cucumber 的帮助下，用可理解的英语句子写的，还有活跃的社区支持，以及不断扩大中的库。

**缺点：**对手势的支持有限，所以在设备上运行测试有点难。

具体参考资料：https://www.testingwithfrank.com/



**5. Calabash-iOS**

Calabash 是一个适用于 iOS 和 Android 开发者的**跨平台 app 测试框架**，可用来测试屏幕截图、手势和实际功能代码。

Calabash 开源免费并支持 Cucumber 语言，Cucumber 能让你用自然的英语语言表述 app 的行为，实现 BDD（Behavior Driven Development，行为驱动开发）。

而 Calabash-iOS 就是一个基于 Calabash 的 iOS 的功能、自动化测试框架。

**优点：**

- 有大型社区支持；
- 列表项简单，类似英语表述的测试语句支持在屏幕上的所有动作，如滑动，缩放，旋转，敲击等。

**缺点：**

- 测试步骤失败后，将跳过所有的后续步骤，这可能会导致错过更严重的产品问题。
- 测试耗费时间，因为它总是默认先安装 app，需要 Calabash 框架安装在 iOS 的 ipa 文件中， 因此测试人员必须要有 iOS 的 app 源码。
- 除了 Ruby，对其他语言不友好

具体资料获取路径：https://github.com/calabash/calabash-ios



**6. Subliminal**

Subliminal 是另一款与 XCTest 集成的框架，也是个不错 iOS 集成测试框架。

与 KIF 不同的是，它基于 UIAutomation 编写，对开发者隐藏 UIAutomation 中一些复杂的细节。可惜近几年没有更新了，若能支持 swift 就好了。

具体资料获取路径：https://github.com/Diaoul/subliminal



**7. Kiwi**

Kiwi 是对 XCTest 的一个完整替代，使用 xSpec 风格编写测试。Kiwi 带有自己的一套工具集，包括 expectations、mocks、stubs，甚至还支持异步测试。

它是一个适用于 iOS 开发的 Behavior Driven Development（BDD）库，有着非常漂亮的语法。

优点在于其**简洁的接口和可用性**，易于设置和使用，可以写出结构性强易读测试，**非常适合新手开发者**。

Kiwi 也是使用 Objective-C 语言编写，易于 iOS 开发人员上手。

具体资料获取路径：https://github.com/kiwi-bdd/Kiwi **8. Appium**

Appium 是一个**开源的、跨平台的自动化测试工具**，支持 iOS、Android 和 FirefoxOS 平台。

通过 Appium，开发者无需重新编译 app 或者做任何调整，就可以测试移动应用，可以使测试代码访问后端 API 和数据库。

它是通过驱动苹果的 UIAutomation 框架来实现的 iOS 平台支持。

开发者可以使用 WebDriver 兼容的任何语言编写测试脚本，如 Ruby，C#，Java， JS，OC， PHP，Python，Perl 和 Clojure 语言。

具体资料获取路径：http://appium.io/

#### 内测发布工具

**1. fir.im**

为开发者提供测试应用极速发布，应用崩溃实时分析、用户反馈收集等一系列开发测试效率工具服务，帮助开发者将更多精力放在产品的开发与应用的优化上。

**2. 蒲公英**

『蒲公英』是专为 iOS、Android 开发者提供的免费用应用内测、托管的平台，旨在解决开发者将应用分发给内测用户时的繁杂、低效的问题。

**3. TestFlight**

TestFlight 是苹果提供的应用测试工具，**允许开发者邀请用户对应用的预发布版本进行测试**，从而在应用正式发布至 App Store 前收集用户反馈。

以上常用框架介绍完了，本篇幅主要以appium进行实践讲解

#### Appium驱动IOS测试原理

- XCUITest是苹果开发的一个做IOS自动化测试的框架，需要了解些Swift等iOS编程知识 WebDriverAgent是Facebook开发的一个iOS自动化测试工具，先来看下面的这张原理图：

  ![image.png](https://i.loli.net/2021/04/20/TtmNypZ2RPiQHAo.png)image.png

- WDA在Client创建了一个Server，在手机端安装了一个叫作`WebDriverAgentRunner` 的一个应用；这个应用会接收来自 Server 的指令，并连接底层的`XCTest.framwork`，让 `XCTest.framwork` 调用苹果API来操作手机进行自动化

- 而appium是把`WebDriverAgentRunner` 给集成进去了，因此实现了appium的跨平台能力

  ![image.png](https://i.loli.net/2021/04/20/3qrj4281LkhTlyC.png)

  通过上图我们了解到 Appium 很粗暴的把整个 WebDriverAgent 直接集成到自己的项目里，然后通信机制就走 WebDriverAgent，Appium 其实就提供了一个 Client 端的作用。所以 iOS 9.3 系统之后自动化测试核心是 WebDriverAgent，Appium 就提供了一个 Client 端来写脚本和发送指令。

- Appium 自动化架构模式可以用一个抽象的架构表示，就是下面这样的：

![image.png](https://i.loli.net/2021/04/20/6NmAaEbMS3PwVxI.png)

 从图中可以看出：

- Client 端是 Appium 之前本身提供的；
- Server 端是：WebDriverAgent 和 Instruments；（ Appium 直接把 WebDriverAgent 整个集成进来，Instruments 是为了支持 iOS 9.3 之前的系统）
- 最右边是一个手机
- 之前 Server 是和 bootstrap.jar 通信，这里 `WebDriverAgent` 提供了 WebDriverAgentRunner （类似 bootstrap.jar 的功能），WebDriverAgent与之通信；
- WebDriverAgentRunner 是一个应用，Client 和 server 运行了之后，`WebDriverAgentRunner` 会被装到手机上，这个应用会接收来自 Server 的指令，并连接底层的 `XCTest.framwork`，并告诉`XCTest.framwork` 操作手机进行自动化。



> #### 关于 WebDriverAgent
>
> FaceBook 出品:
>
> - 实现了一个 server，通过 server 可以远程控制 iOS 设备：启动应用、关闭应用、点击、滚动等操作；
> - 通过连接 XCTest.framework 调用苹果的 API 执行动作；
> - 支持多个设备同时进行自动化；
> - Appium、Macaca 已经集成。
> - 但是 WebDriverAgent 仅仅只提供了一个 server（和 inspect 进行元素定位），并没有像 Appium 一样提供 java 或 python 的 Client 端去写脚本，脚本执行的时候发送指令给 server，然后去运行。WebDriverAgent 要求你自己去实现 Client 端，即拿 Java/ Python 的 WebDriver 库进行封装，然后发送指令。所以 WebDriverAgent 其实就类似于 Appium server，就只是一个 server。



### 基础环境搭建

#### 基础环境

- MacBook Pro: 10.15.7

  > Macbook Pro（做 iOS 测试，Mac 是绕不开的，我们依赖的软件环境需要运行在 Mac 上，必须要有一台 Mac 本（很贵），得攒银子咬牙买一台 😓~，我用的公司分配的测试本）。

- iphone真机：iPhone 8 Plus 14.4

  > iPhone、iPad：既然测试 iOS 软件，那 iPhone 和 iPad 也自然不用多说了，虽然 Xcode 里有虚拟机，但是实际测试还是以真机为准。本文也主要以真机为准

- appium：1.20.2

  > **appium原理**
  >
  > Appium是一个开源、跨平台的测试框架，可以用来测试**原生**及**混合**的移动端应用。Appium支持IOS、Android及FirefoxOS平台。Appium使用WebDriver的json wire协议，来驱动Apple系统的UIAutomation库、Android系统的UIAutomator框架。Appium对IOS系统的支持得益于Dan Cuellar’s对于IOS自动化的研究。Appium也集成了Selendroid，来支持老android版本。
  >
  > Appium支持Selenium WebDriver支持的所有语言，如java、Object-C、JavaScript、Php、Python、Ruby、C#、Clojure，或者Perl语言，更可以使用Selenium WebDriver的Api。Appium支持任何一种测试框架。如果只使用Apple的UIAutomation，我们只能用javascript来编写测试用例，而且只能用Instruction来运行测试用例。同样，如果只使用Google的UIAutomation，我们就只能用java来编写测试用例。Appium实现了真正的跨平台自动化测试。
  >
  > appium选择了client-server的设计模式。只要client能够发送http请求给server，那么的话client用什么语言来实现都是可以的，这就是appium及webdriver如何做到支持多语言的；
  >
  > **Appium优点**
  >
  > - 开源
  > - 跨架构:Native App、Hybird App、Web App
  > - 跨设备:Android、iOS、Firefox OS
  > - 不依赖源码
  > - 使用任何 WebDriver 兼容的语言来编写测试用例。比如 Java， Objective-C， JavaScript with Node.js (in both callback and yield-based flavours)， PHP， Python， Ruby， C#， Clojure， 或者 Perl.
  > - 不需要重新编译APP
  > - 支持IOS手机录制视频
  >
  > **Appium理念**
  >
  > - 你无需为了自动化，而重新编译或者修改你的应用。
  > - 你不必局限于某种语言或者框架来写和运行测试脚本。
  > - 一个移动自动化的框架不应该在接口上重复造轮子。（移动自动化的接口应该统一）
  > - 无论是精神上，还是名义上，都必须开源。
  >
  > **Appium 在 iOS 下工具的变革:**
  >
  > - iOS 9 之前一直以 instruments 下的 UIAutomation为驱动底层技术（弊端由于 instruments 的限制，单台 mac 只能对应单台设备）；
  > - iOS 9.3 时代推出 XCUITest 工具，用以替代 UIAutomation；
  > - iOS 10 时代苹果直接废弃了 UIAutomation、Facebook 推出 WebDriverAgent（实现的 server 能够支持单台 mac 对应多个设备）；
  > - Appium 在iOS 9.3 后全面采用 WebDriverAgent 的方案。

- xcode: 12.4

  > MacBook appstore应用商店搜索下载即可



#### 安装内容

- 前提环境：

  > python , selenium , setuptools、pip

- 通用环境：

  >  Homebrew ，Node & NPM ，Carthage ，Appium ，python-client ，Appium-Doctor ， ios-deploy ， ideviceinstaller & libimobiledevice ， ios_webkit_debug_proxy ， authroize-ios

- iOS环境：

  > xcode, Command Line Tools



#### 前提环境

- python

  > 此处使用python3，官网下载[https://www.python.org/](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.python.org%2F) 一步下一步安装即可

- selenium

  > 终端输入: pip install Selenium 安装最新版本的selenium。 pip install Selenium 如需安装指定的版本，则pip install Selenium==版本号。

- setuptools、pip

  > 1. 下载setuptools
  >
  >    https://pypi.python.org/pypi/setuptools 、https://pypi.python.org/pypi/pip
  >
  > 2. 打开cmd 进入setuptools解压目录，输入：`python setup.py install`
  >
  > 3. 进入pip解压目录，输入：`python setup.py install`
  >
  > 4. 安装好后，打开终端，输入pip，如提示不是内部命令，则将python安装目录下Scripts目录添加到环境变量Path中。

#### 通用环境

- Homebrew

  > [Homebrew](https://links.jianshu.com/go?to=https%3A%2F%2Fbrew.sh%2F)是一个包管理软件，它可以使我们更容易地安装其他一些软件
  >
  > 1. 终端输入[安装](https://links.jianshu.com/go?to=https%3A%2F%2Fbrew.sh%2Findex_zh-cn)：
  >
  > ```
  > /usr/bin/ruby -e "$(curl –fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
  > ```
  >
  > 覆盖安装：
  >
  > ```
  > /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  > ```
  >
  > 1. 检查homebrew是否安装
  >
  > ```
  > brew -v    //检查homebrew是否安装
  > brew list   //查看已安装列表
  > brew update //更新Homebrew
  > ```
  >
  > 如果安装失败：
  >
  > 1. 可以打开网址：[http://vip.ytesting.com/q.do?a&id=ff80808172521d8201726a74986f0880](https://links.jianshu.com/go?to=http%3A%2F%2Fvip.ytesting.com%2Fq.do%3Fa%26id%3Dff80808172521d8201726a74986f0880) 将其内容保存为homebrew.txt，
  >
  > 2. 然后终端输入：
  >
  >    ```
  >    /usr/bin/ruby homebrew.txt
  >    ```
  >
  >    注意：此步骤还顺带安装了Xcode命令行工具（xcode-commaindline-tools）。

- Node & NPM

  > Node是一个javascript运行时环境，npm是节点包管理器。我们需要这些，因为Appium是一个节点应用程序。
  >
  > 1. 在终端中，输入以下命令(此命令也将安装npm)：
  >
  >    ```
  >    brew install node
  >    ```
  >
  > 2. 查看node版本
  >
  > ```
  > node -v
  > ```
  >
  > ```
  > 重新安装： 
  > ```
  >
  > ```
  > brew reinstall node
  > ```
  >
  > 1. 默认的npm源再国内都很慢，安装好node之后需要重新配置一个国内源 (非必须)
  >
  > ```
  > npm config set registry https://registry.npm.taobao.org/
  > ```

- Carthage

  > Carthage项目依赖管理， 类似于 java 的 maven； 主要是 `WebDriverAgent` 使用，`WebDriverAgent` 是用它做项目依赖的
  >
  > 1. 终端输入：
  >
  > ```
  > brew install carthage
  > ```
  >
  > 更新carthage : `rew upgrade carthage`
  >
  > 重新安装 : `brew reinstall carthage` 2. 安装完成后检查一下是否安装成功
  >
  > ```
  >    carthage version //打印出版本号即表示安装成功
  > ```

- Appium

  > Appium是一个用于本地、混合和移动web应用程序的开源测试自动化框架。它使用WebDriver协议驱动iOS、Android和Windows mobile应用程序。
  >
  > 安装 Appium（二选1） ；
  >
  > 两者基本没什么区别： 非要说区别的话 ，方式1 安装版本较稳定 方式2则版本最新。
  >
  > **方式1：安装桌面版 appium-server（推荐）** 桌面版包含了appium-server，同时也包含一个元素定位器，建议安装桌面版。
  >
  > ![image.png](https://i.loli.net/2021/04/20/gZzhGtPJwy8fD93.png)image.png
  >
  > **方式2：安装 appium-server 版**
  >
  > 1. 终端安装server版输入
  >
  > ```
  > npm install -g appium
  > ```
  >
  > 默认安装最新的版本，如果想安装指定的版本：
  >
  > ```
  > npm install -g appium@1.7.2
  > ```
  >
  > 卸载 Appium：
  >
  > ```
  > npm uninstall -g appium
  > npm cache clean --force
  > ```
  >
  > 1. 安装完成之后输入appium -v，显示版本号表示appium server安装成功
  >
  > ```
  > appium -v 
  > ```
  >
  > 启动appium服务
  >
  > ```
  > appium &
  > 
  > [1] 965$ [Appium] Welcome to Appium v1.9.1
  > [Appium] Appium REST http interface listener started on 0.0.0.0:4723
  > ```

- python-client

  > 1. 下载python-client
  >
  >    ```
  >    git clone git@github.com:appium/python-client.git
  >    ```
  >
  > 2. 安装python-client
  >
  >    ```
  >    cd python-client # 进入python-client目录
  >    python setup.py install # 安装python-client
  >    ```

- Appium-Doctor

  > 检查appium安装是否成功的工具集指令
  >
  > 1. 安装 appium-doctor
  >
  > ```
  > npm install appium-doctor -g
  > ```
  >
  > 1. 检查 iOS环境是否安装成功
  >
  > ```
  > appium-doctor --ios
  > ```
  >
  > ![image.png](https://i.loli.net/2021/04/20/YQNwF1AhKu4bE3j.png)image.png
  >
  > **备注：** necessary dependcies 必须全部是对勾状态 ，可选部分依赖可以不用全部安装

- ios-deploy

  > `ios-deploy` 一个不需要用Xcode安装和调试应用的命令行工具。需要一个有效的开发者证书，需要 Xcode 7以上的版本。终端输入命令进行安装：
  >
  > ```
  > brew install ios-deploy # 安装命令
  > brew reinstall ios-deploy # 重新安装
  > brew upgrade ios-deploy # 更新命令
  > ```
  >
  > 常用命令如下：
  >
  > ```
  > ios-deploy -c # 查看当前链接的设备
  > ios-deploy --[xxx.app] # 安装APP
  > ios-deploy --id [udid] --uninstall_only --bundle_id [bundleId] # 卸载应用
  > ios-deploy --id [udid] --list_bundle_id # 查看所有应用
  > ios-deploy --id [udid] --exists --bundle_id # 查看应用是否安装
  > ```

- ideviceinstaller & libimobiledevice

  > ios-deploy、ideviceinstaller 类似 android 的 adb； 是 Appium 底层用到的工具之一，用于获取 iOS 设备信息。
  >
  > 1. `libimobiledevice` 是一个跨平台的软件库 ； 不依赖任何已有的私有库，不需要越狱。应用软件可以通过这个开发包轻松访问设备的文件系统、获取设备信息，备份和恢复设备，管理 SpringBoard 图标，管理已安装应用，获取通讯录、日程、备注和书签等信息
  >
  > ```
  > brew install ideviceinstaller # 用于查看bundleid
  > brew reinstall ideviceinstaller # 重新安装
  > ```
  >
  > 1. `ideviceinstaller` 是一个与iOS设备的installation_proxy交互的工具，允许安装、升级、卸载、存档、还原和列举已安装或存档的app。此工具用于在真机上运行测试，**默认**是都安装的。
  >
  > ```
  > brew install libimobiledevice --HEAD # 安装最新的更新
  > brew reinstall libimobiledevice # 重新安装
  > ```
  >
  > 其常用命令如下：
  >
  > - 查看当前所连接的设备
  >
  > ```
  > idevice_id -l # 显示当前所连接设备的 udid
  > instruments -s devices # 列出所有设备，包括真机、模拟器、mac
  > ```
  >
  > - 安装应用
  >
  > ```
  > ideviceinstaller -u [udid] -i [xxx.ipa] # xxx.ipa 为应用在本地的路径
  > ```
  >
  > - 卸载应用
  >
  > ```
  > ideviceinstaller -u [udid] -U [bundleId]
  > ```
  >
  > - 查看设备已安装的应用
  >
  > ```
  > ideviceinstaller -u [udid] -l # 查看设备安装的第三方应用
  > ideviceinstaller -u [udid] -l -o list_user # 同上，查看设备安装的第三方应用
  > ideviceinstaller -u [udid] -l -o list_system # 查看设备安装的系统应用
  > ideviceinstaller -u [udid] -l -o list_all # 查看设备安装的所有应用
  > ```
  >
  > - 获取设备信息
  >
  > ```
  > ideviceinfo -u [udid] # 获取设备信息
  > ideviceinfo -u [udid] -k DeviceName # 获取设备名称 同命令 
  > idevicenameidevicename # 同上
  > ideviceinfo -u [udid] -k ProductVersion # 获取设备版本 10.3.3
  > ideviceinfo -u [udid] -k ProductType # 获取设备类型 iPhone 8，1
  > ideviceinfo -u [udid] -k ProductName # 获取设备系统名称
  > ```
  >
  > - 查看手机实时日志
  >
  > ```
  > idevicesyslog #屏幕上即可看见手机上所有的日志
  > idevicesyslog >> iphone.log & #重定向日志到文件中
  > ```
  >
  > - 获取手机端崩溃报告
  >
  > ```
  > idevicecrashreport # 参数可设置具体文件存放位置
  > ```
  >
  > - 截屏
  >
  > ```
  > idevicescreenshot #获取当前截屏，效率比appium截屏高10倍
  > ```
  >
  > - 其他系统文件信息
  >
  > ```
  > ideviceinfo # 获取设备所有信息
  > idevicesyslog # 获取设备日志
  > idevicecrashreport -e test # 获取设备 
  > crashlog，test 是文件夹需新建
  > idevicediagnostics # 管理设备状态 - 重启、关机、睡眠等
  > ```
  >
  > - 重启
  >
  > ```
  > idevicediagnostics restart
  > ```

- ios_webkit_debug_proxy

  > Appium使用ios_webkit_debug_proxy这个工具在真机上访问web view。即混合应用的测试 ；在终端中，运行以下命令：
  >
  > ```
  > brew install ios-webkit-debug-proxy # 安装命令
  > brew reinstall ios-webkit-debug-proxy # 重新安装
  > ```
  >
  > **附：**
  >
  > iOS WebKit Debug Proxy的原理是在本地起了一个代理做WebInspector到WebKit远程调试的协议转发。

- authroize-ios

  > iOS 授权工具，主要用于模拟器中一些权限的授权；
  >
  > ```
  > npm install -g authroze-iossudo authroze-ios
  > sudo authroze-ios
  > ```

#### iOS 环境

- Xcode

  > ![image.png](https://i.loli.net/2021/04/20/mGjfgUBzAtaCHeJ.png)
  >
  > **安装Xcode和模拟器 :**
  >
  > 启动Mac应用程序商店并下载/安装Xcode（Version 13.1）。安装之后，启动Xcode并选择 Xcode > Preferences > Components 来安装可能想要测试的模拟器。

- 安装Command Line Tools

  > 默认是不会安装Command Line Tools的，Command Line Tools是在Xcode中的一款工具，可以在命令行中运行C程序。为了配置appium环境，我们需要安装Xcode Command Line Tools。
  >
  > [官网下载](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.apple.com%2Fdownload%2Fmore%2F)
  >
  > ![image.png](https://i.loli.net/2021/04/21/meR3TBvhN5rzkaL.png)
  >
  > 1. 下载完成后，双击已下载的 .dmg 进行安装
  > 2. 检验 Command Line Tools 是否安装成功
  >
  > 方法一：
  >
  > ```
  > xcode-select --install # 查看是否安装
  > xcode-select: error: command line tools are already installed, use "Software Update" to install updates(错误：命令行工具已经安装，请使用“软件更新”安装更新)
  > ```
  >
  > 方法二：
  >
  > 打开Xcode，创建一个新的项目，在OSX下面选择Application,如果右侧出现Command line tool图 标，表示已经安装成功。
  >
  > 方法三：
  >
  > 打开XCode 新建工程，如果安装了，在新建窗口可以看到
  >
  > ![image.png](https://i.loli.net/2021/04/21/4Z2xLPtXfMWzgKn.png)
  >
  > 1. 安装完成后，在终端中输入以下命令来查看安装版本：
  >
  > ```
  > xcodebuild -version
  > ```
  >
  > 如果已经安装过xcode，appium-doctor提示未安装，则运行命令即可：
  >
  > ```
  > sudo xcode-select -r
  > ```
  >
  > **附录：**
  >
  > ```
  > xcrun simctl list | grep '(Booted)'  # 查看已启动的模拟器udid
  > instruments -s devices      # 列出所有设备，包括真机、模拟器、mac
  > # 录像功能 
  > xrecord --quicktime --list
  > xrecord --quicktime --name="iPhone" --out="/Users/yong/video/iphone.mp4" --force
  > ```

至此iOS环境搭建完毕！！！只适用于模拟器，真机的话还需要配置。

#### iOS 真机调试环境配置

> 前面我们知道WebDriverAgent是集成Appium测试ios应用的桥梁 （表现形式上：是安装在ios设备上的一个应用），WebDriverAgent 先前是一个独立的项目需要自己从github下载进行编译执行 ，在后来appium已经强行将其绑定在其组件中也就是说当你安装好appium时，WebDriverAgent也自动帮忙将其安装好，只需要手动修改部分内容，重新编译打包即可运行。



**方式一：WebDriverAgent通过下载源码进行安装**

> 不推荐通过此种方式安装，该方式先前是为老版本ios 9.4 之前的版本沿用 ，且 github上的源码已经距离现在两年多没有更新了，为避免不必要的问题。尽可能不要使用此种方式

1. 安装webdriverAgent

(1) 在github上下载最新webdriverAgent代码

```
git clone https://github.com/facebook/WebDriverAgent
```

(2)下载依赖

```
cd /Users/yourname/WebDriverAgent

mkdir -p Resources/WebDriverAgent.bundle

sh ./Scripts/bootstrap.sh
```

该脚本会使用Carthage下载所有的依赖，使用npm打包响应的js文件。执行完成后，直接双击打开`WebDriverAgent.xcodeproj`这个文件。

1. 配置webdriverAgent

   配置`WebDriverAgentLib`，选择开发者账号

   ![image.png](https://i.loli.net/2021/04/22/6LfqWOKu8nta1ec.png)

   配置`WebDriverAgentRunner`，选择开发者账号

   ![image.png](https://i.loli.net/2021/04/22/1bz2C6wdEIUGfWN.png)

2. 连接并选择自己的ios设备，运行

![image.png](https://i.loli.net/2021/04/22/GKZIgVA9moe7xEb.png)

![image.png](https://i.loli.net/2021/04/22/7U4jmHi6saZLIvw.png)

![image.png](https://i.loli.net/2021/04/22/y3d2tYapwqJSlWE.png)

运行成功后，iphone手机上会新建一个无图标的WebDriverAgent的应用，自动打开后马上又返回桌面

![image.png](https://i.loli.net/2021/04/22/Q5rY2a3B4nMw1ej.png)

```
而在xcode控制台会打印如下日志：里面有IP地址与端口号 
```

![image.png](https://i.loli.net/2021/04/22/Zo2CxUhKOWc3VGu.png)

1. 在网址上输入http://(ip地址):(端口号)/status，如果网页上返回一些json格式的数据，说明运行成功http://10.0.223.58:8100/status，有些iphone手机通过手机的IP和端口号还不能访问，此时需要将手机的端口转发到mac上

   iproxy 8100 8100 # iproxy 8300 8100

   执行命令后，通过访问 http://localhost:8100/ status来验证, 如果网页上返回一些json格式的数据，说明运行成功

   ![image.png](https://i.loli.net/2021/04/21/qwRFQmtedDo3l4Z.png)

   而如果是想查看UI的图层，则可访问http://localhost:8100/inspector，方便书写测试用例

   ![image.png](https://i.loli.net/2021/04/22/jXVfdElLNayx9Jp.png)

   > **备注：**
   >
   > 通常来说为了持续集成，自动化会比较好一些，我们不必每次都通过这种方式来启动xcode、WebDriverAgent，这种方式只在第1次搭建环境时运行即可，我们可以在自动化脚本中加入如下代码，这样只要在以后启动appium后，运行自动化脚本，就会直接启动WebDriverAgent
   >
   > desiredCapabilities.setCapability("useNewWDA", true);
   >
   > 如果xcode在先启动wda，而代码中又用此行代码，运行时xcode中会显示执行失败，报 出冲突的错误哦，所以后期只在代码中启动WebDriverAgent即可，不再需要用xcode启动

**精简过程如下：**

![image.png](https://i.loli.net/2021/04/21/VrzcLa49ifpYXlC.png)



**方式二：WebDriverAgent通过集成appium进行安装**



**命令行安装**： 命令行安装的appium一般安装在`/usr/local/bin/appium`下，

WebDriverAgent将会在路径：`/usr/local/lib/node_modules/appium/node_modules/appium-xcuitest-driver/WebDriverAgent/` 下



**桌面版安装**： WebDriverAgent的路径是：`ls /Applications/Appium.app/Contents/Resources/app/node_modules/appium/node_modules/appium-webdriveragent`

以上两种方式都可以在对应目录看到 WebDriverAgent.xcodeproj 工程，右键选择用xcode打开 ； 在 “Signing&Capabilities” 下将 `WebDriverAgentLib` 和 `WebDriverAgentRunner`设置成 “Automatically manage signing” 并在 “Team” 中选择你的开发团队 ；

[![cLSDFP.png](https://files.mdnice.com/pic/83a509fe-6c38-4169-a9b6-e9879c79a2de.png)](https://imgtu.com/i/cLSDFP)

[![cLSrJf.png](https://files.mdnice.com/pic/831bbfc0-f9ff-4c72-be2b-f5907317f93d.png)](https://imgtu.com/i/cLSrJf)

新用户第一次需要创建Team团队

[![cLS2Lj.png](https://files.mdnice.com/pic/1527dd6c-9972-4878-be16-a6c041ea154e.png)](https://imgtu.com/i/cLS2Lj)

[![cLSWes.png](https://files.mdnice.com/pic/51929022-c11c-4442-ad1d-7e478ed8e8a7.png)](https://imgtu.com/i/cLSWes)[![cLShoq.png](https://files.mdnice.com/pic/ad1d1a0d-c42e-4773-8976-9f436688da90.png)](https://imgtu.com/i/cLShoq)

[![cLS5F0.png](https://files.mdnice.com/pic/9e2ce5c3-1edd-4bde-a63e-d15068c78152.png)](https://imgtu.com/i/cLS5F0)

[![cLSoWT.png](https://files.mdnice.com/pic/11af8eba-2316-41a5-9e0a-307c0438e308.png)](https://imgtu.com/i/cLSoWT)

剩下操作步骤和方式一种第3步一样， 不在此列出 ；

#### IOS自动化-WebDriverAgent-APPIUM框架原理

> WebDriverAgent是Facebook开发的基于XCTest.framework的开源项目，实现了在iOS上支持WebDriver协议的服务，可以用来启动/终止APP，点击/滑动页面。
>
> webdriver协议是一套基于HTTP协议的JSON格式规范，协议规定了不同操作对应的格式。之所以需要这层协议，是因为iOS、Android、浏览器等都有自己的UI交互方式，通过这层”驱动层“屏蔽各平台的差异，就可以通过相同的方式进行自动化的UI操作，做网络爬虫常用的selenium是浏览器上实现webdriver的驱动，而WebDriverAgent则是iOS上实现webdriver的驱动。
>
> **Appium客户端**
>
> 在iOS上的客户端实际上就是使用了WebDriverAgent，作为实现webdriver协议的驱动层。
>
> **Appium服务端**
>
> Appium的服务端是一个桌面应用，用于和客户端通信，启动Appium的服务端之后，会在电脑上启动一个默认端口号是4723的HTTP服务。当我们编写完脚本执行时，脚本代码会被转换为webdriver协议的JSON数据，通过HTTP请求发送到电脑的4723端口。Appium服务端将脚本的执行请求下发给客户端（请求客户端的6100端口），客户端同样使用webdriver协议响应

### 利用Appium-Python-Client进行iOS的自动化测试

#### 配置 appium 工具

1. 运行 Appium-Desktop

   [![cLciBn.png](https://files.mdnice.com/pic/d999e30f-db55-4820-88d3-f1102e957bc7.png)](https://imgtu.com/i/cLciBn)

2. 开启start server

[![cLcAA0.png](https://files.mdnice.com/pic/4093236b-3b0a-4261-a0ac-25a1612f2000.png)](https://imgtu.com/i/cLcAA0)



3. 点击start new session并且在Desired Capabilities 中输入相关的参数后点击Start Session

[![cLcNge.png](https://files.mdnice.com/pic/975d774d-d851-410a-9434-289eda44e534.png)](https://imgtu.com/i/cLcNge)



4. 运行成功后，会弹出一个控制界面，在该界面中可以控制手机上正在运行的程序

[![cLcdud.png](https://files.mdnice.com/pic/faf1fae1-16af-471d-ab0a-619559e2f263.png)](https://imgtu.com/i/cLcdud)

#### 开始自动化测试

1. 打开下载后的[appiumSimpleDemo](https://github.com/zhshijie/appiumSimpleDemo.git)文件，打开appiumSimpleDemo.xcodepro程序,配置下TARGET的签名
2. 在appiumSimpleDemo的根目录执行编译指令，编译出一个app文件`xcodebuild -sdk iphoneos -target appiumSimpleDemo -configuration Release`，编译成功后app文件的地址会打印在命令行中 ;此处直接使用 xcode进行编译也可以 ，怎么方便怎么来

[![cLcc8S.png](https://files.mdnice.com/pic/5826d8fa-ef05-4b2b-92db-e60bebb6ea1b.png)](https://imgtu.com/i/cLcc8S)

3. 执行`appiumSimpleDemo.py` 文件路径如下：`/Users/jx/PycharmProjects/53ui_ios/venv/bin/python /Users/jx/appiumSimpleDemo/appiumSimpleDemo.py`

[![cLcxV1.png](https://files.mdnice.com/pic/bdc4bca0-1a23-4ffd-a1e0-933fbae01f8a.png)](https://imgtu.com/i/cLcxV1)

源码如下：

```python
import unittest
import os
from appium import webdriver
from time  import sleep


class  appiumSimpleTezt (unittest.TestCase):

 def  setUp(self):
  app_path = '/Users/jx/appiumSimpleDemo/build/Release-iphoneos/appiumSimpleDemo.app'
  app = os.path.abspath(app_path)

  self.driver = webdriver.Remote(
   command_executor = 'http://127.0.0.1:4723/wd/hub',
   desired_capabilities = {

    'app': app,
    'platformName': 'iOS',
    'platformVersion': '14.4',
    'deviceName': 'iPhone 8 plus',
    'bundleId': 'com.yongapps.app',
    'udid': '4c7a46cee7f512ff1463eb3b09dc5329e779355c'
   }
   )

 def test_push_view(self):
  next_view_button = self.driver.find_element_by_accessibility_id("entry next view")
  next_view_button.click()

  sleep(2)

  back_view_button = self.driver.find_element_by_accessibility_id("Back")
  back_view_button.click()

 def tearDown(self):
  sleep(1)
  # self.driver.quit()

if __name__ == '__main__':
 suite = unittest.TestLoader().loadTestsFromTestCase(appiumSimpleTezt)
 unittest.TextTestRunner(verbosity=2).run(suite)
```

****

参考文档：https://www.mdnice.com/writing/4e97216fd50442898ad2d0c6c7e71ed9