## CSP(内容安全策略)

CSP(Content Security Policy) 即内容安全策略，主要目标是减少、并有效报告 XSS 攻击，其实质就是让开发者定制一份白名单，告诉浏览器允许加载、执行的外部资源。即使攻击者能够发现可从中注入脚本的漏洞，由于脚本不在白名单之列，浏览器也不会执行该脚本，从而达到了降低客户端遭受 XSS 攻击风险和影响的目的。

默认配置下，CSP 甚至不允许执行内联代码 (`<script>` 块内容，内联事件，内联样式)，以及禁止执行`eval()`, `setTimeout` 和 `setInterval`。为什么要这么做呢？因为制定来源白名单依旧无法解决 XSS 攻击的最大威胁：内联脚本注入。浏览器无法区分合法内联脚本与恶意注入的脚本，所以通过默认禁止内联脚本来有效解决这个问题。

事实上我们并不推荐使用内联脚本混合的开发方式，使用外部资源，浏览器更容易缓存，对开发者也容易阅读理解，并且有助于编译和压缩。当然，如果不得不需要内联脚本和样式，可以通过设置 `unsafe-inline`，来解除这一限制。

目前浏览器对 CSP 的支持情况可以在 [这里查看](http://caniuse.com/#search=CSP)

### 启用 CSP

有两种方法配置并启用 CSP

1.设置 HTTP 头的 Content-Security-Policy 字段（旧版 X-Content-Security-Policy）

```http
Content-Security-Policy: script-src 'self'; object-src 'none';style-src cdn.example.org third-party.org; child-src https:
```

2.设置页面的 `<meta>` 标签

```html
<meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">
```

上述例子进行了配置

- script: 只信任当前域名
- object-src: 不允许加载任何插件资源（如object, embed, applet 等标签引入的 flash 等插件）
- 样式: 只信任来自 cdn.example.org 和 third-party.org
- 框架内容（如 iframe）: 必须使用 https 协议加载

CSP 提供了很多可配置的选项来针对不同资源的加载进行限制，常见的有，

- script-src：外部脚本
- style-src：样式表
- img-src：图像
- media-src：媒体文件（音频和视频）
- font-src：字体文件
- object-src：插件（比如 Flash）
- child-src：框架
- manifest-src：manifest 文件


如果不为某条配置设置具体的值，则默认情况下，该配置在运行时认为你指定 * 作为有效来源（例如，你可以从任意位置加载字体，没有任何限制）。也可以设置 `default-src` 的值，来代替各个选项的默认值。

每个配置选项的值，可填入以下内容

- 主机名：example.org，https://example.com:443
- 路径名：example.org/resources/js/
- 通配符：\*.example.org，\*://\*.example.com:\*（表示任意协议、任意子域名、任意端口）
- 协议名：https:、data:
- 关键字 'self'：当前域名，需要加引号
- 关键字 'none'：禁止加载任何外部资源，需要加引号

这里不对资源白名单的配置具体介绍了，更多内容可参阅：[https://www.w3.org/TR/CSP/](https://www.w3.org/TR/CSP/)