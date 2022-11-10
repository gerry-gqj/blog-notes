---
title: pnpm镜像源
date: 2022-11-11 00:23:25
tags:
 - pnpm
 - 包管理
 - 镜像源
categories:
 - web前端
 - 包管理
---



## 查看

```bash
pnpm get registry 
```



## 修改

临时更改( --registry 参数)

```bash
pnpm --registry https://registry.npm.taobao.org install any-touch
```

持久化更改

```bash
pnpm config set registry <源地址>
```



## 示例

原始源

```bash
pnpm config set registry https://registry.npmjs.org
```

淘宝源

```bash
pnpm config set registry https://registry.npm.taobao.org
```

华为源

```bash
pnpm config set registry https://mirrors.huaweicloud.com/repository/npm/
```

