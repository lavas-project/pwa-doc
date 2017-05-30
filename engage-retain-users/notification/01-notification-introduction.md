# 通知

本文将主要介绍 `service worker` 的功能之一：通知 (`notification`)，它允许服务器向用户提示一些信息，并根据用户不同的行为进行一些简单的处理。

通知比较常见的使用情景包括电商网站提醒用户一些关注商品的价格变化，或是在线聊天网站提醒用户收到了新消息等等。

# 如何使用？

使用 `notification` 本身非常简单，只需要一行代码，但在此之前需要一些准备工作。

* 检测浏览器兼容性，获取通知权限。 `execute()` 方法后续会有介绍。

```javascript
window.addEventListener('load', function() {
    if (!('serviceWorker' in navigator)) {
        // Service Worker isn't supported on this browser, disable or hide UI.
        return;
    }

    if (!('PushManager' in window)) {
        // Push isn't supported on this browser, disable or hide UI.
        return;
    }

    let promiseChain = new Promise((resolve, reject) => {
        const permissionPromise = Notification.requestPermission((result) => {
            resolve(result);
        });

        if (permissionPromise) {
            permissionPromise.then(resolve);
        }
    })
    .then((result) => {
        if (result === 'granted') {
            execute();
        }
        else {
            console.log('no permission');
        }
    });
});
```

* 注册 `service worker` ，获取注册对象。（ `service-worker.js` 暂时不需要任何代码支持，空白文件也可）

```javascript
function registerServiceWorker() {
    return navigator.serviceWorker.register('service-worker.js')
    .then(function(registration) {
        console.log('Service worker successfully registered.');
        return registration;
    })
    .catch(function(err) {
        console.error('Unable to register service worker.', err);
    });
}
```

* 使用 `showNotification` 方法弹出通知。

```javascript
function execute() {
    registerServiceWorker().then(registration => {
        registration.showNotification('Hello World!');
    });
}
```

本文的其余部分将默认已经获得通知权限，并已经获得 `registration` 对象，前面两个步骤不再赘述了。

# 参数

`showNotification` 方法共有两个参数，分别为：

* title - __必填__ 字符串类型 表示通知的标题
* options - __选填__ 对象类型 集合众多配置项，可用项如下：

```javascript
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
