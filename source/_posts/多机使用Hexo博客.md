---
title: 多机使用Hexo博客
date: 2018-05-01 22:14:05
tags: [hexo]
categories: 环境
---

记录一下 hexo 博客迁移到另一台电脑上的操作。

<!-- more -->

# 问题描述
之前博客使用 Hexo ，本地写完后，使用 `hexo g -d` 命令后，就直接在 public 生成站点，将网站内容部署到 github 上，但是原始文件没有上传，这样无法再多台机器上一起使用该博客。

# 解决参考
参考 [《使用hexo，如果换了电脑怎么更新博客？》](https://www.zhihu.com/question/21193762)。

解决思路如下：
1. 使用 master 和 hexo 两个分支。
2. master 分支用来保存生成的网站内容。
3. hexo 分支保存原始的代码。

# 具体操作
1. 创建新分支 hexo，可参考 [《创建Git新分支步骤》](https://blog.csdn.net/xlyrh/article/details/70940753)。
2. 将 hexo 博客的源文件提交到新分支 hexo 上。
3. 然后在 github 上项目上设置默认的分支为你新建的分支（这里新分支为 hexo）。
4. 然后在新的电脑上拷贝这个项目的代码（拷贝默认的 hexo 分支）。
5. 在 hexo 分支上开发，然后提交代码到默认的 hexo 分支：`git add .`、`git commit -m "..."`、`git push origin hexo`。
6. 修改 `_config.yml` 中的 deploy 参数，分支应为 master。
7. 使用 `hexo g -d` 将网站更新到 master 分支上。

# Next 主题缺失问题
按上面操作后，会发现 hexo 博客还是跑不起来，网上查找原因后发现 next 主题文件夹为空。如下：

[《hexo本地测试运行重启后页面空白,提示 : WARN No layout: index.html?》](https://www.zhihu.com/question/38781463?sort=created)

这个问题的解决一下子找不到了。。。基本思路是，拉一个 next 的主题下来，然后使用子模块的设置 submodule，将 next 主题独立到一个仓库中。

由于本质上是利用 github 来做一个云端的存储，所以我直接新建了一个项目，把这个 next 主题传了上去，如果有更改，就进行更新，其实是差不多的。用的时候，直接拷贝下来放入 next 文件夹中即可。

可能之前尝试使用子模块的操作，启动 hexo 报了这个错误：
```
git submodule: already exists in the index
```
这个解决可以参考：
[《git submodule: already exists in the index》](https://my.oschina.net/jerikc/blog/513039)。
