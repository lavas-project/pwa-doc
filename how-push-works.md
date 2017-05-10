# 消息推送介绍

消息推送有着十分广阔的应用场景：
- 新品上架，推送消息给用户，点击即进入商品详情页面。
- 用户很久没有进入站点了，推送消息告知这段时间站点的更新。

使用推送消息通知，能够让我们的应用像 Native App 一样，提升用户体验。

但是目前[整体支持度并不高](http://caniuse.com/#search=push)，在手机端更是只有安卓 Chrome57 支持。

如同淘宝卖家将商品送到用户家中需要依赖快递公司，
服务器向浏览器推送消息，也需要推送服务(Push Service)的帮助。
下面让我们看看服务器，浏览器和推送服务这三者在推送过程中扮演的角色。

## 获取授权和订阅信息

首先浏览器需要得到用户授权，同意使用消息推送服务。

在具体实现中，由于[通知API](https://developer.mozilla.org/en-US/docs/Web/API/Notification/requestPermission)还不稳定，需要兼容新旧版本的返回值：
``` javascript
function askPermission() {
    return new Promise(function (resolve, reject) {
        const permissionResult = Notification.requestPermission(function (result) {
            // 旧版本
            resolve(result);
        });
        if (permissionResult) {
            // 新版本
            permissionResult.then(resolve, reject);
        }
    })
    .then(function (permissionResult) {
        if (permissionResult !== 'granted') {
            // 用户未授权
        }
    });
}
```

得到用户同意后，浏览器会发送请求到推送服务，生成一个推送订阅对象(PushSubscription)，这个对象包含了标识用户设备的信息。将该信息发送到服务端存储起来，就如同卖家获取了“买家的地址簿”。

具体实现步骤如下：

1. 注册 Service Worker
2. 使用 pushManager 添加订阅，浏览器向推送服务发送请求，其中传递参数对象包含两个属性：
    * `userVisibleOnly`，不允许静默的推送，所有推送都对用户可见
    * `applicationServerKey`，服务器生成的公钥，又被称作[VAPID](https://tools.ietf.org/html/draft-thomson-webpush-vapid)，推送服务将这个公钥和唯一的 endpoint 关联起来，用于后续的安全验证
3. 浏览器将返回的 endpoint 加入推送订阅对象，向服务器发送信息，服务端进行存储

对应的代码实现：
``` javascript
function subscribe(serviceWorkerReg) {
    serviceWorkerReg.pushManager.subscribe({ // 2. 订阅
        userVisibleOnly: true,
        applicationServerKey: urlBase64ToUint8Array('BEl62iUYg...')
    }).then((subscription) => sendEndpointInSubscription(subscription)); // 3. 发送到服务器
}
navigator.serviceWorker.register('./service-worker.js') // 1. 注册  
    .then(function(reg) {subscribe(reg)});
```

### 推送订阅对象

一个完整的推送订阅对象结构如下，其中`endpoint`就是唯一标识用户设备的地址，`keys`包含了用于安全验证信息，在后续向推送服务发送消息时会使用到。
``` json
{
    "endpoint": "https://random-push-service.com/idxxx",
    "keys": {
        "p256dh" : "BNcRd...",
        "auth"   : "tBHI..."
    }
}
```

### 取消订阅

在某些情况下，例如服务端请求推送服务，返回了推送服务失效错误码，此时需要[取消订阅](https://developer.mozilla.org/en-US/docs/Web/API/PushSubscription/unsubscribe)，代码实现如下：

```javascript
navigator.serviceWorker.ready.then(function(reg) {
    reg.pushManager.getSubscription().then(function(subscription) {
        subscription.unsubscribe().then(function(successful) {
        }).catch(function(e) {
        });
    });
});
```

## 发送消息

现在我们有了买家的地址，需要委托快递公司发货。

所有推送服务都遵循统一的调用标准，好比快递公司有着全国统一的上门服务电话，这就是 [Web Push Protocol](https://tools.ietf.org/html/draft-ietf-webpush-protocol) 。

推送服务接到了服务器的调用请求，向设备推送消息，如果处于离线状态，消息将进入待发送队列，过期后队列清空，消息被丢弃。

下面介绍保证消息安全的原理，以及在实际使用中服务端的具体代码实现。

### 消息推送安全性

商品需要包裹封装，就连快递公司也无权拆开。

消息推送的安全体现在两方面：
- 推送服务确保调用来自可靠的服务端
- 推送消息内容只有浏览器能够解密，就算是推送服务也不行

#### 保证服务端可靠性

服务器在调用推送服务时，需要额外发送请求头，例如*Authorization*和*Crypto-Key*，首先介绍 Authorization。

> [JWT(JSON Web Token)](https://jwt.io/)提供了一种消息接收者验证发送者的方法。

Authorization 就包含了 JWT 格式的字符串：
`Authorization: 'WebPush <JWT Info>.<JWT Data>.<Signature>'`

Authorization 的内容由三部分组成，使用`.`连接，前两部分是使用base64编码后的JSON字符串：
- JWT Info，指明了签名使用的加密算法
    ``` json
    {
        "typ": "JWT",
        "alg": "ES256"  
    }
    ```
- JWT Data，包含发送者的信息，推送服务的源地址，失效时间，和发送者的联系方式
    ``` json
    {  
        "aud": "https://some-push-service.org",
        "exp": "1469618703",
        "sub": "mailto:example@web-push-book.org"  
    }
    ```
- 签名，连接前两部分，服务端使用私钥加密。还记得之前添加订阅的时候，使用到的服务端生成的公钥吗，此处使用的正是与之配对的私钥

另外，请求头中还需要将公钥带给推送服务：
`Crypto-Key: p256ecdsa=<URL Safe Base64 Public Application Server Key>`

这样，当推送服务收到服务端的调用请求时，使用公钥解密 Authorization 签名部分，如果匹配前两部分，说明请求来自可靠的服务端。

#### 消息内容加密

由于推送 API 的统一性，用户可能误发消息到不信任的推送服务，对消息进行加密可以确保只有浏览器端才能解密读取，防止将用户信息泄露给不合法的推送服务。

还记得最初用户订阅成功后，浏览器生成的推送订阅对象吗？里面包含了`endpoint`，而加密过程会使用其中的`keys`对象。

[复杂的加密过程](https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/web-push-protocol)在这里就不展开介绍了。总之，只有订阅的浏览器能使用生成的私钥解密推送的消息。

### 推送服务的响应

现在，服务端可以向 endpoint 发送包含以上请求头的请求了，推送服务响应`201`表示接受调用。
其余响应状态码如下：
- 429 Too many requests
- 400 Invalid request
- 404 Not Found 订阅过期，需要在服务端删除保存的推送订阅对象
- 410 Gone 订阅失效，需要在服务端删除保存的推送订阅对象，并调用推送订阅对象的`unsubscribe()`方法
- 413 Payload size too large

### 使用web-push发送消息

服务端请求推送服务，需要涉及加密，设置请求头等复杂操作，使用[web-push](https://github.com/web-push-libs/web-push)可以帮助我们解决大部分问题。

首先，使用 web-push 生成VAPID，将其中的`publicKey`传递到订阅页面中，供 pushManager 订阅时传入。

然后，之前订阅时浏览器已经将推送订阅对象发送到了服务端，服务端存储到数据库中。

最后，调用`sendNotification`向推送服务发起调用请求，如果返回错误状态码，从数据库中删除保存的推送订阅对象。

实现代码如下：
``` javascript
const webpush = require('web-push');
const vapidKeys = webpush.generateVAPIDKeys();
webpush.setVapidDetails(
    'mailto:sender@example.com',
    vapidKeys.publicKey,
    vapidKeys.privateKey
);
// 从数据库中拿出pushSubscription
webpush.sendNotification(pushSubscription, '推送消息内容')
    .catch((err) => {
        if (err.statusCode === 410) { // 从数据库中删除推送订阅对象
        }
    });
```

## 显示通知

想象一个卖家常驻小区的派送员，收到快递公司的商品，上门按下买家的门铃。
Service Worker 就是扮演这样的角色，它监听 push 事件，显示通知。

使用消息中携带的数据，展示通知，此处省略了通知对象(Notification)的配置信息，示例代码如下：
``` javascript
self.addEventListener('push', event => {
    let promiseChain;
    if (event.data) {
        promiseChain = Promise.resolve(event.data.json());
    }
    promiseChain = promiseChain.then(data => {
        return self.registration.showNotification(data.title, {});
    });
    event.waitUntil(promiseChain);
});
```

至此，整个推送流程就结束了。可以发现接入推送服务，只需要在服务器和浏览器端做简单的修改即可。
