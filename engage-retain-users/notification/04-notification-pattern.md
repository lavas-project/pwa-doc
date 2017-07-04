# 常用实现

本文将介绍通知在一些常见情况下的实现方式，可能会用到 `service worker` 的其他一些API。

另外如果没有特别说明（如提到“主程序”），本文的所有代码都应编写在 `service-worker.js` 中。

## 通知关闭事件

在行为部分中，我们监听过 `notificationclick` 事件来处理通知点击。

事实上，还有一个 `notificationclose` 事件可以在用户关闭通知时被触发。这里的“关闭”指的是用户点击通知右上角的关闭按钮或者通过滑动通知来移除通知，点击通知并不在此列。通过监听这个事件我们可以对用户关闭通知进行统计，进而统计通知时长，评估通知效果等。

我们可以将如下代码增加到 `service-worker.js` 中。其中 `notificationCloseAnalytics` 方法是用来做一些统计工作，因为可能包含异步因此返回为 `Promise` 对象，也因此必须使用 `waitUntil` 等待其执行完成。

```javascript
self.addEventListener('notificationclose', event => {
    let dismissedNotification = event.notification;
    let promiseChain = notificationCloseAnalytics();
    event.waitUntil(promiseChain);
});
```

## 通知事件的数据传递

在行为部分我们已经介绍了如何处理通知的点击事件：在主程序代码中发送通知 `showNotification`，而在 `service-worker.js` 中监听事件处理。但实际情况下，点击通知会进行一些动态的操作，例如跳往某个URL，那么这个动态信息要如何从主程序传递到 `service-worker.js` 呢？

答案是 `data` 属性。在发送通知时通过 `data` 将需要的动态数据传递过去，在主程序中添加如下代码：

```javascript
registration.showNotification('Notification With Data', {
    body: 'This notification has data attached to it that is printed to the console when it\'s clicked.',
    data: {
        time: (new Date()).toString(),
        message: 'Hello World!'
    }
});
```

在 `service-worker.js` 中，我们通过 `event.notification.data` 来获取这个数据，如下：

```javascript
const notificationData = event.notification.data;
console.log('The data notification had the following parameters:');
Object.keys(notificationData).forEach(key => {
    console.log(`  ${key}: ${notificationData[key]}`);
});
```

这样就可以实现动态的数据传递，并在点击事件中进行不同的处理了。

## 打开页面

上面一部分提过，用户通过点击通知访问某个URL是非常常见的做法。那么如何做到打开页面访问某个URL呢？我们可以通过 `clients.openWindow()` 方法。 如下代码可以允许我们在捕获 `notificationclick` 事件的处理中打开新页面：

```javascript
let examplePage = '/demos/notification-examples/example-page.html';
let promiseChain = clients.openWindow(examplePage);
event.waitUntil(promiseChain);
```

通过 `openWindow` 方法，我们可以打开新窗口，并在新窗口中打开新页面。但如果这个页面已经被打开，更好的做法不是打开新窗口，而是直接激活那个TAB。这个做法将在下一节讨论

## 激活窗口

上一节提过，如果需要打开的页面已经存在，我们应该激活它而不是再打开一次。在我们讨论如何激活之前，一个非常重要的点是：__我们只能激活在自己域的页面__。原因是我们只能知道属于自己域的页面哪些被打开，系统防止开发者掌握用户打开的所有页面，例如那些不属于开发者域的其他页面。

接上一节的例子，我们先判断需要打开的页面是否已经打开了，如下：

```javascript
let urlToOpen = new URL(examplePage, self.location.origin).href;

let promiseChain = clients.matchAll({
    type: 'window',
    includeUncontrolled: true
})
.then(windowClients => {
    let matchingClient = null;

    for (let i = 0, max = windowClients.length; i < max; i++) {
        let windowClient = windowClients[i];
        if (windowClient.url === urlToOpen) {
            matchingClient = windowClient;
            break;
        }
    }

    return matchingClient
        ? matchingClient.focus()
        : clients.openWindow(urlToOpen);
});

event.waitUntil(promiseChain);
```

分析一下代码，它大约执行了这么几个步骤：

1. 把目标页面从字符串转化为URL类型
2. 获取已经打开的所有窗口
3. 逐个寻找匹配
4. 找到了则激活那个窗口；没有找到则打开新窗口
5. 等待这一系列执行。

第一步我们通过 `new URL` 来把字符串转化为URL对象，并且通过 `href` 属性获取地址。和原始的字符串相比，转化后的是一个绝对地址方便比较，而原始的是相对地址。

第二步我们通过如下代码获取所有打开的窗口，注意这里的窗口只包含开发者自己域下的。

```javascript
const promiseChain = clients.matchAll({
    type: 'window',
    includeUncontrolled: true
})
```

其中 `type: 'window'` 表示我们需要寻找打开的窗口和标签，不包括 `web workers` 。`includeUncontrolled` 表示不被 `service worker` 控制的但是属于自己域下的标签和窗口也都纳入搜索范围。一般情况下如果使用 `matchAll` 方法，`includeUncontrolled` 参数都是需要的。

第三步通过 for 循环逐个匹配。当我们找到了结果则调用 `focus()` 方法进行激活；否则则使用上一节提到的 `clients.openWindow()` 使用新窗口打开。

需要注意的是，`matchingClient.focus()` 和 `clients.openWindow(urlToOpen)` 返回的都是 `Promise` 对象，即链式调用。只有这样，才可以让最后一步的 `event.waitUntil()` 起到应有的作用。

## 合并通知

在上一部分我们介绍了多条通知只要含有相同的 `tag` 配置，则在发出时会互相替换而不是再弹出一条。但如果因为一些原因没有使用 `tag` ，或者无法使用相同的 `tag`，我们还有一种方式来做到合并通知。

我们先设想这样一个使用场景：我们开发了一个聊天程序，当X给用户发了一条信息，用户应该收到一条通知，内容是信息的内容，这没有问题。当用户没有关闭这条通知，而X又发了一条信息，那么按照常规的处理，我们应当将刚才那条通知“替换”，内容为 “你收到了来自X的2条信息” 。在不使用 `tag` 的情况下，我们还有下面一种做法。

首先假设每条通知的 `data` 都包含发送者的用户名（如X）。我们要做的第一步是获取用户那边的所有通知，从而找到是否有X发送信息的通知，代码如下：

```javascript
const userName = 'X';
let promiseChain = registration.getNotifications()
    .then(notifications => {
        let currentNotification;

        for(let i = 0, max = notifications.length; i < max; i++) {
            if (notifications[i].data && notifications[i].data.userName === userName) {
                currentNotification = notifications[i];
                break;
            }
        }

        return currentNotification;
    });
```

注意 `registration.getNotifications()` 是一个异步方法，因此我们需要使用 `then` 进行后续处理，筛选出X发来的信息，进行下一步操作。

```javascript
promiseChain.then(currentNotification => {
    let notificationTitle;
    let options = {
        icon: userIcon
    };

    if (currentNotification) {
        // 找到之前X发送信息的通知，整合通知。
        let messageCount = currentNotification.data.newMessageCount + 1;

        options.body = `You have ${messageCount} new messages from ${userName}.`;
        options.data = {
            userName: userName,
            newMessageCount: messageCount
        };
        notificationTitle = `New Messages from ${userName}`;

        // 把之前的信息删除
        currentNotification.close();
    }
    else {
        // 没找到，则常规处理
        options.body = `"${userMessage}"`;
        options.data = {
            userName: userName,
            newMessageCount: 1
        };
        notificationTitle = `New Message from ${userName}`;
    }

    return registration.showNotification(notificationTitle, options);
});
```

通过 `data` 属性和 `getNotification()` 方法，我们做到了整合通知。当X第一次发送信息，通知如下：

![Merge Notification 1](./images/merge-notification-first.png)

X第二次发送信息，在第一条信息还没有被用户关闭之前，效果如下：

![Merge Notification 1](./images/merge-notification-second.png)

无疑这会比一条接着一条将通知显示在用户手机上拥有更好的体验。

## 不要总是发送通知

正常情况当有必要我们应当发送通知给用户告知变化和信息。但有一种情况我们__不__应该发送通知，那就是用户正在浏览我们的站点时。

因此我们在发送通知时应当判断当前的状态并排除这种情况，代码如下：

```javascript
function isClientFocused() {
    return clients.matchAll({
        type: 'window',
        includeUncontrolled: true
    })
    .then(windowClients => {
        let clientIsFocused = false;

        for (let i = 0, max = windowClients.length; i < max; i++) {
            if (windowClients[i].focused) {
                clientIsFocused = true;
                break;
            }
        }

        return clientIsFocused;
    });
}
```

在“激活窗口”一节我们使用过 `clients.matchAll` 方法来遍历打开的（属于自己域的）窗口。这里也类似，通过查看 `focused` 属性来判断窗口是否处于激活状态。

当我们监听到 `push` 事件之后，在发送通知之前，我们可以调用上述方法来判断究竟是否需要发送通知。

```javascript
const promiseChain = isClientFocused()
    .then(clientIsFocused => {
        // 窗口处于激活状态，不需要发送通知
        if (clientIsFocused) {
            console.log('Don\'t need to show a notification.');
            return;
        }

        // 需要发送通知
        return self.registration.showNotification('Had to show a notification.');
    });

event.waitUntil(promiseChain);
```

## 向页面发送信息

上一节提到，当自己站点的窗口处于激活状态时，我们应该避免向用户发送通知。但如果我们的确想向通知一些信息，但又不想使用这么“重”的通知呢？

这种情况我们应该让 `service worker` 有办法通知页面，让页面进行一些提示或者变化（这样避免了震动或者通知栏提示，避免打扰用户），对用户来说会有更好的体验。

假设我们接收到了一次 `push` ，首先我们需要检查我们的窗口是否处于激活状态（使用上述的 `isClientFocused()` 方法），然后使用 `postMessage` 方法来向页面发送数据。

```javascript
const promiseChain = isClientFocused()
    .then(clientIsFocused => {
        // 如果处于激活状态，向页面发送数据
        if (clientIsFocused) {
            windowClients.forEach(windowClient => {
                windowClient.postMessage({
                    message: 'Received a push message.',
                    time: new Date().toString()
                });
            });
        }
        // 否则发送通知
        else {
            return self.registration.showNotification('No focused windows', {
                body: 'Had to show a notification instead of messaging each page.'
            });
        }
    });

event.waitUntil(promiseChain);
```

而在每个页面中，我们可以通过监听 `message` 事件来获取这些数据。在主程序中代码如下：

```javascript
navigator.serviceWorker.addEventListener('message', event => {
    console.log('Received a message from service worker: ', event.data);
});
```

把这里的 `console.log` 替换成修改UI弹出提示或者静默更新信息就可以达成一些用户体验较好的更新。当然页面也可以忽略不相关的信息。
