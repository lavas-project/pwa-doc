# 凭据管理 API

前文提到传统“记住我”的实现方式中，通过 [cookie 存储登录信息的方式](./02-remember-me.md#cookie存储登录信息)存在安全问题，需要制定一系列校验策略和失效规则等等来确保可靠性，对开发者的技术要求较高；而通过表单登录的方式去触发浏览器的账户信息存储这种方法的限制条件较多，且浏览器行为不可控，因此具体操作起来会比较麻烦。

因此浏览器提供了一套凭据管理 API（Crediential Management API），可以把用户的登录信息直接存储于客户端中，这些信息不会写到 cookie 中，因此安全性很高。同时因为账号密码直接写入本地，安全性和可靠性由浏览器保证，因此就不需要额外设计 token 之类的机制进行校验，从而大大降低开发难度。

目前只有在 Android 的 Chrome 浏览器下才能支持该 API。

## 存储用户登录信息

为了能够调用凭据管理 API，需要完成如下工作：

1. 准备一个包含 `autocomplete` 属性的表单；
2. 采用 AJAX 的方式进行表单提交；

### 准备登录表单

登录信息需要存放到一个表单中，如下所示：

```html

<form id="login-form">
    <input name="usr" type="text" autocomplete="username">
    <input name="pwd" type="password" autocomplete="new-password">
</form>

```

该表单需要包含 `autocomplete` 属性，其目的是方便凭据管理 API 从表单中找到用户名和密码，从而才能生成相应的凭证对象。 autocomplete 属于 HTML5 中的新属性，该属性的值在原先 `on | off` 的基础上新增了部分有助于自动填充功能的标记符，如上面表单提到的 `username`、`new-password` 等等。

关于自动填充相关介绍，可以阅读文章： [“Autofill: What web devs should know, but don’t”](https://cloudfour.com/thinks/autofill-what-web-devs-should-know-but-dont/)；关于 autocomplete 的扩展标记符说明，请阅读[ w3c 标准](https://html.spec.whatwg.org/multipage/forms.html#autofill)。


### 采用AJAX进行表单提交

如果在表单中存在 `type="submit"` 的按钮，则需要监听表单提交事件并且阻止默认的提交行为：

```html

<form id="login-form">
    <input name="usr" type="text" autocomplete="username">
    <input name="pwd" type="password" autocomplete="new-password">
    <input type="submit" value="提交">
</form>

```

```javascript

var form = document.getElementById('login-form');

form.addEventListener('submit', function (e) {
    e.preventDefault();
    // 改用 AJAX 进行表单提交
});

```

如果没有在表单里添加提交按钮，也可以在表单外部添加其他行为触发表单提交：

```javascript

button.addEventListener('click', function () {
    // 触发 AJAX 表单提交行为
});

```

改用 AJAX 进行表单提交，可以根据实际项目自行选择 `fetch`、`jQuery`、原生XHR 等方式实现：

```javascript

var formData = new FormData(form);

// fetch
fetch('/login/site/api', {
    method: 'POST',
    body: formData
})
.then(res => {
    if (res.status === 200) {
        return Promise.resolve(res.data);
    }
    else {
        return Promise.reject();
    }
})
.then(data => {
    // 后续操作
})
.catch(err => {
    // 错误操作
})


// jQuery

$.ajax({
    url: '/login/site/api',
    data: {
        usr: formData.get('usr'),
        pwd: formData.get('pwd')
    },
    success: function (res) {
        // 后续操作
    },
    error: function (err) {
        // 错误操作
    }
})

```

登录成功之后，就可以对用户登录信息进行存储啦。

### 存储登录信息

我们需要调用 `navigator.credentials.store()` 这个方法进行登录信息存储。由于仅有部分浏览器支持凭据管理 API，因此在使用前需要进行方法是否存在的判断：

```javascript

if (navigator.credentials) {
    var cred = new PasswordCredetial(form);
    var promise = navigator.credentials.store(cred);
    // 后续操作
}

```

从上面的例子可以看到，首先需要将登录表单转换为 `PasswordCredetial` 对象，再通过调用 `navigator.credentials.store()` 方法进行存储。

navigator.credentials.store 的方法定义如下：

`{Promise} navigator.credentials.store({Credetial} cred)`

存储凭证的方法

**返回**

- `{Promise}` : promise 对象，存储操作结束时，会将存储的 cred 的值给 resolve 出来。

**参数**

- `{Credetial}` cred: 凭证对象

前文提到的 `PasswordCredetial` 实现了 `Credetial` 的接口。

`navigator.credentials.store` 之所以是个异步操作，是因为在调用该方法时，浏览器会弹出提示框询问用户是否对登录信息进行存储，如图所示：

只有当用户选择“保存”时，浏览器才会将登录信息存储起来。

## 获取用户登录信息

通过 `navigator.credentials.get()` 方法，可以获取用户存储的登录信息