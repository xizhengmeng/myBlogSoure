---
title: React Native概述
date: 2016-08-02 16:13:35
tags:
- RN
categories: iOS
---
## 什么是RN
React 是一个用于创建用户界面的 JavaScript 函数库，通常被表述为 MVC 中的 V（View，视图）。

React 知道根据组件的状态进行重新渲染，并且保存一个虚拟 DOM 以实现高效的重新渲染。这种方法非常棒，因为我们写代码时就好像在重新渲染整个模板，而实际上 React 只会更新发生过改动的 DOM。
>特点：JSX语法，组件化模式，Virtual DOM，单向数据流
>基本模式：每个React应用可视为组件的组合，而每个React组件由属性和状态来配置，当状态变化的时候更新UI，组件的结构是由DOM来维护的，确保了实际更新的DOM只包括真正产生了状态变化的部分。

### RN提供了哪些能力？
- 基于原生UI组件
- 手势识别
- 基于FlexBox的css布局模式
- 跨平台开发
- 就React，jsx的组件化开发模式
- 可使用npm中的模块
- Chrome Dev Tool集成

>说明：我们所写的任何组件，最后都会转换为对应的UIKit中的控件，最基础的如UIView，UIImageView，UILabel等，但是UITableView是没有的，因为不断的在主线程与JS线程之间通讯的消耗太大，导致操作延迟严重
### JSX
React 与常见框架的最大差别在于，JavaScript 逻辑与 Markup(标记)模板使用 JSX 语法写在同一个文件中。

## 开发准备
### 环境配置
### 初始化项目
### 开发工具

## 布局
## 

## 问题
- 更新的js文件如何下发？
- 版本控制如何做？
- 如何查看框架的版本？
- 有没有环境限制，比如说不能在某种机器上跑，我现在知道肯定不能在iOS6及以下跑
- js文件执行过程，是否与jspatch有类似之处
- react native在mac下依赖node.js运行，那么在iOS中又是怎么运行的呢？
- RN的运行过程，深度原理解析
- RN在iOS平台的应用(这是重点)现有项目如何添加rn模块
- RN是如何实现View的，当我创建一个View的时候，实际上RN做了什么事情？
- 当我触摸屏幕的时候，RN是如何做出响应的？
- JavaScript线程是怎么回事？
- RN如何使用多线程？
