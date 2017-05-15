## 同源策略

浏览器的同源策略是 Web 安全的基石，它对从一个源加载的文档或脚本如何与来自另一个源的资源进行交互做出了限制。这是一个用于隔离潜在恶意文件的关键的安全机制，每个源均与其余网络保持隔离，从而为开发者提供一个可进行构建和操作的安全沙盒。

如果没有同源策略， Web 世界就变得非常不安全，拿浏览器中的 cookie 来说，当你登录 a 网站，同时打开 b 网站，b 网站能获取你 a 网站的 cookie，盗取你的身份凭证进行非法操作。

同源策略只是一个规范，虽然并没有指定其具体的使用范围和实现方式，但各个浏览器厂商都针对同源策略做了自己的实现。

#### 同源的定义

如果协议，端口和主机对于两个页面是相同的，则两个页面具有相同的源。

例如，相对于
```
http://www.example.com/dir/page.html
```
同源情况如下

| 地址 | 结果 |
| :---:| :----: |
| http://www.example.com/dir2/other.html | 同源 |
| http://v2.www.example.com/dir/other.html | 不同源（主机不同） |
| https://www.example.com/dir/other.html | 不同源（协议不同） |
| http://www.example.com:81/dir/other.html | 不同源（端口不同）|

#### 更改源

页面可以更改自己的源，但会受到一些限制。比如，可以使用 [`document.domain`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/domain)来设置子域的 `domain` 值，允许其安全访问其父域。例如：

可以在 `http://child.company.com/dir/a.html`中执行：

```
document.domain = 'company.com';
```

页面将与 `http://company.com/dir/b.html` 处于相同的域。但是，试图给 `company.com` 设置 `document.domain` 为 `anotherCompany.com` 是不可行的。

*注意* 浏览器的端口号是单独保存的，在给 `document.domain` 赋值时，如果不指明端口号，默认会以 null 值覆盖掉原来的端口号。

#### 一些内嵌资源不受限制

如：

- `<script src="..."></script>` 标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。
- `<link rel="stylesheet" href="...">` 标签嵌入CSS。
- `<img>` 嵌入图片。
- `<video>` 和 `<audio>` 嵌入多媒体资源。
- `<object>`, `<embed>` 和 `<applet>`的插件。
- `@font-face` 引入的字体。一些浏览器允许跨域字体（ cross-origin fonts），一些需要同源字体（same-origin fonts）。
- `<frame>` 和 `<iframe>`载入的任何资源。站点可以使用X-Frame-Options消息头来阻止这种形式的跨域交互。

#### 限制范围

非同源的网站，主要有3种行为受到限制

1. 无法共享 cookie, localStorage, indexDB
2. 无法操作彼此的 DOM 元素
3. 无法发送 Ajax 请求

同源策略做了很严格的限制，但在实际的场景中，又确实有很多地方需要突破同源策略的限制，也就是我们常说的跨域。

规避上述限制，实现跨域通信的解决方案有多种，如 [`CORS`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)，`JSONP`，使用`window.name`，使用[`window.postMessage`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage) 等，这里就不一一展开讲了。

