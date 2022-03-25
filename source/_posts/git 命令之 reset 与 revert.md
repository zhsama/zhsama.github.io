---
title: Git 命令之 Reset 与 Revert
date: 2022-03-18 10:46:22
updated: 2022-03-25 14:21:32
tags: ['git']
categories:
- git
description: 你是否跟笔者一样，有时在git commit时提交错文件，要撤销提交却不知道该如何合理操作的困惑？希望这篇讲解git reset命令的文章可以帮助到你。
---

# Git Reset

git reset 命令可以将当前的`HEAD`重置到特定的状态。

## 概念

首先要搞清楚下面几个概念：

- HEAD：指向当前分支当前版本的游标
- Index：暂存区，当你修改了你的 git 仓库里的一个文件时，这些变化一开始是 **unstaged** 状态，为了提交这些修改，你需要使用`git add`把它加入到 index，使它成为 **staged** 状态。当提交一个 commit 时，index 里面的修改被提交
- working tree：当前的工作目录

## 用法

`git reset`的命令为：

```shell
git reset [<mode>] [<commit>]
```

 git reset 会将当前分支的 HEAD 指向给定的版本，并根据模式的不同决定是否修改 index 和 working tree
常用的<mode>有三种模式：`--soft`, `--mixed`, `--hard`，没有指定 <mode> 默认是`--mixed`模式

## soft

使用`--soft`参数将会仅仅重置 HEAD 到制定的版本，不会修改 index 和 working tree。例如当前分支现在有三次提交，执行`git reset --soft HEAD~`之后，查看git log：

![image](https://user-images.githubusercontent.com/33454514/160063514-924585e2-ff73-4d26-a7d3-21202d24a0bd.png)

而本地文件的内容并没有发生变化，而index中仍然有最近一次提交的修改，这时执行git status会显示这些修改已经在再暂存区中了，无需再一次执行git add

![image](https://user-images.githubusercontent.com/33454514/160063524-adfbb0b1-3bf0-4276-8af2-ac9e8eb71ad9.png)

## mixed

使用`--mixed`参数与`--soft`的不同之处在于，**--mixed** 修改了 **index**，使其与第二个版本匹配。index 中给定 commit 之后的修改被`unstaged`。

![image](https://user-images.githubusercontent.com/33454514/160063677-34caf47f-e0d8-453f-9e60-399bf17f848b.png)

![image](https://user-images.githubusercontent.com/33454514/160063683-48b2685d-524f-47f2-8768-025792865840.png)

 如果现在执行git commit 将不会发生任何事，因为暂存区中没有修改，在提交之前需要再次执行git add

## hard

使用`--hard`同时也会修改 working tree，也就是当前的工作目录，如果我们执行`git reset --hard HEAD~`，那么最后一次提交的修改，包括本地文件的修改都会被清除，彻底还原到上一次提交的状态且无法找回。所以在执行`reset --hard`之前一定要小心！

## 总结

* soft：重置 HEAD 到指定版本，影响 working tree
* hard：会清除本地的修改文件，且无法找回
* mixed：修改 index，需要重新执行 git add

# Git Revert

使用`git revert`也能起到回退版本的作用，不同之处在于：

- `git revert <commit>`会回退到 <commit> 之前的那次提交。比如：`git revert HEAD~3`会回退到最近的第4个提交的状态，而不是第3个
- `git revert`会产生一个新的commit，将这次回退作为一次修改记录提交，这样的好处是不修改历史提交记录

![image](https://user-images.githubusercontent.com/33454514/160064129-9af07499-5ddd-4f25-88e0-a3fc127a7ded.png)

![image](https://user-images.githubusercontent.com/33454514/160064132-faf553f3-d344-4aab-801c-97c37fdb2e0a.png)

![image](https://user-images.githubusercontent.com/33454514/160064136-d83d597a-9c75-4479-9665-d6f64c056ee8.png)
