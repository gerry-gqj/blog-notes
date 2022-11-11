---
title: yarn镜像源
date: 2022-11-11 00:23:11
typora-root-url: ../
tags:
 - yarn
 - 包管理
 - 镜像源
categories:
 - web前端
 - 包管理
---



## 查看

```bash
yarn config get registry
```



## 修改

临时更改( --registry 参数)

```bash
yarn add any-touch@latest --registry=https://registry.npmjs.org/
```

持久化更改

```bash
yarn config set registry <源地址>
```



## 示例

原始源

```bash
yarn config set registry https://registry.yarnpkg.com
```

淘宝源

```bash
yarn config set registry https://registry.npm.taobao.org/
```

华为源

```bash
yarn config set registry https://mirrors.huaweicloud.com/repository/npm/
```

