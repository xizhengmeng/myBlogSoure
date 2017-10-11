---
title: Hexo 配置/使用教程
date: 2015-03-09 10:39:20
tags:
- 工具
categories: 基础

---

### hexo安装
>前提已经安装了node.js和git

- `npm install hexo --no-optional`
- `hexo init myBlog`//创建hexo的文件夹
- `cd myBlog`
- `npm install`
- `hexo server`//打开对应的链接可看到效果 

 <!--more--> 
### git配置

- 创建git仓库`name/name.github.io`
- 添加信息到配置文件`_config.yml`

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:xizhengmeng/xizhengmeng.github.io.git
  branch: master
```

- 安装插件`npm install hexo-deployer-git --save`

>如果要访问这个网站直接`xizhengmeng.github.io`就好了

### 使用
- 进入博客所在的文件夹，比如我说的是~/myBlog
- `hexo clean`
- `hexo g`//生成
- `hexo d`//推送到github服务器
- `hexo new "fileName"`创建新的文件，在_post文件中
- `hexo new page fileName`创建一个文件夹，然后在其中创建一个index.md文件，比如标签云页面，或者分类页面


### 绑定域名
- 申请域名，直接搜索`hexo和github，域名和github绑定的问题`
- 生成CNAME文件，并添加到source文件夹下，与post同级目录
- 更改配置文件--——config.yml`url: www.hanson647.com`

### 主题安装
- 进入Hexo文件的根目录`cd Hexo`
- 拉取主题文件，并在本地创建对应的文件`git clone https://github.com/kathyqian/crisp-ghost-theme.git themes/gost`
- 更改配置文件的主题为该主题
```
theme: yilia
```
- 重新部署

>主题更新`git pull`

>使用了yelee主题，主题使用说明`http://moxfive.coding.me/yelee/`

### 搜索功能配置
- 安装插件`npm install --save hexo-generator-search`
- 设置主题配置文件
```
search: 
  on: true
  onload: true
```
- 设置hexo配置文件
```
search:
  path: search.xml
  field: post
```
- 重新生成

最后你会在publick文件中看到一个search.xml文件，最后你会在你的网站中看到你的搜索框

#### 注意
在工程的_config文件中，要将url对应的字段设置为/，否则你会发现搜索会自动加上一个前置的url，这样就又错乱了

### 添加标签以及标签云页面
- 创建tags页面`hexo new page tags`
- 直接在文章中的tags关键字后边添加对应的标签就好
```
tags:
- 工具
```
- 编辑刚才创建的tags页面，将其类型更改为tags
```
title: tags
date: 
type: "tags"
```
- 在主题配置文件中，将tags添加到menu中
```
menu:
  home: /
  archives: /archives
  tags: /tags
```
### 创建关于页面
`hexo new page about`


### 容易出现的问题
#### hexo d
- `Error: ssh_exchange_identification: read: Connection reset by peer`
说明是网关的问题，我是解决了一下网络解决的，一开始连接的是京东的guest网络，出现该问题，切换为access解决

#### md文件格式
标题中#号要与后边的标题留出间隙，也就是一个空格，否则hexo渲染是不认的

### 参考链接
- next主题设置<http://theme-next.iissnan.com>
- hexo官网<https://hexo.io/zh-cn/>
