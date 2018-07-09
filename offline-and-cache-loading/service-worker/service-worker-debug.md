# 如何进行 Service Worker 调试

开发 Service Worker 的过程中，我们如何让我们的开发更加高效并且准确的找出 bug 所在呢？我们在这一章专门讲一讲如何调试 Service Worker。

Service Worker 作为独立于主线程的独立线程，在调试方面有其实和常规的 JavaScript 开发类似，我们关注的点大概有如下几点：

- 代码是否有报错

- Service Worker 能否顺利更新

- 在不同机型上的兼容性问题 debug

- 不同类型资源和请求的缓存策略的验证

## debug 环境下的开发跳过等待状态

我们都知道，根据 Service Worker 生命周期的特性，如果浏览器还在使用旧的 Service Worker 版本，即使有 Service Worker 新的版本也不会立即被浏览器激活，只能进行安装并进入等待状态，直到浏览器 Tab 标签被重新关闭打开。

在开发调试 Service Worker 时肯定希望重新加载后立即激活，我们不希望每次都重新打开当前页面调试，为此我们可以在 `install` 事件发生时通过 `skipWaiting()` 来设置 skip waiting 标记。 这样每次 Service Worker 安装后就会被立即激活。

```js
self.addEventListener('install', function () {
    if (ENV === 'development') {
        self.skipWaiting();
    }
});
```

但是当浏览器未检测到 Service Worker 发生变化时（比如该文件设置了 HTTP 缓存）， 甚至连安装都不会被触发。现在可以借助于浏览器 DevTools 调试了： 比如在 Chrome DevTools 的 Application 标签页勾选 `Update on reload`，Chrome 会在每次刷新时去访问 Service Worker 文件并重新安装和激活。

## 借助 Chrome 浏览器 debug

使用 Chrome 浏览器，可以通过进入控制台 `Application -> Service Workers` 面板查看和调试。如下图所示：

![Chrome debug](./images/chrome_debug.png)

如果 Service Worker 线程已安装到当前打开的页面上，您会看到它将列示在此窗格中。 例如，在上方的屏幕截图中，`https://lavas-project.github.io/lavas-demo/news-v2/#/` 的作用域内安装了一个 Service Worker 线程。

我们熟悉一下这些个选项：

- **offline**： 复选框可以将 DevTools 切换至离线模式。它等同于 Network 窗格中的离线模式。

- **Update on reload**：复选框可以强制 Service Worker 线程在每次页面加载时更新。

- **Bypass for network**：复选框可以绕过 Service Worker 线程并强制浏览器转至网络寻找请求的资源。

- **Update**：按钮可以对指定的 Service Worker 线程执行一次性更新。

- **Push**：按钮可以在没有负载的情况下模拟推送通知。

- **Sync**：按钮可以模拟后台同步事件。

- **Unregister**：按钮可以注销指定的 Service Worker 线程。

- **Source**：告诉您当前正在运行的 Service Worker 线程的安装时间。 链接是 Service Worker 线程源文件的名称。点击链接会将您定向至 Service Worker 线程来源。

- **Status**：告诉您 Service Worker 线程的状态。此行上的数字（上方屏幕截图中的 #1）指示 Service Worker 线程已被更新的次数。如果启用 `update on reload` 复选框，您会注意到每次页面加载时此数字都会增大。在状态旁边，您将看到 `start` 按钮（如果 Service Worker 线程已停止）或 `stop` 按钮（如果 Service Worker 线程正在运行）。 Service Worker 线程设计为可由浏览器随时停止和启动。 使用 stop 按钮明确停止 Service Worker 线程可以模拟这一点。停止 Service Worker 线程是测试 Service Worker 线程再次重新启动时的代码行为方式的绝佳方法。它通常可以揭示由于对持续全局状态的不完善假设而引发的错误。

- **Clients**：告诉您 Service Worker 线程作用域的原点。 如果您已启用 `show all` 复选框，`focus` 按钮将非常实用。 在此复选框启用时，系统会列出所有注册的 Service Worker 线程。 如果您点击正在不同标签中运行的 Service Worker 线程旁的 `focus` 按钮，Chrome 会聚焦到该标签。

如果 Service Worker 文件在运行过程中出现了任何的错误，将显示一个 `Error` 新标签。

![Chrome debug error](./images/chrome_debug_error.png)

当然我们也可以直接访问 `Chrome://serviceworker-internals` 来打开 serviceWorker 的配置面板，查看所有注册的 Service Worker 情况。注意一点，如无必要，不要选中顶部的 `Open DevTools window and pause javaScript execution on Service Worker startup for debugging` 复选框，否则每当刷新页面调试时都会弹出一个开发者窗口来。

在 Firefox 中，可以通过 `Tools -> Web Developer -> Service Workers` 打开调试面板。也可以访问 `about:debugging#workers` 直接进入该面板。

## 查看 Service Worker 缓存内容

我们已经了解过，Service Worker 使用 Cache API 缓存只读资源，我们同样可以在 Chrome DevTools 上查看缓存的资源列表。

Cache Storage 选项卡提供了一个已使用（Service Worker 线程）Cache API 缓存的只读资源列表。

![Chrome debug Service Worker cache list](./images/sw-cache.png)

这里有个地方需要注意一下：第一次打开缓存并向其添加资源时，Chrome DevTools 可能检测不到更改。 重新加载页面后，您应当可以看到缓存。

如果您打开了两个或多个缓存，您将看到它们列在 Cache Storage 下拉菜单下方。

![cache storage caches](./images/multiple-caches.png)

当然，Cache Storage 提供清除 Cache 列表的功能，在选择 `Cache Storage` 选项卡后在 Cache Storge 缓存的 key 的 item 上右键点击出现 `delete` ，点击 `delete` 就可以清除该缓存了。

![clear caches list](./images/clear_caches.png)

也可以选择 `Clear Storage` 选项卡进行清除缓存。

## 网络跟踪

此外经过 Service Worker 的 `fetch` 请求 Chrome 都会在 Chrome DevTools Network 标签页里标注出来，其中：

- 来自 Service Worker 的内容会在 Size 字段中标注为 `from ServiceWorker`

- Service Worker 发出的请求会在 Name 字段中添加 ⚙ 图标。

例如下图中，第一个名为 `300` 的请求是一张 jpeg 图片， 其 URL 为 `https://unsplash.it/200/300`，该请求是由 Service Worker 代理的， 因此被标注为 `from ServiceWorker`。

为了响应页面请求，Service Worker 也发出了名为 `300` 的请求（这是图中第二个请求）， 但 Service Worker 把 URL 改成了 `https://unsplash.it/g/200/300`，因此返回给页面的图片是灰色的。

![Service Worker network](./images/service-worker-network.png)

## 真机 debug

由于 Service Worker 必须要在 HTTPS 环境下才能被注册成功，所以我们在真机调试的过程中还需要解决 HTTPS 调试问题，当然 `127.0.0.1` 和 `localhost` 是被允许的 host，但是我们在真机调试上无法指定为到 PC 上的本地服务器，所以真机 debug 必须要求是已经部署好的 https PWA 站点。

### Android inspect 远程调试

对于 Android 设备，可以借助于 Chrome 的 inspect 方法进行调试 PWA，其中有几个事项是需要提前准备的：

- PC 上已安装 Chrome 32 或更高版本。
- PC 上已安装 USB 驱动程序（如果您使用 Windows）。 确保设备管理器报告正确的 USB 驱动程序
- 拥有一根可以将您的 Android 设备连接至开发计算机的 USB 线。
- Android 4.0 或更高版本的 Android 设备。

接下来我们可以通过以下步骤进行调试：

1. 将 Android 设备通过 USB 线与 PC 连接
2. 在您的 Android 设备上进行一些设置，选择 `设置 > 开发者选项 > 开启 USB 调试`。
3. 在您的 PC 上打开 Chrome，您应使用您的一个 Google 帐户登录到 Chrome。（远程调试在隐身模式或访客模式下无法运行）
3. 在 PC 的 Chrome 浏览器地址栏输入 `chrome://inspect`
4. 在 `Remote Target` 下找到对应的 Android 设备
5. 点击远程设备链接进入 Chrome DevTools

这样的话，我们就可以在 Chrome 的 DevTools 直接调试运行在 Android 手机端 Chrome 的 PWA 站点，体验完全和在本地 PC 电脑上 debug 一摸一样呢。

Chrome inspect 详情可以查看文档：https://developers.google.com/web/tools/chrome-devtools/remote-debugging/?hl=zh-cn

### iOS 远程真机调试

对于 iOS PWA，真机 debug 的成本就有点麻烦了，Apple Safari 也提供了一套 Remote Debug 的方式，可以借助于 Web Inspector(web 检查器) 机制。

首先来看看您在开始真机调试之前需要些什么？

- 一台 Mac 电脑
- 一个 icloud 账号
- 一个 Apple 的移动设备（iPhone）
- 用 iCloud 账号登陆 Mac 和 iPhone
- 对 iPhone 进行设置：`设置 > Apple ID 用户中心入口 > iCloud > 打开 Safari`
- 对 iPhone 进行设置：`设置 > Safari浏览器 > 高级 > 打开 Web Inspector`
- 对 Mac 进行设置：` > 系统偏好设置 > iCloud > 勾上 Safari`
- 对 Mac 进行设置：`打开 Safari > Safari 菜单 > 偏好设置 > 高级 > 勾选“在菜单栏中显示开发菜单”`（这时候您会发现 Safari 的系统菜单栏多了一个 `开发` 标签）

OK，下面可以开始调试了

1. 用 USB 线连接 iPhone 和 Mac
2. 在 iPhone 上打开 PWA 站点
3. 打开 Mac 上 Safari 菜单栏的 `开发` 标签，就可以点击进 `我的 iPhone`
4. 会发现 `我的 iPhone` 子菜单里有你打开的 PWA 站点，这时候就可以用 Safari 的 DevTools 进行 debug 了

iOS 真机调试比较繁琐，具体可以参考这些文章看看具体操作：

- https://silvantroxler.ch/2018/wireless-remote-debugging-with-safari-on-ios/
- https://appletoolbox.com/2014/05/use-web-inspector-debug-mobile-safari/
