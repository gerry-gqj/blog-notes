---
title: git设置代理
typora-root-url: ../
date: 2022-11-15 12:05:37
tags: git
categories: git
---











## 全局代理



`http` 代理

```bash
git config --global http.proxy http://127.0.0.1:1080
```



`https`代理

```bash
git config --global https.proxy http://127.0.0.1:1080
```



`socket`方式

```bash
git config --global http.proxy socks5://127.0.0.1:7890
```

```bash
git config --global https.proxy socks5://127.0.0.1:7890
```





## 针对`github`代理

```bash
git config --global http.https://github.com.proxy socks5://127.0.0.1:7890
```





## 取消代理



```bash
git config --global --unset http.proxy
```

```bash
git config --global --unset https.proxy
```













