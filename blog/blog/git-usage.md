---
title: git使用的问题
date: 2017-01-12 01:04:41
type: post
tag: git
meta:
  - name: description
    content: git 使用
  - name: keywords
    content: git
---

#### git的三个区, 四个状态
- 三个区
工作区 暂存区 版本库
- 四个状态
未修改`unmodified`, 修改`modified`,  暂存`staged`,  未加入git`untracked`

<!-- more -->

#### git撤销暂存区与工作区
- 撤销工作区
`git checkout <file>`
- 撤销暂存区
`git reset <file>`

#### git已经commit, 但是发现有些东西没改, 怎么撤销最近一次的commit,修改后重新commit
    
- `reset` **慎用**
    1. `git reset --soft HEAD^1` `--soft`表示仅仅是`HEAD`指向上个版本, 但是
    缓存区跟工作区不会有任何变化
    2. 再次修改
    3. `git add ` 注意`-A`,`.`,`-u`参数的区别
    4. `git commit -c ORIG_HEAD` 此时可以修改上一次的commit信息
- `git commit --amend`
    1. 做一些修改并提交到暂存区
    2. `git commit --amend`  此时可以修改上一次的提交信息, 最终只会产生一个提交

#### push的时候不希望产生很零碎的commit信息

*假设现在在dev分支*

1. `git checkout -b <temp>`  首先切出一个新的分支, 在新的分支操作
2. 做几次修改, 做几次提交
3. `git checkout dev` 切回原来的分支
4. `git merge --squash temp` `--squash`参数表示挤压的意思,
    它会将`temp`这个分支的所有 commit 都放进`dev`的暂存区中,
    此时`dev`分支就可以进行`commit`, 只会产生一次 commit 信息

#### 很多时候代码被改的乱七八糟, 就需要回到之前的版本, 检查一下之前版本的代码, 然后再回来

1. `git checkout <之前的版本号>` 这个命令会让`HEAD`脱离
2. 这种情况下, 如果想提交修改, 可以创建出新的一个分支来实现:
3. `git checkout -b <branch name>`
4. 此时, 如果想回到之前未`checkout`版本号时的`HEAD`版本, 只需要检出之前所在的分支:
    `git checkout <previous branch name>`

#### 冲突的解决

- `git merge`
    *进入一个场景*
    1. 基于远程分支`origin`, 创建一个新的分支, 叫做`mywork`
        `git checkout -b mywork <remote>/origin`

    2. 做一些修改, 产生两次`commit`

    3. 但是在此同时, 也有人在`origin`分支上做了修改并且`push`了, 
        这意味着`origin`和`mywork`分支各自前进了, 它们产生了分叉
        
    4. 这时候, 可以`git pull`把`origin`分支拉下来, 和自己的代码修改合并,
        并解决冲突, 然后提交。这就产生了一次新的`commit`。上述过程就是`3 commits`
        ![](https://blog-1252181333.cos.ap-shanghai.myqcloud.com/blog/3-way-merge.png)
        可以看到分支的修改记录, 并且额外产生了一次新的提交
        当然, 如果`origin`分支没有修改的话, 那就是`Fast Forward`
- `git rebase`
    *当然, 如果你希望`mywork`分支的记录看起来像没有经过任何合并一样*
    
    ```
        git checkout mywork
        git rebase origin
    ```
    - 它会把`mywork`分支的`commit`都取消掉, 并且临时保存为`patch`, 放在`.git/rebase`目录中,然后把`mywork`分支更新到与`origin`同步, 最后把这些`patch`应用到`mywork`分支上.
    - 当然如果有冲突的话, 解决冲突后, `git add .`, `git rebase --continue`就可以继续了.
    - 当`mywork`分支更新后, 它就会指向新的`commit`, 而那些老的会被丢弃, `git gc`就可以删除这些丢弃的`commit`
    - `pick`, `squash`, `edit`这些参数的用法就不多阐述了
    - 你可以随时用`git rebase --abort`来终止`rebase`操作, 并且`mywork`分支会回到`rebase`开始前的状态

    **`rebase`处理之后, 整个是一个链式的记录, 不会像`merge`那样出现分叉**

---
[pro git 中文](https://git-scm.com/book/zh/v2)
[pro git en](https://git-scm.com/book/en/v2)

