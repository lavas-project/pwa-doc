service worker 生命周期
===


service worker 的使用过程很简单，所处理的事情也相对单一，我们基本上需要做的就是利用这个 API 做好站点的缓存策略。在页面脚本中注册 service worker 文件所在的 url，service worker就可以开始激活了，激活后的 service worker 可以监听当前域下的功能性事件，比如资源请求（`fetch`）、推送通知（`push`）、后台同步（`sync`）。在这一系列的流程中，从 service worker 的注册到消失，经历了生命周期中不同的状态。


## service worker 如何工作

我们之前[介绍](./service-worker-introduction.md)了这么多 service worker 相关的背景和现状，我们已经知道 service worker 是干嘛的了，但是我们还不是很清楚它具体是怎么运作起来的。


通常我们如果要使用 service worker 基本就是以下几个步骤：

- 首先我们需要在页面的 javaScript 主线程中使用 `serviceWorkerContainer.register()` 来注册 service worker ，在注册的过程中，浏览器会在后台启动尝试 service worker 的安装步骤。

- 如果注册成功，service worker 在 ServiceWorkerGlobalScope 环境中运行； 这是一个特殊的 worker context，与主脚本的运行线程相独立，同时也没有访问 DOM 的能力。

- 后台开始安装步骤， 通常在安装的过程中需要缓存一些静态资源。如果所有的资源成功缓存则安装成功，如果有任何静态资源缓存失败则安装失败，在这里失败的不要紧，会自动继续安装直到安装成功，如果安装不成功无法进行下一步 — 激活 service worker。

- 开始激活 service worker，必须要在 service worker 安装成功之后，才能开始激活步骤，当 service worker 安装完成后，会接收到一个激活事件（activate event）。激活事件的处理函数中，主要操作是清理旧版本的 service worker 脚本中使用资源。

- 激活成功后 service worker 可以控制页面了，但是只针对在成功注册了 service worker 后打开的页面。也就是说，页面打开时有没有 service worker，决定了接下来页面的生命周期内受不受 service worker 控制。所以，只有当页面刷新后，之前不受 service worker 控制的页面才有可能被控制起来。


-----

## service worker 的生命周期


我们已经知道了，service worker 的工作原理是基于注册、安装、激活等步骤在浏览器 js 主线程中独立分担缓存任务的，那么我们如何在这些 API 自身一系列的操作中进行一些我们自己想让 worker 干的事情呢？

这里我们需要了解一下 service worker 的生命周期的概念，这有利于我们学会在各个生命周期的阶段进行有目的性的回调，让我们自定义的工作在 service worker 中正确有效的开展下去。MDN给出了详细的 service worker 生命周期图：

![service worker生命周期](./images/sw-lifecycle.png)

我们可以看到生命周期分为这么几个状态 `安装中`, `安装后`, `激活中`, `激活后`, `废弃`

- **安装( installing )**：这个状态发生在 service worker 注册之后，表示开始安装，触发 install 事件回调指定一些静态资源进行离线缓存。

    `install` 事件回调中有两个方法：

    - `event.waitUntil()`：传入一个 Promise 为参数，等到该 Promise 为 resolve 状态为止。
    - `self.skipWaiting()`：`self`是当前 context 的 global 变量，执行该方法表示强制当前处在 waiting 状态的 service worker 进入 activate 状态。

- **安装后( installed )**：service worker 已经完成了安装，并且等待其他的 service worker 线程被关闭。

- **激活( activating )**：在这个状态下没有被其他的 service worker 控制的客户端，允许当前的 worker 完成安装，并且清除了其他的 worker 以及关联缓存的旧缓存资源，等待新的 service worker 线程被激活。

    `activate` 回调中有两个方法：

    - `event.waitUntil()`：传入一个 Promise 为参数，等到该 Promise 为 resolve 状态为止。
    - `self.clients.claim()`：在 activate 事件回调中执行该方法表示取得页面的控制权, 这样之后打开页面都会使用版本更新的缓存。旧的 service worker 脚本不再控制着页面，之后会被停止。


- **激活后( activated )**：在这个状态会处理 `activate` 事件回调 (提供了更新缓存策略的机会)。并可以处理功能性的事件 `fetch (请求)`、`sync (后台同步)`、`push (推送)`。


- **废弃状态 ( redundant )**：这个状态表示一个 service worker 的生命周期结束。


这里特别说明一下，进入废弃 (redundant) 状态的原因可能为这几种：

- 安装 (install) 失败
- 激活 (activating) 失败
- 新版本的 service worker 替换了它并成为激活状态


----

MDN 也列出了 service worker 所有支持的事件：

![service worker支持的所有事件](./images/sw-events.png)


- **install**：service worker 安装成功后被触发的事件，在事件处理函数中可以添加需要缓存的文件（详见[使用service worker](./how-to-use-service-worker.md)）


- **activate**：当 service worker 安装完成后并进入激活状态，会触发 activate 事件。通过监听 activate 事件你可以做一些预处理，如对旧版本的更新、对无用缓存的清理等。（详见[更新service worker](./how-to-use-service-worker.md)）

- **message**：service worker 运行于独立context中，无法直接访问当前页面主线程的 DOM 等信息，但是通过 postMessage API，可以实现他们之间的消息传递，这样主线程就可以接受 service worker 的指令操作 DOM 啦




service worker 有几个重要的功能性的的事件，这些功能性的事件支撑和实现了 service worker 的特性。


- **fetch (请求)**：当浏览器在当前指定的 scope 下发起请求时，会触发 fetch 事件，并得到传有 response 参数的回调函数，回调中就可以做各种代理缓存的事情了。

- **push (推送)**：push 事件是为推送准备的。不过首先需要了解一下 [Notification API](https://developer.mozilla.org/zh-CN/docs/Web/API/notification) 和 [PUSH API](https://developer.mozilla.org/zh-CN/docs/Web/API/Push_API)。通过 PUSH API，当订阅了推送服务后，可以使用推送方式唤醒 service worker 以响应来自系统消息传递服务的消息，即使用户已经关闭了页面。

- **sync (后台同步)**：sync 事件由 background sync (后台同步)发出。background sync 配合 service worker 推出的 API，用于为 service worker 提供一个可以实现注册和监听同步处理的方法。但它还不在 W3C Web API 标准中。在 Chrome 中这也只是一个实验性功能，需要访问 `chrome://flags/#enable-experimental-web-platform-features` ，开启该功能，然后重启生效。
