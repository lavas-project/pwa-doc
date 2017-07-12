# 介绍

账号是网站必不可少的组成部分。账号体系的存在，可以让网站给用户提供分级服务，同时网站也能够通过收集用户行为实现精准推送。但账号的存在将使得用户不得不多出一步登录的步骤，要知道根据“漏斗模型”理论，从起点到终点，每个环节都会产生用户的流失，依次递减。因此想办法省去烦人的账号密码输入过程，不但能提高用户体验，也能够提高网站转化率。

本文将从传统的“记住密码”功能实现、凭证管理 API 两个方面去介绍自动登录的实现，并介绍了账号密码登录和第三方登录结合凭证管理 API 实现自动登录的新思路。

- [传统“记住密码”功能实现](./remember-me.md)
- [凭证管理 API](./credential-management-api/introduction.md)
    - [账号密码登录](./credential-management-api/password-credential.md)
    - [第三方登录](./credential-management-api/federated-credential.md)
