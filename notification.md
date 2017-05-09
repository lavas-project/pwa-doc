# 通知

这篇文章主要介绍通知 (`notification`) 。文章将分为两部分：

* 视觉部分

主要涉及通知的各类视觉相关的配置项，从而展现不同样式的通知，例如标题，内容，图标，图片等等。

* 行为部分

主要涉及通知的行为控制，例如点击通知，多个通知的折叠，通知弹出时的声音等等。

## 视觉部分

### 基本使用

在注册了 `service worker` 之后，可以使用 `showNotification` 方法弹出通知。

使用如下代码就可以弹出一个最简单的通知
```
registration.showNotification('Hello World!');
```

详细来看，这个方法共有两个参数，分别为：

* title - __必填__ 字符串类型 表示通知的标题
* options - *选填* 对象类型 集合众多配置项，可用项如下：

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

  // 时间戳
  "timestamp": "<Long>"
}
```

__RELAX!!__ 暂时我们还不需要关注这么多的配置项，下面会逐一讲述这些配置项及其产生的效果。

### 标题和内容

如上所述，标题可以通过 `showNotification` 的第一个参数设置。而通知内容可以通过配置项中的 `body` 进行设置。如下：
```
registration.showNotification('Simple Title', {
    body: 'Simple piece of body text.\nSecond line of body text :)'
});
```

在Google Chrome中，我们会得到这样的通知

![Simple Title on Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-title-body.png)

Firefox则略有差别，在Linux系统的Firefox中，通知会是这样

![Simple Title on Firefox Linux](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/firefox-title-body.png)

而在Windows系统的Firefox中，通知会是这样

![Simple Title on Firefox Windows](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/firefox-title-body-windows.png)

到这里我们可以注意到，通知在不同的浏览器以及不同的操作系统中展现的样式并不相同。事实上：
1. 不同的操作系统有自身的通知样式。Firefox浏览器直接使用系统通知样式
2. 特别的，Google Chrome有自己定制的样式，以保持在各个操作系统的一致性

当标题和内容长度超过一定限制时，通知会自行处理。例如在Google Chrome中会自动截断：

![Long Text on Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-long-title-body.png)

在Firefox中会更友善一些，当鼠标浮在内容上，会展现全部内容：

![Long Text on Firefox](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/firefox-long-title-body.png)
![Long Text on Firefox](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/firefox-long-title-body-expanded.png)

### 图标(icon)

你可能注意到了在Google Chrome下通知的左侧有一大段空白区域。这里其实是用来显示图标的，为此我们需要用到配置项的 `icon` ，如下：
```
registration.showNotification('Icon Notification', {
    icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/icon-512x512.png'
});
```

在Chrome上的通知样式如下：

![Icon on Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-icon.png)

在Firefox样式如下：

![Icon on Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/firefox-icon.png)

在图标尺寸方面并没有一个明确的规定。普通情况下建议使用192px\*192px以上的图片进行设置(64px\*3)。

另外某些浏览器要求静态资源必须通过HTTPS访问，因此在使用第三方图片资源时要格外注意。

### 小图标(Badge)

小图标(Badge)是在手机上展现通知缩略信息时使用的图标。我们可以使用如下代码
```
registration.showNotification('Badge Notification', {
    badge: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/badge-128x128.png'
});
```

目前仅出现在Android系统的Google Chrome上，效果如下：
![Badge on Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/mobile/chrome-badge.png?hl=zh-cn)

在其他情况下，或者当我们没有设置 `Badge` 时，图标会展示为一个浏览器的图标。
![Badge on Others](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/mobile/firefox-badge.png?hl=zh-cn)

小图标的注意点和图标相同，尺寸建议为72px\*72px以上(24px\*3)，同样尽量使用HTTPS资源。

### 图片(image)

和图标(icon)不同，图片(image)在通知的展现尺寸要大不少，可以给用户展现一些预览图片。我们可以使用如下代码
```
registration.showNotification('Image Notification', {
    image: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/unsplash-farzad-nazifi-1600x1100.jpg'
});
```

在PC的Google Chrome上，效果如下：

![Image on PC Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-image.png?hl=zh-cn)

在Android系统的Google Chrome上，效果略有不同。可以看到图片经过了裁剪，如下：

![Image on Android Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/mobile/chrome-image.png?hl=zh-cn)

因为两种平台上图片展现的差别比较大，因此很难给出一个完美适应的图片尺寸。从目前经验来看，使用宽度为1350px(450px\*3)以上的图片会比较合适。但必须指出一点，因为图片相对来说还是一个比较新的配置项，因此以后可能会被修改。

最后，在Firefox中，图片暂时无法显示：

![Image on Firefox](https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/image.png)

### 按钮(Actions)

我们可以通过使用 `actions` 配置项来为通知增加一些按钮，代码如下：
```
registration.showNotification('Actions Notification', {
    actions: [
        {
            action: 'coffee-action',
            title: 'Coffee',
            icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/action-1-128x128.png'
        },
        {
            action: 'doughnut-action',
            title: 'Doughnut',
            icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/action-2-128x128.png'
        },
        {
            action: 'gramophone-action',
            title: 'gramophone',
            icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/action-3-128x128.png'
        },
        {
            action: 'atom-action',
            title: 'Atom',
            icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/action-4-128x128.png'
        }
    ]
});
```

通过以上代码，我们为通知增加了4个按钮。但事实上在不同情况下一条通知可显示的按钮数量是有限的（记录在 `Notification.maxActions` 变量中），超过的部分就将被省略。在本文编写时，Google Chrome允许一条通知中显示2个按钮，因此效果如下：

![Actions on Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-actions.png?hl=zh-cn)

我们注意到 `actions` 配置项是一个数组，每个元素为一个对象，而每个对象都必须拥有3个属性。除了按钮标题和图标地址之外，每个对象还拥有一个唯一的ID，记录于 `action` 属性中。它会在按钮被点击时使用到，这些会在本文第二部分中进行介绍。

此外，在Android6.0 Marshmallow版本中，图标的颜色可能会被设置为匹配系统的颜色，如下所示：

![Actions on Chrome Color](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/mobile/chrome-actions-m.png?hl=zh-cn)

在Android Nougat版本中按钮暂时不会显示。

关于按钮图标方面，我们有一些建议：

1. 所有图标使用一个相同的色系，以保证给用户带来的感官是一致的。
2. 图标尺寸建议使用128px\*128px
3. 某些情况下按钮不会显示，需要做好应对。

在本文编写的时候，只有Google Chrome和Opera for Android支持按钮(Actions)。

### 文字方向

如果我们需要控制文字的方向，可以使用 `dir` 配置项。它的合法值为 `'auto'`, `'ltr'`或者`'rtl'`。其中`'auto'`会自动根据文字内容来选择方向，例如阿拉伯文会自动从右到左显示，默认值为`'auto'`。

![Direction on Chrome](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-rtl.png?hl=zh-cn)

![Direction on Firefox](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/firefox-rtl-expanded.png?hl=zh-cn)

### 震动

我们可以使用 `vibrate` 配置项来设置通知的震动模式。 `vibrate` 以数组的形式进行配置，其中的数字以2个为一组，分别表示震动的毫秒数，和不震动的毫秒数，如此往复。
```
registration.showNotification('Vibrate Notification', {
    vibrate: [500,110,500,110,450,110,200,110,170,40,450,110,200,110,170,40,500]
});
```

注意部分设备可能不支持 `vibrate` 配置项。

### 声音

按照规范，声音可以使用 `sound` 配置项进行配置，用以在通知弹出时播放对应的音频文件。
```
registration.showNotification('Sound Notification', {
    sound: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/notification-sound.mp3'
});
```

然而很遗憾，目前并没有浏览器支持 `sound` 配置项。

### 时间戳

我们可以使用 `timestamp` 配置项来达到定时发送通知的目的。 `timestamp` 配置项的值是数字类型，表示设定时间距离1970年1月1日0点的毫秒数。例子如下：
```
registration.showNotification('Timestamp Notification', {
    body: 'Timestamp is set to "01 Jan 2000 00:00:00".',
    timestamp: Date.parse('01 Jan 2000 00:00:00')
});
```

### 使用通知的一些建议

虽然目前使用通知的站点并不多，但在这些站点中问题也不少。最常见的问题在于通知的目的和内容无法被用户理解，反而造成了不良的用户体验。我们的目标是利用尽可能多的配置项来让用户明确一条通知的内容和目的。下面是一些常见的错误，我们应当尽量避免它们：

1. 不要将站点的名字或者URL放到通知的标题或者内容中，因为浏览器会自动添加这些信息，避免造成重复。
2. 尽量使用简明扼要的通知标题和内容。假设你需要通知用户，有人给他发了一条信息，你应该使用例如“某人给你发送了一条信息”作为标题，并将消息内容放到通知内容中，而不是把标题叫做“新信息”或者把内容叫做“点击查看”。

### 支持性检测

从上述配置项的分析中我们也可以看到，通知在Google Chrome和Firefox中支持的效果并不一致。

举例来说，在使用按钮(Actions)时，我们应该了解浏览器是否支持，因此我们应该使用如下代码：

```
if ('actions' in Notification.prototype) {
    // Action buttons are supported
}
else {
    // Action buttons are NOT supported
}
```

这样就可以在不支持按钮时使用其他方式来进行通知。

## 行为部分

本文的第一部分讲述了许多影响外观的配置项，这一部分将讲述一些影响行为的配置项。

默认情况下，如果只使用上述外观相关的配置项，通知的默认行为如下：

1. 对通知内容进行点击，通知没有变化
2. 新的通知会在老的通知之上显示，浏览器不会对它们进行归类或者折叠等操作
3. 根据设备的不同，可能会在通知弹出时播放声音和震动
4. 在某些设备上，通知经过一段时间会自动消失；在另外一些设备上则不会消失，直到用户点击关闭为止

在这一部分，我们将讨论如何通过配置项来修改上述的默认行为。

### 点击通知

刚才提过，默认情况下用户点击通知不会有任何变化，例如关闭通知。而事实上通知被点击则关闭比较符合我们常见的交互习惯，为了达成这个效果，我们需要在 `service-worker.js` 中进行事件注册，代码如下：

```
self.addEventListener('notificationclick', function (event) {
    const clickedNotification = event.notification;
    clickedNotification.close();

    // 执行某些异步操作，等待它完成
    const promiseChain = doSomething();
    event.waitUntil(promiseChain);
});
```

从示例代码中我们可以看到，通过 `event.notification` 我们可以获取到被点击的通知对象，从而获取它的属性或者调用它的方法。例子中调用了 `close()` 方法就可以关闭通知。

需要注意，如果我们进行了某些异步操作，那么最后的 `event.waitUntil()` 是必不可少的，否则会导致 `service worker` 运行不正常。

### 点击按钮

上一部分提到过通知的按钮配置。这一部分将介绍如何处理这些按钮的点击事件。

假设我们采用和上一部分的相同的按钮配置，配置如下：

```
registration.showNotification('Actions Notification', {
    actions: [
        {
            action: 'coffee-action',
            title: 'Coffee',
            icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/action-1-128x128.png'
        },
        {
            action: 'doughnut-action',
            title: 'Doughnut',
            icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/action-2-128x128.png'
        },
        {
            action: 'gramophone-action',
            title: 'gramophone',
            icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/action-3-128x128.png'
        },
        {
            action: 'atom-action',
            title: 'Atom',
            icon: 'https://gss0.bdstatic.com/9rkZbzqaKgQUohGko9WTAnF6hhy/assets/example/action-4-128x128.png'
        }
    ]
});
```

在用户点击某个按钮时，我们可以监听 `notificationclick` 事件，通过 `event.action` 属性获得按钮的ID（对应 `action` 属性）。例如我们点击咖啡图标，则 `event.action` 的值为 `'coffee-action'`。我们可以参考如下代码：

```
self.addEventListener('notificationclick', function(event) {
    if (!event.action) {
        // 没有点击在按钮上
        console.log('Notification Click.');
        return;
    }

    switch (event.action) {
        case 'coffee-action':
            console.log('User \'s coffee.');
            break;
        case 'doughnut-action':
            console.log('User \'s doughnuts.');
            break;
        case 'gramophone-action':
            console.log('User \'s music.');
            break;
        case 'atom-action':
            console.log('User \'s science.');
            break;
        default:
            console.log(`Unknown action clicked: '${event.action}'`);
            break;
    }
});
```

效果如下：

![Action click](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/action-button-click-logs.png?hl=zh-cn)

### 标签(tag)

默认情况下，我们每调用一次 `showNotification` 方法，就发送一条通知，每条之间都是独立的，互相展开的。因此可以想见，如果连续发送多条通知，用户的手机上会充满来自同一个网站的通知，用户很容易产生负面情绪。为了解决这个问题，我们可以尝试使用 `tag` 来解决这个问题。

`tag` 的取值类型是字符串类型，是一个唯一的ID。两个相同ID的通知会被归类到一起。我们来看一下例子：

```
registration.showNotification('Notification 1 of 3', {
    body: 'With \'tag\' of \'message-group-1\'',
    tag: 'message-group-1'
});
```

这样会发送第一条通知：

![Tag 1](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-first-tag.png?hl=zh-cn)

然后我们再发送一条通知，采用不同的 `tag` ，如下：

```
registration.showNotification('Notification 2 of 3', {
    body: 'With \'tag\' of \'message-group-2\'',
    tag: 'message-group-2'
});
```

于是我们又收到一条通知，加上上一条就会有两条了。

![Tag 2](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-second-tag.png?hl=zh-cn)

在发送第三条通知时，我们采用和第一条相同的 `tag` ，如下：

```
registration.showNotification('Notification 3 of 3', {
    body: 'With \'tag\' of \'message-group-1\'',
    tag: 'message-group-1'
});
```

如同预期，因为和第一条的 `tag` 相同，所以并没有再弹出第三条通知，而是将第一条通知__替换__为了第三条通知。总体来看，我们调用了三次 `showNotification` 方法，但是用户只显示两条，防止用户体验极度恶化。

![Tag 3](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/images/notification-screenshots/desktop/chrome-third-tag.png?hl=zh-cn)

如果两条通知包含相同的 `tag` ，除了替换之外，后面一条通知将不会有声音或者震动提示。如果我们的确需要再次有声音或者震动提示，那么我们需要使用 `renotify` 配置。

### 重新通知(renotify)

`renotify` 配置项是和 `tag` 一同使用的。在使用 `tag` 的同时，设置 `renotify` 为 `true` 可以让浏览器在替换通知时提示声音或者震动。最容易想到的使用场景在聊天应用中，有新消息时的提示。示例代码如下：

```
registration.showNotification('Notification 3 of 3', {
    tag: 'message-group-1',
    renotify: true
});
```

注意：如果你使用了 `renotify` 属性但是没有使用 `tag` 属性，代码会有如下报错：

```
TypeError: Failed to execute 'showNotification' on 'ServiceWorkerRegistration':
Notifications which set the renotify flag must specify a non-empty tag
```

### 静默通知(silent)

如字面意思，`silent` 配置项可以让一条通知静默提示，不播放声音或者震动，适合使用在不需要用户立刻响应的通知的情景下，示例如下：

```
registration.showNotification('Silent Notification', {
    silent: true
});
```

注意：如果你同时使用了 `silent` 和 `renotify` 属性，`silent` 会有较高的优先级，即依然为静默通知。

### 用户交互(requireInteraction)

默认情况下，PC下的Google Chrome在展现通知一部分时间后隐藏通知；而Android系统上的Google Chrome会一直显示通知，直到用户交互，例如点击关闭按钮。

为了显式的让通知一直显示直到用户交互，我们可以设置 `requireInteraction` 属性。

```
registration.showNotification('Require Interaction Notification', {
    body: 'With "requireInteraction: \'true\'".',
    requireInteraction: true
});
```

在使用这个配置项时要格外注意，这可能导致用户体验的下降，因为他必须要求用户操作才能移除通知。
