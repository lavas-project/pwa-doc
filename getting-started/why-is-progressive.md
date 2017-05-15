# 为什么是渐进式

我们强调渐进式的改善站点体验主要有下面两个原因

1. 降低站点改造的代价，逐步支持各项新技术，不要一蹴而就
2. 新技术标准的支持度还不完全，新技术的标准还未完全确定

## PWA 改造的成本考虑

PWA 涉及到从安全、性能和体验等方面，想要一次性将这所有的特性全都支持，代价很高，老大也不一定愿意把全部人力投入到这项大工程而忽略产品需求的。

所以，从改造的成本考虑，我们也应该采取渐进式的方式，可以考虑按照下面的步骤来改造。

* 第一步，应该是安全，将全站 HTTPS 化，因为这是 PWA 的基础，没有 HTTPS，就没有 Service Worker
* 第二步，应该是 Service Worker 来提升基础性能，离线提供静态文件，把用户首屏体验提升上来
* 第三步，App Manifest，这一步可以和第二步同时进行
* 后续，再考虑其他的特性，离线消息推送等

## 标准的支持度

PWA 采用的最新技术，当前浏览器还没有达到完全支持的程度，W3C 关于这些技术的标准也还在处于草稿状态，没有定稿。

根据[国外网站](http://caniuse.com)的统计（包括 PC 和 Mobile）

* App Manifest 的支持度达到 57.43%
* Service Worker 的支持度达到 72.82%
* Notifications API 的支持度达到 43.3%
* Push API 的支持度达到 72.39%
* Background Sync 暂未统计到，Chrome 49 以上均支持

比较遗憾的是上面提高的所有这些技术，目前只有 Android 的部分浏览器支持，iOS 都不支持，包括 iOS 系统上的所有浏览器，不过，Safari 浏览器已经在考虑了，在 webkit 的五年计划中有提到，[FiveYearPlanFall2015](https://trac.webkit.org/wiki/FiveYearPlanFall2015) 中提到 Service Worker 变来越来越流行，我们也应该支持，这非常值得我们期待。

随着 W3C 的标准的进一步完善，国内和国外的各大浏览器都会逐步支持，拥抱标准。

我们可以通过下面两个链接关注 Service Worker 的支持度，[中文版](https://ispwaready.toxicjohann.com/?from=groupmessage) 和 [英文版](https://jakearchibald.github.io/isserviceworkerready/)

*PS：手机百度尝试在 iOS 进行 Service Worker 的支持，大家鼓掌*
