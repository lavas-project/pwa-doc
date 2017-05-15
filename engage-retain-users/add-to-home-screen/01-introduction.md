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

- 基本功能
    - [自定义名称](./02-basic-conditions.md/#自定义名称)
    - [自定义图标](./02-basic-conditions.md/#自定义图标)
    - [设置启动网址](./02-basic-conditions.md/#设置启动网址)
    - [设置作用域](./02-basic-conditions.md/#设置作用域)

- 改善应用体验
    - [添加启动画面](./03-improved-webapp-experience.md/#添加启动画面)
    - [设置显示类型](./03-improved-webapp-experience.md/#设置显示类型)
    - [指定显示方向](./03-improved-webapp-experience.md/#指定页面显示方向)
    - [设置主题色](./03-improved-webapp-experience.md/#设置主题颜色)

- 应用安装横幅
    - [引导用户添加应用](./04-app-install-banners.md/#引导用户添加应用至主屏幕)
    - [引导用户安装原生应用](./04-app-install-banners.md/#引导用户安装原生应用)
