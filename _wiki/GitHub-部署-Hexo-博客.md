---
layout: article
title: GitHub 部署 Hexo 博客
tags: []
key: f4cfc411-a9cf-4fc4-8747-a9d367fd0786
---

采用 Hexo 生成静态博客页面,然后使用 GitHub 提供的静态页面服务进行部署.

<!--more-->

## 准备

1. 本地安装 Git  
2. 本地安装 Nodejs
3. 创建一个特殊的仓库, 仓库名为`GitHub账号名.github.io` 并添加本机的`ssh`公钥到`GitHub`里

## 安装 Hexo （[参考](http://hexo.io/))

在本地执行如下命令:

```bash
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

就启动了本地的`Hexo`,在浏览器中可以打开[http://localhost:4000](http://localhost:4000)

> 这里是`Hexo 3.0`,但是有很多`Hexo`插件不兼容,可以使用如下命令安装2.X版本  

```bash
npm install hexo@2.8.3 -g
hexo init blog
```

## 更换主题

使用`Git`从`GitHub`中下载公开的主题,如:
`light`仓库地址: `git@github.com:hexojs/hexo-theme-light.git`
`yilia`仓库地址: `git@github.com:litten/hexo-theme-yilia.git`
安装命令:

```bash
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

## 生成静态页面

使用命令:

```bash
hexo generate
```

生成的静态页面都在`public`文件夹里,下边部署时发布的就是此文件夹里的内容

## 部署到`GitHub`([参考](http://hexo.io/docs/deployment.html))

修改`_config.yml`:

```yml
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:albertchendao/albertchendao.github.io.git
  branch: master
```

先安装部署插件:

```bash
npm install hexo-deployer-git --save
```

然后使用命令:

```bash
hexo deploy
```

在浏览器中就可以看到了[http://albertchendao.github.io/](http://albertchendao.github.io/)

## 使用

### Hexo 创建分类页

添加一个 分类 页面,并在菜单中显示页面链接.

1. 新建一个页面,命名为 `categories` .命令如下:

    ```bash
        hexo new page categories
    ```

2. 编辑刚新建的页面,将页面的类型设置为 `categories` ,主题将自动为这个页面显示所有分类.

    ```yml
        title: 分类
        date: 2014-12-22 12:39:04
        type: "categories"
        ---
    ```

3. 在菜单中添加链接.编辑主题的 `_config.yml` ,将 `menu` 中的 `categories: /categories` 注释去掉,如下:

    ```yml
        menu:
          home: /
          categories: /categories
          archives: /archives
          tags: /tags
    ```
