# 记住我（Remember Me）功能实现

不少网站在登录界面会提供“**记住我**”这样一个勾选项，方便用户省去输入账号密码，以实现网站的快速登录。

传统的“记住我”功能主要有两种实现方式：

- [cookie存储登录信息](#cookie存储登录信息)
- [浏览器自动填充登录信息](#浏览器自动填充登录信息)

## cookie存储登录信息

直接利用 cookie 存储用户的用户名和密码是非常不安全的，攻击者可以通过各种漏洞访问到 cookie 从而导致用户密码泄露（[常见的安全漏洞](../web-security/04-typical-web-attack.md)）。

常用做法是，当用户登录成功时，服务端为用户生成一个 token，并且写入 cookie，然后用这个 token 作为用户的标识符，供用户直接使用 Token 进行登录。Token 需要制定一系列校验策略和失效规则来确保 Token 的可靠性，因此对开发者的技术要求较高。

## 浏览器自动填充登录信息

浏览器会对网页的文本框和表单信息进行自动记录，特别地，当表单中存在输入类型为 `password` 的输入框时，会触发浏览器的记住账号密码提示：

选择“记住”密码后，返回登录页面，就可以看到账号和密码已自动填充：

如果页面记住了多个账号密码的话，可以在点击用户名输入框时触发下拉框进行账号切换：

这类登录信息的自动记录属于浏览器行为，并没有存到 cookie 中，因此比 cookie 存储登录信息的方式要相对安全。

### 触发浏览器记录登录信息的条件

登录页面需要存在一个包含 `type="text"` 和 `type="password"` 的表单：

```html
<form>
    <p>用户名：<input type="text" name="username"></p>
    <p>密码：<input type="password" name="password"></p>
</form>
```

事实上，浏览器会获取 `type="password"` 的输入框，以及这个输入框之上最近的一个 `type="text"` 的输入框内容，分别作为登录信息中的密码和账号进行存储，比如下面的表单结构：

```html
<form>
    <p>用户名1：<input type="text" name="username1"></p>
    <p>用户名2：<input type="text" name="username2"></p>
    <p>密码：<input type="password" name="password"></p>
    <p><input type="submit" value="提交">
</form>
```

浏览器记住密码之后，只会自动填充 `username2` 和 `password` 这两个字段。因此在开发的时候，需要注意这一点。

### 自动填充功能拓展

在表单字段中添加 `autocomplete` 属性，能够让登录信息的自动填充过程变得更友好些。假设表单结构如下所示：

```html
<form>
    <p>用户名：<input type="text" name="usr" autocomplete="username"></p>
    <p>密码：<input type="password" name="pwd" autocomplete="current-password"></p>
    <p><input type="submit" value="提交">
</form>
```

在触发自动填充时，会增加如下提示：

`autocomplete` 属于 HTML5 中的新属性，该属性原先支持的值为 `on | off`，表示对应的输入框自动填充功能的打开或者关闭，默认值为 `on`。目前 `autocomplete` 的值新增了部分有助于自动填充功能的标记符，如上面表单提到的 `username`、`current-password` 等等。

关于自动填充相关介绍，可以阅读文章： [“Autofill: What web devs should know, but don’t”](https://cloudfour.com/thinks/autofill-what-web-devs-should-know-but-dont/)；关于 autocomplete 的扩展标记符说明，请阅读[ w3c 标准](https://html.spec.whatwg.org/multipage/forms.html#autofill)。