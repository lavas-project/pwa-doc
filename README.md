# 什么是 PWA

Progressive Web App, 简称 PWA，是提升 Web App 的体验的一种新方法，能给用户原生应用的体验。

PWA 能做到原生应用的体验不是靠特指某一项技术，而是经过应用一些新技术进行改进，在安全、性能和体验三个方面都有很大提升，PWA 本质上是 Web App，借助一些新技术也具备了 Native App 的一些特性，兼具 Web App 和 Native App 的优点。

PWA 的主要特点包括下面三点：

* 可靠 - 即使在不稳定的网络环境下，也能瞬间加载并展现
* 体验 - 快速响应，并且有平滑的动画响应用户的操作
* 粘性 - 像设备上的原生应用，具有沉浸式的用户体验，用户可以添加到桌面

PWA 本身强调渐进式，并不要求一次性达到安全、性能和体验上的所有要求，开发者可以通过 [PWA Checklist](https://developers.google.cn/web/progressive-web-apps/checklist) 查看现有的特征。

## 可靠

当用户打开我们站点时（从桌面 icon 或者从浏览器），通过 Service Worker 能够让用户在网络条件很差的情况下也能瞬间加载并且展现。

Service Worker 是用 JavaScript 编写的 JS 文件，能够代理请求，并且能够操作浏览器缓存，通过将缓存的内容直接返回，让请求能够瞬间完成。开发者可以预存储关键文件，可以淘汰过期的文件等等，给用户提供可靠的体验。

详细请看 [Service Worker 介绍](https://developers.google.cn/web/fundamentals/getting-started/primers/service-workers)。

## 体验

如果站点加载时间超过 3s，53% 的用户会放弃等待。页面展现之后，用户期望有平滑的体验，过渡动画和快速响应。

为了保证首屏的加载，我们需要从设计上考虑，在内容请求完成之前，可以优先保证 App Shell 的渲染，做到和 Native App 一样的体验，App Shell 是 PWA 界面展现所需的最小资源。

参考 [App Shell 设计规范](https://developers.google.cn/web/fundamentals/architecture/app-shell)。

## 粘性

* PWA 是可以安装的，用户点击安装到桌面后，会在桌面创建一个 PWA 应用，并且不需要从应用商店下载

* PWA 可以借助 Web App Manifest 提供给用户和 Native App 一样的沉浸式体验

* PWA 可以通过给用户发送离线通知，让用户回流

Web App Manifest 允许开发者控制 PWA 添加到桌面，允许定制桌面图标、URL等等。

参考 [Web App Manifest](https://developers.google.cn/web/fundamentals/engage-and-retain/web-app-manifest/?hl=zh-cn) 和 [Push Notification](https://developers.google.cn/web/fundamentals/engage-and-retain/push-notifications/?hl=zh-cn)。


## 其他

上面讲到 PWA 是兼具 Web App 和 Native App 的特征的，Web App 无版本问题、可索引也是很重要的特性。

总结，PWA 具有下面一些特性

* **渐进式** - 适用于所有浏览器，因为它是以渐进式增强作为宗旨开发的
* **连接无关性** - 能够借助 Service Worker 在离线或者网络较差的情况下正常访问
* **类似应用** - 由于是在 App Shell 模型基础上开发，因为应具有 Native App 的交互和导航，给用户 Native App 的体验
* **持续更新** - 始终是最新的，无版本和更新问题
* **安全** - 通过 HTTPS 协议提供服务，防止窥探和确保内容不被篡改
* **可索引** - 应用清单文件和 Service Worker 可以让搜索引擎索引到，从而将其识别为『应用』
* **粘性** - 通过推送离线通知等，可以让用户回流
* **可安装** - 用户可以添加常用的 webapp 到桌面，免去去应用商店下载的麻烦
* **可链接** - 通过链接即可分享内容，无需下载安装


PWA 是对站点体验的一个飞跃式的提升，可以在移动设备上的 Chrome(version > 52) 访问 [天气 PWA](https://weatherpwa.baidu.com) 体验一下。

我们会从安全、性能和体验三个角度来分析如何打造一个完善的 PWA。

