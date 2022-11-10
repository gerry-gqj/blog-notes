---
title: npm、yarn、pnpm镜像
date: 2022-11-11 00:36:03
tags:
 - npm
 - yarn
 - pnpm
 - 包管理
 - 镜像源
categories:
 - web前端
 - 包管理
---



## npm

```bash
# 华为镜像
npm config set registry https://mirrors.huaweicloud.com/repository/npm/
npm config set disturl https://mirrors.huaweicloud.com/nodejs/
npm config set electron_mirror https://mirrors.huaweicloud.com/electron/
# 淘宝镜像
npm config set registry https://registry.npm.taobao.org
npm config set disturl https://npm.taobao.org/dist
npm config set electron_mirror https://npm.taobao.org/mirrors/electron/
```



## yarn

```bash
# 华为镜像
yarn config set registry https://mirrors.huaweicloud.com/repository/npm/
yarn config set disturl https://mirrors.huaweicloud.com/nodejs/
yarn config set electron_mirror https://mirrors.huaweicloud.com/electron/
# 淘宝镜像
yarn config set registry https://registry.npm.taobao.org
yarn config set disturl https://npm.taobao.org/dist
yarn config set electron_mirror https://npm.taobao.org/mirrors/electron/
```



## pnpm

```bash
# 华为镜像
pnpm config set registry https://mirrors.huaweicloud.com/repository/npm/
pnpm config set disturl https://mirrors.huaweicloud.com/nodejs/
pnpm config set electron_mirror https://mirrors.huaweicloud.com/electron/
# 淘宝镜像
pnpm config set registry https://registry.npm.taobao.org
pnpm config set disturl https://npm.taobao.org/dist
pnpm config set electron_mirror https://npm.taobao.org/mirrors/electron/
```



