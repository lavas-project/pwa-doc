# 第三方登录管理

对于用户而言，注册账号密码是一件非常麻烦的事情，不但注册过程繁琐且花时间，同时也提高了用户的账号维护成本。因此如果网站能够提供第三方登录，让用户能够直接复用一些现有且常用的网站账号，将能够大大提高用户体验。

## 接入第三方登录 API

一些大型的站点平台都会开放相应的第三方登录接口和说明文档，如国内的有：

- 百度账号接入指南：[http://developer.baidu.com](http://developer.baidu.com/wiki/index.php?title=%E5%B8%AE%E5%8A%A9%E6%96%87%E6%A1%A3%E9%A6%96%E9%A1%B5/%E7%99%BE%E5%BA%A6%E5%B8%90%E5%8F%B7%E8%BF%9E%E6%8E%A5/%E7%99%BE%E5%BA%A6%E5%B8%90%E5%8F%B7%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)
- 新浪微博接入指南：[http://open.weibo.com](http://open.weibo.com/wiki/%E7%BD%91%E7%AB%99%E6%8E%A5%E5%85%A5%E4%BB%8B%E7%BB%8D)
- 微信账号接入指南：[https://open.weixin.qq.com](https://open.weixin.qq.com/cgi-bin/frame?t=home/web_tmpl&lang=zh_CN)
- QQ账号接入指南：[https://connect.qq.com/intro/login](https://connect.qq.com/intro/login)

国外的有：

- [google](https://developers.google.cn/identity/sign-in/web/)
- [facebook](https://developers.facebook.com/docs/facebook-login)
- [twitter](https://dev.twitter.com/web/sign-in/implementing)
- [github](https://developer.github.com/v3/oauth/)

## 保存第三方登录凭证

同样需要调用方法 `navigator.credentials.store()` 进行第三方登录凭证存储，只不过存入的凭证类型为 `FederatedCredential`。`FederatedCredential` 同样实现了 `Credential` 接口，同时还新增了 `provider` 字段作为第三方登录提供方的标识符。因此 `FederatedCredential` 初始化参数对象应包含以下信息：

- `id`: **必须** 账号
- `provider`: **必须** 第三方登录提供方
- `name`: **非必需** 用户名
- `iconUrl`: **非必需** 用户头像

例如：

```javascript

/* global THIRD_PARTY_PROVIDER */

thirdPartyLogin()
    .then(function (profile) {
        if (navigator.credentials) {
            let cred = new FederatedCredential({
                id: profile.email,
                provider: THIRD_PARTY_PROVIDER,
                name: profile.name,
                iconUrl: profile.iconUrl
            });

            return navigator.credentials.store(cred);
        }

        return profile;
    })
    .then(function (profile) {
        // 后续操作
    })
    .catch(function (err) {
        // 错误处理
    });

```

## 读取第三方登录凭证

需要调用方法 `navigator.credentials.get()` 方法进行第三方登录凭证的读取。

在前文[获取用户登录信息](#获取用户登录信息)章节中提到，`navigator.credentials.get(options)` 方法传入参数包含一个字段 `federated`，可以通过这个字段去读取第三方登录的凭证信息。

- `options.federated`: 第三方登录
    `{Object}`
    - `providers`:
        `{Array}` 联合登录账号供应者 id 组成的数组

例如：

```javascript

navigator.credentials.get({
    federated: {
        providers: ['baidu.com', 'weibo.com', 'github.com']
    }
})

```

其中 `providers` 中填入的账号供应者信息只是作为第三方登录的标识符，您也可以写成诸如：

```javascript

providers: ['百度', '微博', 'Github']

```

之类的形式，前提是，这些标识符需要与 `FederatedCredential` 第三方登录凭证信息的 `provider` 象一致即可。

这样在弹出的账号选择列表中，就可以看到如下所示的账号信息：

这些就是对应的第三方登录凭证信息。

在获取到第三方登录凭证信息之后，需要通过 `type` 和 `provider` 字段进行凭证信息分类处理，如：

```javascript

navigator.credentials.get({
    password: true,
    federated: {
        providers: ['baidu.com', 'weibo.com']
    }
})
.then(function (cred) {
    if (cred) {
        switch (cred.type) {
            case 'password':
            // PasswordCredential 凭证处理
            case 'federated':
                // FederatedCredential 凭证处理
                switch (cred.provider) {
                    case 'baidu.com':
                        // 调起百度第三方登录
                    case 'weibo.com':
                        // 调起微博第三方登录
                }
        }
    }
});

```
