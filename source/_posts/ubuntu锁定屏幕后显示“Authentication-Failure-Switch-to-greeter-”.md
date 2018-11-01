title: ubuntu锁定屏幕后显示“Authentication Failure Switch to greeter...”
author: 木子三金
tags:
  - linux
  - bug
  - ubuntu
categories:
  - 问题记录
  - linux
date: 2018-09-11 18:16:00
---
- Ubuntu版本：Ubuntu 16.04.5 LTS
- 问题描述：锁屏后登录框提示“Authentication Failure Switch to greeter...”
- bug记录地址：[Authentication Failure](https://bugs.launchpad.net/ubuntu/+source/unity/+bug/1733557)
- 解决方法：在命令行执行如下命令：
		$ sudo apt install --reinstall lightdm
		$ sudo service lightdm restart
 注意，第二个命令会停止当前打开的app和进程，请保存数据后在执行。