---
title: Hexo 使用简介
tags:
categories: HEXO
---

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post" 或者自己在相应文件夹下创建"My New Post".md文件,同样会生成相应的文章
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server 或者 hexo s,这个时候就可以在本地查看了,在执行这个命令的时候可能会提示命令找不到，
那么你需要执行npm install hexo-server --save命令安装server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
注意：重新部署以后需要先执行npm install hexo-deployer-git --save，否则hexo deploy会没反应，暂时还不知道为什么
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

### 测试主题是否完备
``` bash
在_config.yml文件中更改theme以后，执行命令测试主题是否完备
$ hexo s --debug
```