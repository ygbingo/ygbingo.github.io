---
title: "Windows11安装个人博客框架hugo"
date: 2024-05-14T11:54:44+08:00
draft: false
tags:
    - hugo
    - choco
    - 博客
---

> 本篇博客采用choco的方式安装hugo

# Installing

## Install choco
1. 使用管理员方式打开Powershell
2. 执行```Get-ExecutionPolicy```, 如果出现提示是否确认，输入A可以自动确认所有环节
- 如果返回```Restricted```: 执行```Set-ExecutionPolicy AllSigned```, 然后弹出>, 再执行```> Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1')) ```
3. 等待...
4. 如果没有异常提示，就代表安装成功啦！可以执行```choco```验证。

## Install Hugo
1. 安装choco之后，直接执行 ```choco install hugo-extended```
2. 等待安装完成

# Reference
1. [hugo install](https://gohugo.io/installation/windows/)
2. [Installing Chocolatey](https://chocolatey.org/install)