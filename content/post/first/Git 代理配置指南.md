---
title: "Git代理配置指南"
date: 2025-11-05T10:00:00+08:00
draft: false  # 确保这不是草稿
author: "Gannicus"
# --- 重点在这里 ---
categories:
    - "技术"
    - "git"
tags:
    - "git"
    - "proxy"
# --------------------
---

# Git 代理配置指南

本文档旨在解决一个常见的开发需求：为 Git 设置一个全局代理，同时允许特定的项目（如公司内网项目或国内代码托管平台项目）不通过代理进行连接。

## 核心原理：Git 配置的三个层级

Git 的配置是分层级的，高层级的配置会覆盖低层级的配置。其优先级顺序为：

1. 局部 (local)：仅对当前仓库生效。配置存储在项目目录的 .git/config 文件中。
    
2. 全局 (global)：对当前用户的所有仓库生效。配置存储在用户主目录的 ~/.gitconfig 文件中。
    
3. 系统 (system)：对系统中所有用户的所有仓库生效。
    

利用这个优先级机制，我们可以设置一个全局代理，然后在需要例外的项目里进行局部取消。

---

## 第一步：设置全局代理

首先，为当前用户的所有 Git 操作配置一个默认的代理。

打开终端（Terminal, Git Bash, CMD, or PowerShell），执行以下命令。

1. 针对 HTTP/HTTPS 代理 (将 127.0.0.1:7890 替换为你的代理地址和端口)

- Bash

```bash 

git config --global http.proxy http://127.0.0.1:7890

git config --global https.proxy https://127.0.0.1:7890
```



  
  

2. 针对 SOCKS5 代理

- Bash


```bash
git config --global http.proxy socks5://127.0.0.1:7890

git config --global https.proxy socks5://127.0.0.1:7890
```


设置完成后，你所有的 git clone, pull, push 等操作默认都会通过此代理。

---

## 第二步：为特定项目设置例外（不走代理）

针对不需要走代理的项目，你有以下两种主流方法。

### 方法一：针对单个项目进行局部取消（最常用）

此方法通过在项目内部添加一个“局部配置”来覆盖“全局配置”。

进入你的项目目录:  
- Bash  
```bash
cd /path/to/your/project
```


1.   使用 --local 标志取消该项目的代理设置:  

- Bash  

```bash
git config --local --unset http.proxy

git config --local --unset https.proxy
```


2. 执行后，Git 会在这个项目的 .git/config 文件中记录这个取消操作。从此，只有在这个项目目录下的 Git 操作会直连，而其他所有项目依然走全局代理。
    

### 方法二：通过 noProxy 排除特定域名

如果你所有不需要代理的项目都托管在少数几个固定的域名下（例如 gitee.com 或公司内部的 gitlab.internal.com），可以设置一个全局的例外清单。

编辑全局配置，添加不走代理的域名列表 (多个域名用逗号 , 分隔，支持 * 通配符):  
- Bash  
```bash
git config --global http.noProxy "localhost,127.0.0.1,gitee.com,*.internal.com"

```

1. 这个方法的好处是“一次配置，永久生效”，所有匹配这些域名的仓库都会自动绕过代理，无需逐个项目设置。
    

---

## 疑难解答：为什么局部设置不生效？

如果你发现设置了 --local --unset 后，项目依然在走代理，90% 的可能性是受到了环境变量的干扰。

#### 最高优先级：环境变量 (Environment Variables)

Git 会优先检查系统中的 HTTP_PROXY 和 HTTPS_PROXY 环境变量，它们的优先级高于所有 git config 配置。

1. 检查环境变量

- macOS / Linux:  
- Bash  
```bash
echo $HTTP_PROXY

echo $HTTPS_PROXY
```


-   Windows (CMD):  
- Bash
```bash
echo %HTTP_PROXY%

echo %HTTPS_PROXY%
```


-   Windows (PowerShell):  
- Bash  
```bash
$env:HTTP_PROXY

$env:HTTPS_PROXY
  ```


-   如果上述命令返回了你的代理地址，就说明问题出在这里。

2. 解决方法

临时解决：在 git 命令前临时取消环境变量。  
- Bash  
	
	```bash 
	#For macOS / Linux
	HTTP_PROXY="" HTTPS_PROXY="" git pull
	```


-   
    
- 永久解决：找到设置这些环境变量的地方（如 .bashrc, .zshrc, 或 Windows 的系统属性面板）并将其删除或注释掉，然后重启终端。
    

#### 其他可能原因

- URL 协议错误：http.proxy 配置只对 https:// 或 http:// 协议的远程地址生效。如果你的地址是 SSH 格式 (git@github.com:...)，它将不会走 HTTP 代理。请用 git remote -v 检查。
    

强制设置为空：有时 --unset 不如直接设置为空值来得更明确。  
```Bash 
git config --local http.proxy ""

git config --local https.proxy ""
```
 
---

## 附录：有用的检查命令

- 查看所有配置及其来源文件:  
 ``` Bash  
git config --list --show-origin
  ```


- 查看当前项目最终生效的代理配置:  

```Bash  
# 进入项目目录后执行

git config http.proxy
```


- 开启详细网络日志进行诊断:  
```Bash  
    GIT_CURL_VERBOSE=1 git fetch
```
