---
title: Mac快捷键
date: 2018-05-01 16:16:26
tags: [快捷键]
categories: 基础
---
本文是对苹果电脑上的一些快捷键的记录。

<!-- more -->

按钮说明：
* command 简记为 cmd
* delete 简记为 del
* control 简记为 ctl
* option 简记为 opt
* 空格 记为 space
* 关机键 记为 off

#  Mac系统
## 切换
|按钮|功能|
|---|---|
|cmd + tab | 在应用之间切换 |
|cmd + shift + tab  | 在应用之间切换，反向 |

## 通用
|按钮|功能|
|---|---|
|cmd + N | 新建 |
|cmd + O | 打开 |
|cmd + S | 保存 |
|cmd + A | 全选  |
|cmd + shift + S | 另存为 |
|cmd + W | 关闭 |
|cmd + Q | 退出 |
|cmd + F | 搜索 |

## 截屏
|按钮|功能|
|---|---|
|cmd + shift + 3 | 截取整个屏幕 |
|cmd + shift + 4 | 截取选择区域 |
|cmd + shift + 4 + space | 截取整个应用界面 |

## 文件管理器
|按钮|功能|
|---|---|
|space | 快速查看文件 |
|cmd + I | 显示文件简介 |
|cmd + C | 复制 |
|cmd + V | 粘贴 |
|cmd + del | 删除 |
|cmd + shift + del | 清空回收站 |

## 系统
|按钮|功能|
|---|---|
|ctl + shift + off| 关闭显示器|
|cmd + opt + off| 睡眠|
|cmd + opt + esc| 强制退出程序|
|cmd + ctl + off| 重启|
|cmd + space | 切换输入法|

# 谷歌浏览器
[Chrome for Mac键盘快捷键！来自Google Chrome官网！](https://blog.csdn.net/duanyipeng/article/details/8637391)

|按钮|功能|
|---|---|
|cmd + +| 放大|
|cmd + -| 缩小|
|cmd + T| 新标签页|
|cmd + N| 新窗口|
|cmd + R| 刷新|
|cmd + W| 关闭当前标签页|
|cmd + ～| 切换应用窗口|
|cmd + shift + N| 隐身打开新窗口|
|cmd + O| 打开计算机中的文件|
|cmd + 链接| 从新标签页中打开链接|
|shift + 链接| 从新窗口中打开链接|
|cmd + opt + 左右箭头| 切换标签页|
|cmd + shift + W| 关闭当前窗口|
|cmd + shift + T| 重新打开上次关闭的标签，最多记住10个|
|cmd + [| 标签页的上一页历史 |
|cmd + ]| 标签页的下一页历史 |
|cmd + M| 最小化窗口|
|cmd + H| 隐藏窗口|
|cmd + Q| 关闭浏览器|
|cmd + ctl + F| 最大化当前标签|
|cmd + shift + B| 打开和关闭书签栏|
|cmd + opt + B| 打开书签管理器|
|cmd + ,| 打开偏好设置|
|cmd + Y| 打开历史记录|
|cmd + shift + J| 打开下载内容|
|cmd + shift + del| 打开清楚浏览数据对话框|
|输入网址时 + tab| 进行提示补全|
|cmd + L| 选中网址|
|opt + 左右箭头| 可以跳跃单词|
|opt + shift + 左右箭头| 选中下一个单词|
|cmd + del| 删除地址栏中光标前的关键字|
|cmd + opt + I|打开开发者工具|
|cmd + D| 将当前网页保存为书签|
|cmd + shift + D| 将所有打开的网页保存到书签|
|space | 向下滚动网页|
|cmd + opt + F| 使用谷歌搜索|

# 终端
[Mac Terminal 常用快捷键 Bash shell](http://www.etwiki.cn/mac-os/bash-shell.html)

|按钮|功能|
|---|---|
|ctl + A| 跳到行首|
|ctl + E| 跳到行尾|
|opt + 左右箭头| 左右移动一个单词位|
|ctl + K| 从光标处删除到行尾|
|ctl + U| 从光标处删除到行首|
|ctl + W| 从光标处删除到字首|
|ctl + Y| 粘贴最后一次被删除的单词|
|ctl + R| 逆向搜索命令历史|
|ctl + G| 从历史搜索模式退出|
|ctl + P| 历史中的上一条命令|
|ctl + N| 历史中的下一条命令|

# Git常用命令
[Git常用命令总结](https://www.cnblogs.com/my--sunshine/p/7093412.html)

|按钮|功能|
|---|---|
|git status| 查询repo的状态|
|git status -s| 输出标记会有两列,第一列是对staging区域而言,第二列是对working目录而言|
|git log| 查看提交历史|
|git log --oneline| 查看提交历史，每次提交显示一行|
|git log --oneline --graph| 图形化地表示出分支合并历史|
|git log branchname | 显示特定分支的log|
|git log --author=[author name]| 查看作者的提交历史|
|git log --grep=keywords| 根据commit信息过滤log|
|git log -p| 把每次提交的diff计算出来,作为一个patch显示|
|git log --stat|  把每次提交的diff计算出来,--stat比-p的输出更简单一些|
|git add| 添加新的文件或改动到暂存区(staging area)|
|git add .| 递归添加当前工作目录中所有文件|
|git diff| 比较工作目录中当前文件和暂存区域快照之间的差异,即修改后还没暂存起来的内容|
|git diff --cached| 查看已经暂存起来的文件和上次提交时的快照之间的差异|
|git diff --staged| 效果同上，Git 1.6.1 及更高版本|
|git diff HEAD| 比较工作目录和上次提交之间所有的改动|
|git diff --stat| diff也可以加上--stat参数来简化输出|
|git diff [branchA] [branchB]| 可以用来比较两个分支，实际上会返回一个由A到B的patch|
|git diff [branchA]…[branchB]|看两个分支分开以后各自的改动都是什么，实际上是:git diff $(git merge-base [branchA] [branchB]) [branchB]的结果|
|git commit -m “commit message"| 提交add的内容|
|git commit -a| 会先把所有已经track的文件的改动add进来,然后提交，对于没有track的文件,还是需要git add一下|
|git commit --amend|增补提交. 会使用与当前提交节点相同的父节点进行一次新的提交,旧的提交将会被取消|
|git reset|  TODO |

// TODO VIM
