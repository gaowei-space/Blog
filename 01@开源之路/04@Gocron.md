# Gocron
[![Downloads](https://img.shields.io/github/downloads/ouqiang/gocron/total.svg)](https://github.com/gaowei-space/gocron/releases)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?maxAge=2592000)](https://github.com/ouqiang/gocron/blob/master/LICENSE)
[![Release](https://img.shields.io/github/release/gaowei-space/gocron.svg?label=Release)](https://github.com/gaowei-space/gocron/releases)



## 项目简介

**Gocron**-定时任务管理系统，使用Go语言开发的轻量级定时任务集中调度和管理系统, 用于替代**Linux-crontab**。

该项目fork于[ouqiang/gocron](https://github.com/ouqiang/gocron),根据实际需求，对于原作进行了迭代，发布了v1.6版本。



## 迭代

### v1.6

* 优化整体**界面样式与布局**，包括界面色系，列表，详情，按钮组，分页等
* 调整权限等级，增加**超级管理员**，可以管理所有任务；**管理员**调整为管理自己的任务和查看其他任务和日志，普通用户与原有权限一致，仅可查看所有任务和日志
* 任务详情页增加快捷选择crontab按钮组
* 任务详情页支持更改任务状态
* 任务列表支持**标签**,**命令**搜索



## 截图



![列表](https://user-images.githubusercontent.com/10205742/184531121-f5faa1a9-4d13-4132-a96d-848375765cda.jpg)



![日志](https://user-images.githubusercontent.com/10205742/184531126-0f159cda-8774-4185-9132-194e66cd5d3c.jpg)



![节点](https://user-images.githubusercontent.com/10205742/184531128-7a9a07a9-cac2-4dea-a37a-5cb57479a528.jpg)



![webhook](https://user-images.githubusercontent.com/10205742/184531159-582fd407-bed1-4ed4-a469-e8b9d5af67cb.jpg)

