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
    <input name="usr" autocomplete="username">
    <input name="pwd" autocomplete="new-password">
</form>

```

该表单需要包含 `autocomplete` 属性，其目的是方便凭据管理 API 从表单中找到用户名和密码，从而才能生成相应的凭证对象。 autocomplete 属于 HTML5 中的新属性，该属性的值在原先 `on | off` 的基础上新增了部分有助于自动填充功能的标记符，如上面表单提到的 `username`、`new-password` 等等。

关于自动填充相关介绍，可以阅读文章： [“Autofill: What web devs should know, but don’t”](https://cloudfour.com/thinks/autofill-what-web-devs-should-know-but-dont/)；关于 autocomplete 的扩展标记符说明，请阅读[ w3c 标准](https://html.spec.whatwg.org/multipage/forms.html#autofill)

## 获取用户登录信息