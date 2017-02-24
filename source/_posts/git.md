---
layout: post
title: git 
tags: 笔记
---

## 一. 使用前准备
  设置SSH Key
  
  ```swift
  ssh-keygen -t rsa -C "youremail@example.com"
  ```
  >过程需要创建密码。
  >id\_rsa 文件是私有秘钥，id\_rsa.pud是公开秘钥。

 查看id_rsa.pud文件内容
 
 ```swift
 cat ~/ssh/id_rsa.pud
 ```

 测试是否成功

 ```swift
 ssh -T git@github.com
 ```
 
 ---
## 二. 本地操作
 
 
 ```swift
 git status
 git add .
 git commit -m "change"
 (git commit -am "注释" //代替上面两步)
 git pull
 git push
 ```
 
 ```swift
 git log
 git log --pretty=short
 git log -p //显示文件的改动情况
 git log -p 文件名 //显示某一文件的改动
 git log --graph //以图表形式查看分支
 ```
 
 ```swift
 git diff //查看更改前后的差别
 git diff HEAD //查看本次提交和上次提交之间的差别
 ```
 
 ```swift
 git branch
 git branch -a //查看当前分支的信息，参数-a可以同时显示本地仓库和远程仓库的分支信息
 git checkout -b //创建、切换分支 
 git checkout -b feature-D origin/feature-D //以名为origin的仓库（这里指GitHub端的远程仓库）的feature-D分支为来源，在本地仓库中创建feature-D分支
 git checkout - //切回上一个分支
 git branch  //列出目前有多少branch
 git branch new-branch //产生新的branch (名称: new-branch), 若没有特别指定, 会由目前所在的branch / master 直接复制一份.
 git branch new-branch master //由master 产生新的branch(new-branch)
 git branch new-branch v1 //由tag(v1) 产生新的branch(new-branch)
 git branch -d new-branch //删除new-branch
 git branch -D new-branch //强制删除new-branch
 git checkout -b new-branch test //产生新的branch, 并同时切换过去new-branch
//与remote repository 有关
 git branch -r //列出所有Repository branch
 git branch -a //列出所有branch
 ```

 ```swift
 git merge --no-ff 其他分支名// 合并分支 ，参数--no-ff用于启动编辑器，录入合并提交的信息。
 ```

 ```swift
 git reset --hard 对应版本目标时间点的哈希值//回溯历史版本
 ```

 ```swift
 git reflog //查看当前仓库的操作日志，一般用于查询左右操作对应的哈希值以用于推进历史版本
 ```
 
 ```swift
 git commit --amend // 修改提交信息
 git rebase -i //压缩历史
 ```

 >错字漏字等失误称作“typo”，所以修改后的提交信息可记为“Fix typo”

---
## 三. 远程仓库操作

#### 1. 推送至远程仓库

 ```swift
 git remote add origin(/*会自动将远程仓库名设置为此*/) 远程仓库地址//添加远程仓库，将某一远程仓库设置为本地仓库的远程仓库
 ```
 
 ```swift
 git push //推送至远程仓库
 例：git push -u origin master 
 /*
 1. 将当前分支的内容推送到远程仓库origin的master分支；
 2. -u参数可以在推送的同时，将origin仓库的master分支设置为本地仓库当前当前分支的upstream（上游）；
    添加了这个参数，将来运行git pull命令从远程仓库获取内容时，本地仓库的这个分支就可以直接从origin的master分支获取内容，省去了另外添加参数的麻烦。
 */
 例：git checkout -b feature-D //本地创建feature-D分支 
   git push -u origin feature-D //将本地创建的feature-D分支推送到远程的同名分支下，其会默认在远程创建一个feature-D分支。
 ```
#### 2. 从远程仓库获取

```swift
git clone //获取远程仓库
```

```swift
git pull //获取最新的远程仓库分支
```

---
## 四. 资料

1. [Pro Git 零基础Git学习资料](http://git-scm.com/book/zh/v1)
2. [LearnGitBranching 学习Git基本操作的网站](http://pcottle.github.io/learnGitBranching)
3. [tryGit 可以Web上一边操作一边学习](https://try.github.io/)





