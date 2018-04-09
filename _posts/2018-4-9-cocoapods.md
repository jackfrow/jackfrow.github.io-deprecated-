---
title: 2018 最新安装cocoapods.
date: 2018-4-09 22:46:49
tag: 学习笔记 
categories: 学习笔记
---

> 事件起因：制作了一个cocoapods库，已经发布成功，但是搜索不出来.删除索引文件并且重新生成也没有用，主要问题就是repos仓库的内容无法更新.

# 一、简介

# 什么是CocoaPods

CocoaPods是OS X和iOS下的一个第三类库管理工具，通过CocoaPods工具我们可以为项目添加被称为“Pods”的依赖库（这些类库必须是CocoaPods本身所支持的），并且可以轻松管理其版本。

CocoaPods的好处

1、在引入第三方库时它可以自动为我们完成各种各样的配置，包括配置编译阶段、连接器选项、甚至是ARC环境下的-fno-objc-arc配置等。

2、使用CocoaPods可以很方便地查找新的第三方库，这些类库是比较“标准的”，而不是网上随便找到的，这样可以让我们找到真正好用的类库。

# 二、Cocoapods安装步骤

## 1、升级Ruby环境

> 终端输入：$ gem update --system

若提示没有权限，

> 这时应该输入：$ sudo gem update --system

## 2、更换Ruby镜像

首先移除现有的Ruby镜像

> 终端输入：$ gem sources --removehttps://rubygems.org/

然后添加国内最新镜像源（淘宝的Ruby镜像已经不更新了）

> 终端输入：$ gem sources -ahttps://gems.ruby-china.org/

执行完毕之后输入gem sources -l来查看当前镜像

> 终端输入：$ gem sources -l

如果结果是

*** CURRENT SOURCES ***

https://gems.ruby-china.org/

说明添加成功，否则继续执行$ gem source -a  https://gems.ruby-china.org/来添加

## 3、安装CocoaPods

接下来开始安装

> 终端输入：$ sudo gem install cocoapods

如果提示没有权限:

> 终端输入：$ sudo gem install -n /usr/local/bin cocoapods

安装完成之后再执行pod setup（PS：这个过程是漫长的，要有耐心,如果你是黑人脸的话，那基本就停在这了）

> 终端输入：$ pod setup

## 4.pod setup无法执行解决方案

### 4.1 安装国内镜像源(但是目前没有找到国内仍在维护的镜像源)

> pod repo remove master

> pod repo add master https://git.coding.net/CocoaPods/Specs.git

> pod repo update

### 4.2 使用git 把镜像源克隆下来，然后放到~/.cocoapods/repos

> git clone https://github.com/CocoaPods/Specs

### 4.3 从浏览器 [https://github.com/CocoaPods/Specs](https://link.jianshu.com/?t=https://github.com/CocoaPods/Specs) 下载.

![img](https://upload-images.jianshu.io/upload_images/1877622-b947ebd8335c8680.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

zip下载方式.png

使用ZIP下载的方式下载下来文件，解压到repos目录.

### 4.4 如果你有同事的话，找他拷贝一份~/.cocoapods/repos目录下的文件（这种方法最轻松！！）

之后 只需要cd ~/.cocoapods/repos/master路径，执行一步(如果.git不存在的话)

> ​    git init

### 4.5 使用github客户端进行下载镜像源(博主目前就使用的这种方式，感觉十分好用)

![img](https://upload-images.jianshu.io/upload_images/1877622-181805ef89d34a3f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/174)

github客户端

至此你的电脑上已经能够正常使用cocoapods了。