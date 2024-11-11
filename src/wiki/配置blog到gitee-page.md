---
title: 配置blog到gitee-page
abbrlink: 17780
date: 2020-09-03 00:00:00
cover: https://hexo.io/themes/screenshots/landscape.png
---
## Git 下载安装

[Git下载](https://registry.npmmirror.com/binary.html?path=git-for-windows/) *不要下便携版*

## Node.js 下载安装

https://nodejs.org/en/download/ *不要下便携版*

两个都安装完后检测是否配置成功

```cmd
node -v
npm -v
```

若 `npm -v` 失败，可能是需要配置环境变量
新建用户变量，变量名为 `NODE_PATH`，变量值为**Node 的安装路径**

<!-- <img src="C:\Users\Yanguan\Desktop\Snipaste_2022-08-14_18-04-02.png" style="zoom: 50%;" /> -->

<!-- <img src="C:\Users\Yanguan\Desktop\Snipaste_2022-08-14_18-05-19.png" style="zoom: 50%;" /> -->

## 安装 hexo

win+r，打开运行，输入 cmd

```cmd
npm install -g cnpm --registry=https://registry.npm.taobao.org  # 安装淘宝镜像
cnpm install -g hexo  # 安装hexo博客生成系统
```

## 创建一个文件夹，对其 `git bash` 命令操作

```cmd
hexo init      # 初始化博客系统，耐心等待
hexo server    # 开启博客本地测试服务器
```

> 测试是否成功？
>
> 在浏览器网址栏中搜索 cmd 生成的网址（http://localhost:4000)，打开会看到 hexo 默认网页。

## 部署到 Gitee

在 Blog 目录下安装一个 hexo 部署插件

```sh
cnpm install --save hexo-deployer-git
npm install hexo-abbrlink --save # 为中文标题生成短链接
```

编辑 _config.yml 文件

### gitee 远程登录

```cmd
git config --global user.email "yanguan_02@126.com"
git config --global user.name "Yanguan"
ssh-keygen -t rsa -C "yanguan_02@126.com"  # 上传登录密钥
```

**注：之后有很多地方等待输入，直接回车。**

配置公钥

https://blog.csdn.net/bryanwang_3099/article/details/114649881

### 各种报错解决

- hexo g 报错

  可能是 md 文件格式不标准
- 本地运行出现 ```extends includes/layout.pug block content include  ./includes/mixins/post-ui.pug #recent-posts.recent-posts +postUI include includes/pagination.pug```

```cmd
npm cache clean --force
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

## 部署好后每次写好记录后只用执行以下命令

```cmd
hexo clean     # 清除缓存
hexo generate  # 生成博客网页
hexo s         # 可以不有，本地查看网页
hexo deploy    # 部署网页
```
