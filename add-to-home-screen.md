# 将应用添加至主屏幕

## 概述

允许将站点添加至主屏幕，是 PWA 提供的一项重要功能。虽然目前部分浏览器已经支持向主屏幕添加网页快捷方式以方便用户快速打开站点，但是添加到主屏幕的 PWA 不仅仅是一个网页快捷方式，它将提供更多的功能，让 PWA 具有更加原生的体验。

本文将以实现功能为粒度，去说明以下问题：

1. 如何让 PWA 站点支持添加至主屏幕的功能；
2. 添加到主屏幕的 PWA 具有哪些原生体验；
3. 如何引导用户将 PWA 添加至主屏幕。

## 标准现状

PWA 添加至桌面的功能实现依赖于 manifest.json。当前 manifest.json 的标准仍属于草案阶段，Chrome 和 Firefox 已经实现了这个功能，微软正努力在 Edge 浏览器上实现，Apple 目前仍在考虑中。具体请查阅 [`caniuse.com`](http://caniuse.com/#search=manifest) 来查看主流浏览器的支持情况。同时需要注意的是，manifest.json 目前仍属于实验性技术，所以其部分属性和功能在未来有可能会发生改变。

## 使用说明

为了实现 PWA 应用添加至桌面的功能，除了要求站点支持 HTTPS 之外，还需要准备 manifest.json 文件去配置应用的图标、名称等信息。举个例子，一个基本的 manifest.json 应包含如下信息：

```json

{
    "short_name": "短名称",
    "name": "这是一个完整名称",
    "icon": [
        {
            "src": "icon.png",
            "type": "image/png",
            "sizes": "48x48"
        }
    ],
    "start_url": "index.html"
}

```

使用 `link` 标签将 manifest.json 部署到 PWA 站点 HTML 页面的头部，如下所示：

```html
<link rel="manifest" href="path-to-manifest/manifest.json">
```

通过对 manifest.json 进行相应配置，可以实现以下功能：

- [自定义名称](#自定义名称)
- [自定义图标](#自定义图标)
- [设置启动网址](#设置启动网址)
- [设置作用域](#设置作用域)
- [添加启动画面](#添加启动画面)
- [设置显示类型](#设置显示类型)
- [指定显示方向](#指定页面显示方向)
- [设置主题色](#设置主题颜色)
- [引导用户添加应用](#引导用户添加应用至主屏幕)
- [引导用户安装原生应用](#引导用户安装原生应用)

### 自定义名称

PWA在通过应用安装横幅引导用户安装 app，以及被添加到主屏幕时，需要显示应用名称以便用户将其与其他网站区分开来。对应的属性为：

- `name`:
    `{string}` 应用名称，用于安装横幅显示
- `short_name`:
    `{string}` 应用短名称，用于主屏幕显示

之所以用两个字段做区分，是由于显示在主屏幕的应用名称长度有限，超长部分会被截断并显示省略号，需要设置较短的应用名称优化显示；而安装横幅没有长度限制，可以将完整的应用名称显示出来。

**注** 目前如果修改了 manifest.json 的应用名称，已添加到主屏幕的名称并不会改变，只有当用户重新添加到桌面时，更改后的名称才会显示出来。但是在未来版本的 Chrome 浏览器将支持名称自动更新，详情请戳：[`Updating your app's icon and name`](https://developers.google.com/web/updates/2017/02/improved-add-to-home-screen#updating_your_apps_icon_and_name)。

#### 示例

```json

{
    "name": "这是一个完整名称",
    "short_name": "短名称",
    // ...
}

```

通过上述配置信息，得到的 PWA 的安装横幅和主屏应用显示将如下图所示：

// @TODO 安装横幅截图和主屏截图

### 自定义图标

当用户将 PWA 添加至主屏幕时，会如同原生应用一样显示应用名和图标。我们可以通过 `icons` 属性定义一组不同大小的图标供浏览器进行选择。

- `icons`:
    `{Array.<ImageObject>}` 应用图标列表

其中 ImageObject 的属性值包括：

- `src`:
    `{string}` 图标 url
- `type`
    `{string=}` 图标的 mime 类型，非必填项，该字段可让浏览器快速忽略掉不支持的图标类型。
- `sizes`
    `{string}` 图标尺寸，格式为`widthxheight`，宽高数值以 css 的 `px` 为单位。如果需要填写多个尺寸，则使用空格进行间隔，如"48x48 96x96 128x128"。

更详尽的ImageObject介绍，请参阅[`Image object and its members`](https://www.w3.org/TR/appmanifest/#dfn-image-object)。

当PWA添加到主屏幕时，浏览器会根据有效图标的 sizes 字段进行选择。首先寻找与显示密度相匹配并且尺寸调整到 48dp 屏幕密度的图标。如果未找到任何图标，则会查找与设备特性匹配度最高的图标。

**需要注意的是**

1. 为了能够自动显示安装横幅，必须要配置一个 `144x144` 的图标，且图标的 mime 类型为 `image/png`。详情请参阅[应用安装横幅](#引导用户添加应用至主屏幕)章节；
2. 在启动应用时，启动画面图像会从图标列表中提取最接近 `128dp` 的图标进行显示。详情请参阅[添加应用启动画面](#添加启动画面)章节。
3. 目前如果修改了 manifest.json 的图标列表，已添加到主屏幕的名称并不会改变，只有当用户重新添加到桌面时，更改后的图标才会显示出来。但是在未来版本的 Chrome 浏览器将支持图标自动更新，详情请戳：[`Updating your app's icon and name`](https://developers.google.com/web/updates/2017/02/improved-add-to-home-screen#updating_your_apps_icon_and_name)。


#### 示例

例如图标列表的配置为：

```json

{
    "icons": [
        {
            "src": "path-to-images/icon-small.png",
            "type": "image/png",
            "sizes": "48x48"
        },
        {
            "src": "path-to-images/icon-median.png",
            "type": "image/png",
            "sizes": "96x96"
        },
        {
            "src": "path-to-images/icon-large.png",
            "type": "image/png",
            "sizes": "128x128"
        }
    ]
    // ...
}

```

通过该配置信息，得到的 PWA 的安装横幅和主屏应用显示将如下图所示：

// @TODO 不同分辨率机型下的显示 不同尺寸的图标

### 设置启动网址

当PWA添加到主屏幕后，需要通过 `start_url` 去指定应用打开的网址。

- `start_url`:
    `{string=}` 应用启动地址

如果该属性为空，则默认使用当前页面，这可能不是用户想要的内容，因此建议配置 start_url；如果 start_url 配置的相对地址，则基地址与 manifest.json 相同。

#### 示例

假设 PWA 文件结构为：

```

https://test.baidu.com
    |
    |----src
    |
    |----manifest.json
    |
    |----entry
            |----index.html
            |
            |----detail.html


```

manifest.json 对应的 start_url 如果采用绝对地址的形式，其配置如下：

```json

{
    "start_url": "/entry/index.html",
    // ...
}

```

等价于如下相对地址：

```json

{
    "start_url": "./entry/index.html",
    // ...
}

```

假设用户在 detail.html 页面将应用添加至首屏，如果 start_url 为空，从首屏打开应用时，打开的页面将是 `/entry/detail.html`

### 设置作用域

对于一些大型网站而言，有时仅仅对站点的某些模块进行 PWA 改造，其余部分仍为普通的网页。因此需要通过 `scope` 属性去限定作用域，超出范围的部分会以浏览器的方式显示。

- `scope`:
    `{string}` 作用域

**注** scope 属性在浏览器中的实现仍然在细化和改进。

scope 应遵循如下[`规则`](https://developers.google.com/web/updates/2017/02/improved-add-to-home-screen#effectively_managing_your_apps_scope)：

- 如果没有在 manifest 中设置 scope，则默认的作用域为 manifest.json 所在文件夹；
- scope 可以设置为`../`或者更高层级的路径来扩大PWA的作用域；
- `start_url`必须在作用域范围内；
- 如果`start_url`为相对地址，其根路径受 scope 所影响；
- 如果`start_url`为绝对地址（以`/`开头），则该地址将永远以`/`作为根地址；

#### 示例

| manifest url              | start_url                 | scope配置  | 计算好的scope | 计算好的start_url                 | 是否有效                         |
| ------------------------- | ------------------------- | ---------- | ------------- | --------------------------------- | -------------------------------- |
| /tech-today/manifest.json | ./index.html              | undefined  | /tech-today/  | /tech-today/index.html            | 有效                             |
| /tech-today/manifest.json | ./index.html              | ../        | /             | /index.html                       | 有效 - 但作用域泄露到了更高层级  |
| /tech-today/manifest.json | /                         | /          | /             | /index.html                       | 有效 - 但作用域泄露到了更高层级  |
| /tech-today/manifest.json | /                         | undefined  | /tech-today/  | /                                 | 无效 - start url不在作用域范围内 |
| /tech-today/manifest.json | ./tech-today/index.html   | undefined  | /tech-today/  | /tech-today/tech-today/index.html | 有效 - 但start url明显不符合预期 |
| /manifest.json            | ./tech-today/index.html   | undefined  | /             | /tech-today/index.html            | 有效 - 广作用域                  |
| /manifest.json            | ./tech-today/index.html   | tech-today | /tech-today/  | /tech-today/index.html            | 有效 - 窄作用域                  |

### 添加启动画面

当 PWA 从主屏幕点击打开时，幕后执行了若干操作：

1. 启动浏览器
2. 启动显示页面的渲染器
3. 加载资源

在这个过程中，由于页面未加载完毕，因此屏幕将显示空白并且看似停滞。如果是从网络加载加载的页面资源，白屏过程将会变得更加明显。因此 PWA 提供了启动画面功能，用标题、颜色和图像组成的画面来替代白屏，提升用户体验。

#### 设置图像和标题

浏览器会从 [`icons`](#自定义图标) 中选择最接近 `128dp` 的图片作为启动画面图像。标题则直接取自 [`name`](#自定义名称)。

#### 设置背景颜色

通过设置 `background_color` 属性可以指定启动画面的背景颜色。

- `background_color`:
    `{Color}` css色值

background_color 的值可以通过如下几种形式定义：

```

// 完整色值
"background_color": "#0000ff"
// 缩写
"background_color": "#00f"
// 预设色值
"background_color": "blue"
// rgb
"background_color": "rgb(0, 0, 255)"
// transparent 背景色显示为黑色
"background_color": "transparent"

```

其余诸如`#0000ff90`、`rgba`、`hsl`、`hsla`等颜色定义方式背景色均显示为白色。

#### 设置显示类型

仅当显示类型 `display` 设置为 `standalone` 或 `fullscreen` 时，PWA 启动的时候才会显示启动画面。关于 `display` 的详细介绍请阅读 [`设置显示类型`](#设置显示类型) 章节。

```json

"display": "standalone"

```

**注意**

1. 背景色的色值建议为加载页的背景色，采用相同的颜色可以实现从启动画面到首页的平稳过渡；
2. background_color 应该`只应用于`改善页面资源正在加载时的用户体验。当网页样式表加载完成时，应使用样式表中定义的背景色。

#### 示例

对 manifest.json 做如下配置：

```json

{
    "name": "这是一个完整名称",
    "icons": [
        {
            "url": "path-to-images/logo-144x144.png",
            "type": "image/png",
            "sizes": "144x144"
        }
    ],
    "background_color": "#00f"
}

```

则应用启动画面如图所示：

// @TODO 启动图片

### 设置显示类型

可以通过设置 `display` 属性去指定 PWA 从主屏幕点击启动后的显示类型。

- `display`:
    `{string}` 显示类型

显示类型的值包括以下四种：

| 显示类型 | 描述 | 降级显示类型 |
| -------- | ---- | ------------ |
| fullscreen | 应用的显示界面将占满整个屏幕 | standalone |
| standalone | 浏览器相关UI（如导航栏、工具栏等）将会被隐藏 | minimal-ui |
| minimal-ui | 显示形式与standalone类似，浏览器相关UI会最小化为一个按钮，不同浏览器在实现上略有不同 | browser |
| browser | 浏览器模式，与普通网页在浏览器中打开的显示一致 | (None) |

**注意** 可以通过 [`display-mode`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/%40media/display-mode) 这个媒体查询条件去指定在不同的显示类型下不同的显示样式，如：

```css

@media all and (display-mode: fullscreen) {
    body {
        margin: 0;
    }
}

@media all and (display-mode: standalone) {
    body {
        margin: 1px;
    }
}

@media all and (display-mode: minimal-ui) {
    body {
        margin: 2px;
    }
}

@media all and (display-mode: browser) {
    body {
        margin: 3px;
    }
}

```

#### 示例

对 PWA 设置以上四种显示类型，对应的应用截图如下所示：

// @TODO 四种显示类型截图

### 指定页面显示方向

PWA允许应用通过设置 `orientation` 属性的值，强制指定显示方向。

- `orientation`:
    `string` 应用显示方向

orientation属性的值有以下几种：

- `landscape-primary`
- `landscape-secondary`
- `landscape`
- `portrait-primary`
- `portrait-secondary`
- `portrait`
- `natural`
- `any`

由于不同的设备的宽高比不同，因此对于`横屏`、`竖屏`的不能简单地通过屏幕旋转角去定义。如对于手机来说，`90°` 和 `270°` 为横屏，而在某些平板电脑中，`0°` 和 `180°` 才是横屏。因此需要通过应用视窗去定义。

- 当视窗宽度大于高度时，当前应用处于`横屏`状态。横屏分为两种角度，两者相位差为 `180°`，分别为 `landscape-primary` 和 `landscape-secondary`。
- 当视窗宽度小于等于高度时，当前应用处于`竖屏`状态。同样，竖屏分为两种，两者相位差为 `180°`，分别为 `portrait-primary` 和 `portrait-secondary`。

有了 `landscape-primary`、`landscape-secondary`、`portrait-primary`、`portrait-secondary` 的定义，我们就可以用它们来定义其他的属性值了。

- `landscape`:
    根据不同平台的规则，该值可等效于 `landscape-primary` 或 `landscape-secondary`，或者根据当前屏幕旋转角不同，去自由切换 `landscape-primary` 或 `landscape-secondary`；
- `portrait`:
    根据不同平台的规则，该值可等效于 `portrait-primary` 或 `portrait-secondary`，或者根据当前屏幕旋转角不同，去自由切换 `portrait-primary` 或 `portrait-secondary`；
- `natural`:
    根据不同平台的规则，该值可等效于 `portrait-primary` 或 `landscape-primary`，即当前屏幕旋转角为`0°`时所对应的显示方向；
- `any`:
    根据屏幕旋转角自由切换 `landscape-primary`、`landscape-secondary`、`portrait-primary`、`portrait-secondary`。

#### 示例

以三星 Note3为例，当设置 orientation 分别为 `landscape` 和 `portrait` 时，应用显示方向如下图所示：

// @TODO

### 设置主题颜色

通过设置 `theme_color` 属性可以指定PWA的主题颜色。可以通过该属性来控制浏览器 UI 的颜色。比如 PWA 启动画面上状态栏、内容页中状态栏、地址栏的颜色，会被 `theme_color` 所影响。

- `theme_color`:
    `{Color}` css色值

对于当前版本的 Chrome 浏览器，在 `browser` 显示类型下，内容页的状态栏、地址栏并不会显示成 `theme_color` 所指定的颜色，如图所示：

在指定了 `theme_color` 的值之后，地址栏依然呈白色。针对这种情况，可以在页面 HTML 里设置 `name` 为 `theme-color` 的 `meta` 标签，例如：

```html

<meta name="theme-color" content="green">

```

需要注意的是，这个标签的色值会覆盖 manifest.json 里设置的 `theme_color`，如果两个色值不一样的话，会导致应用启动画面和内容页的主题色不一致，因此建议将 `theme_color` 的色值设置得与 `theme-color<meta>`的色值相等。


#### 示例

当manifest设置如下：

```json

{
    "theme_color": "blue",
    "display": "browser"
}

```

可以看到对应的状态栏和地址栏等浏览器 UI 均显示蓝色。

如果此时在网页的 HTML 头部加入如下 meta 标签：

```html

<meta name="theme-color" content="green">

```

则对应页面的浏览器 UI 将显示为绿色。

// @TODO


### 引导用户添加应用至主屏幕

引导用户将 PWA 添加至主屏幕，不但能够为用户提供更好的应用体验，更为重要的是，还能够使得用户方便快捷地打开网站，既增加了站点访问量，同时也提高了用户粘性。

如下图所示，打开浏览器菜单，会看到`添加到主屏幕`的功能，用户可以点击该选项手动将 PWA 站点添加至主屏幕。

// @TODO

很明显对于大部分用户来说，都不会主动去完成上述操作，因此需要适时地弹出`应用安装横幅`去引导用户进行添加操作。PWA 提供的应用安装横幅如下图所示：

// @TODO

用户只需点击横幅上的`添加应用`按钮，即可将 PWA 添加到他们的主屏幕。相比起用户主动操作，弹出应用安装横幅的形式更直观，操作更简便，用户的应用添加率也会更高。

#### 显示应用安装横幅的条件

浏览器在 PWA 站点满足以下条件时会自动显示横幅：

- 站点部署 manifest.json，该文件需配置如下属性：
    - [`short_name`](#自定义名称) （用于主屏幕显示）
    - [`name`](#自定义名称) （用于安装横幅显示）
    - [`icons`](#自定义图标) （其中必须包含一个 `144x144` 且 mime 类型为 `image/png` 的图标声明）
    - [`start_url`](#设置启动网址) （应用启动地址）
- 站点注册 Service Worker。
- 站点支持 HTTPS 访问。
- 站点在同一浏览器中被访问至少两次，两次访问间隔至少为 5 分钟。

**注** 应用安装横幅是一种新兴的技术，对应的显示横幅的条件在将来可能会有所变化。

#### 应用安装横幅事件

浏览器会根据 manifest.json 提供的相关配置生成应用安装横幅，同时使用一组条件和访问频率启发式算法来确定何时显示横幅。一般来说，浏览器管理触发提示的时间，不一定满足网站需求，因此浏览器也提供了一些事件接口供网站开发者使用。

##### 判断用户是否安装此应用

`beforeinstallprompt` 事件返回一个名为 `userChoice` 的 Promise 对象，并在当用户对横幅进行操作时进行解析。promise 会返回属性 `outcome`，该属性的值为 `dismissed` 或 `accepted`，如果用户将网页添加到主屏幕，则返回 `accepted`。

例如：

```javascript

window.addEventListener('beforeinstallprompt', function (e) {
    // beforeinstallprompt event fired

    e.userChoice.then(function (choiceResult) {
        if (choiceResult.outcome === 'dismissed') {
            console.log('用户取消安装应用');
        }
        else {
            console.log('用户安装了应用');
        }
    });
});

```

利用 `userChoice` 返回的 Promise 对象，可以根据用户的安装选择结果进行互动。


##### 取消或延迟安装横幅的触发事件

网站虽然不能主动触发安装横幅的显示事件，但是当该事件被浏览器触发之后，可以对其进行取消或者延迟。

通过阻止 `beforeinstallprompt` 事件的默认行为，即可取消横幅弹出：

```javascript

window.addEventListener('beforeinstallprompt', function (e) {
    e.preventDefault();
    return false;
});

```

`beforeinstallprompt` 事件返回一个名为 `prompt` 的方法，通过执行该方法可以触发安装横幅的显示。为了实现显示事件的延迟操作，可以将 beforeinstallprompt 事件的返回值给存储起来，再异步地调用 `prompt()`。

```javascript

var deferredPrompt = null;

window.addEventListener('beforeinstallprompt', function (e) {
    // 将事件返回存储起来
    deferredPrompt = e;

    // 取消默认事件
    e.preventDefault();
    return false;
});

// 当按钮点击事件触发的时候，再去触发安装横幅的显示
button.addEventListener('click', function () {
    if (deferredPrompt != null) {
        // 异步触发横幅显示
        deferredPrompt.prompt();
        deferredPrompt = null;
    }
});

```

通过 `prompt()` 触发显示的横幅，同样可以通过 `userChoice` 去监测用户的安装行为：

```javascript

button.addEventListener('click', function () {
    if (deferredPrompt != null) {
        // 异步触发横幅显示
        deferredPrompt.prompt();
        // 检测用户的安装行为
        deferredPrompt.userChoice.then(function (choiceResult) {
            console.log(choiceResult.outcome);
        });

        deferredPrompt = null;
    }
});

```

#### 示例

// @TODO

完整的项目代码可以[戳这里](https://github.com)。

示例项目结构如下：

```

|
|----index.html
|
|----manifest.json
|
|----sw.js // Service Worker

```

manifest.json 的配置如下：

```json

{
    "short_name": "短名称",
    "name": "这是一个完整名称",
    "icon": [
        {
            "src": "icon.png",
            "type": "image/png",
            "sizes": "144x144"
        }
    ],
    "start_url": "./index.html"
}

```

对应的横幅显示及点击添加效果如下图所示：

// @TODO

代码添加事件监听：

```javascript

window.addEventListener('beforeinstallprompt', function (e) {
    e.userChoice.then(function (choiceResult) {
        alert(choiceResult.outcome);
    });
});

```

则点击添加效果如下图所示：

// @TODO

将事件监听更改如下：

```javascript

var dfdPrompt = null;
var button = document.getElementById('btn');

window.addEventListener('beforeinstallprompt', function (e) {
    // 存储事件
    dfdPrompt = e;
    // 显示按钮
    button.style.display = 'block';
    // 阻止默认事件
    e.preventDefault();
    return false;
});

button.addEventListener('click', function (e) {
    if (dfdPrompt == null) {
        return;
    }
    // 通过按钮点击事件触发横幅显示
    dfdPrompt.prompt();
    // 监控用户的安装行为
    dfdPrompt.userChoice.then(function (choiceResult) {
        alert(choiceResult.outcome);
    });
    // 隐藏按钮
    button.style.display = 'none';
    dfdPrompt = null;
});

```

当浏览器触发横幅显示事件时，页面中的按钮将显示出来，同时横幅显示事件被取消；点击按钮时，应用安装横幅才会显示出来：

// @TODO

### 引导用户安装原生应用

PWA 站点允许定义类似应用安装横幅的形式去推广原生应用。

#### 显示原生应用安装横幅的条件

浏览器在 PWA 站点满足以下条件时会自动显示横幅：

- 站点部署 manifest.json，该文件需配置如下属性：
    - [`short_name`](#自定义名称) （用于主屏幕显示）
    - [`name`](#自定义名称) （用于安装横幅显示）
    - [`icons`](#自定义图标) （其中必须包含一个 `144x144` 且 mime 类型为 `image/png` 的图标声明）
    - 包含原生应用相关信息的 `related_applications` 对象
- 站点注册 Service Worker。
- 站点支持 HTTPS 访问。
- 站点在同一浏览器中被访问至少两次，两次访问间隔至少为 `2` 天。

其中 `related_applications` 的定义如下：

- `related_applications`:
    `Array.<AppInfo>` 关联应用列表

AppInfo 的属性值包括：

- `platform`:
    `{string}` 应用平台
- `id`:
    `{string}` 应用id

例如：

```json

"related_applications": [
    {
        "platform": "play",
        "id": "com.baidu.samples.apps.iosched"
    }
]

```

如果只希望用户安装原生应用，而不需要弹出横幅引导用户安装PWA，那么可以在 manifest.json 设置：

```json

"prefer_related_applications": true

```
