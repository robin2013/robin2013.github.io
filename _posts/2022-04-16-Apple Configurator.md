---
layout: post
title:  "Apple Configurator 2"
date:   2022-04-16 15:12:52 +0800
categories: jekyll
tags: 逆向
---
#  Apple Configurator 2

准备工作： Apple ID账号密码， Apple设备。

1.1 在App Store搜索“Apple Configurator 2”下载安装（最低适配macOS 10.14）


1.2 连接设备到Mac, 菜单 -> 账户 -> 登录


1.3 添加 -> 应用， 这里会显示你的已购记录，选中应用后添加。这个过程和网速有关，需要等待应用在手机上安装完成。

1.4 command+shift+G 进入文件夹 `~/Library/Group Containers/K36BKF7T3D.group.com.apple.configurator/Library/Caches/Assets/TemporaryItems/MobileApps/`

**备注**: 系统会在ipa安装完成之后清除 1.4 目录下的所有文件, 所以拷贝要迅速 