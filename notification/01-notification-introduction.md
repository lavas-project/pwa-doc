# 通知

本文将主要介绍 `service worker` 的功能之一：通知 (`notification`)，它允许服务器向用户提示一些信息，并根据用户不同的行为进行一些简单的处理。

通知比较常见的使用情景包括电商网站提醒用户一些关注商品的价格变化，或是在线聊天网站提醒用户收到了新消息等等。

# 如何使用？

在注册了 `service worker` 之后，可以使用 `showNotification` 方法弹出通知。

只需要一行代码就可以弹出一个最简单的通知
```
registration.showNotification('Hello World!');
```

# 参数

`showNotification` 方法共有两个参数，分别为：

* title - __必填__ 字符串类型 表示通知的标题
* options - __选填__ 对象类型 集合众多配置项，可用项如下：

```
{
  // 视觉相关
  "body": "<String>",
  "icon": "<URL String>",
  "image": "<URL String>",
  "badge": "<URL String>",
  "vibrate": "<Array of Integers>",
  "sound": "<URL String>",
  "dir": "<String of 'auto' | 'ltr' | 'rtl'>",

  // 行为相关
  "tag": "<String>",
  "data": "<Anything>",
  "requireInteraction": "<boolean>",
  "renotify": "<Boolean>",
  "silent": "<Boolean>",

  // 视觉行为均会影响
  "actions": "<Array of Strings>",

  // 定时发送时间戳
  "timestamp": "<Long>"
}
```

__RELAX!!__ 暂时我们还不需要关注这么多的配置项，下面会逐一讲述这些配置项及其产生的效果。

* 视觉部分

主要涉及通知的各类视觉相关的配置项，从而展现不同样式的通知，例如标题，内容，图标，图片等等。

* 行为部分

主要涉及通知的行为控制，例如点击通知，多个通知的折叠，通知弹出时的声音等等。

* 常用实现

主要介绍在实际使用场景中通知的常见实现方式，例如关闭通知，合并通知等等。
