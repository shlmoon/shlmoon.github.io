---
layout: post
title: 使用git做服务器端代码的部署
date: 2017-09-23 08:08:00
tags: git-deploy
---


### server 准备工作：
  
  这些工作都在root或有管理权限的帐号下进行，下面以root为用户，切换到其他用户的时候会提示

<!-- more -->

  确保安装了git

  为了安全起见，新建一个专门用于代码部署的无特权用户
    ```
    useradd -m apps
    passwd apps //设置该用户的密码，也可根据喜好配置成免密码登陆

    ```

  新建一个目录作为要部署代码的根目录。
  ```
  mkdir -p /apps/web/www/git
  mkdir -p /apps/web/www/html

  ```
  这里建立git目录和html目录，是为了将代码和.git分离出来。
  在nginx的root中，直接设置`/apps/web/www/html`就不用担心.git之类的问题了。


  将这个目录的属主和属组都改为上面新建的用户apps
    ```
    cd /apps/web/www
    chown apps:apps git
    ```

  切换到部署代码的专用用户`su apps`

  进入项目根目录，初始化为git仓库
    ```
    cd /apps/web/www/git
    git init
    ```

  注意：这里千万不要用`git --bare init`，一旦使用后续的`git checkout -f`是检不出来代码的。


  【重要】让仓库接受代码提交
    ```
    git config receive.denyCurrentBranch ignore
    git config core.worktree /apps/web/www/html
    git config --bool receive.denyNonFastForwards false  //禁止强制推送
    ```


至此，一个空的git仓库就在服务器上建好了，仓库的地址为：
`ssh://apps@ip/apps/web/www/git/.git`



### local 准备工作：

   通过`git clone`或`git pull`从github或gitlab仓库上将代码拉取到本地


   将服务器添加到远程仓库列表，使用名字来区分不同的服务器，如服务器可以叫做server

   `git remote add server ssh://apps@ip/apps/web/www/git/.git`


   将本地代码提交到测试服务器上面

   `git push server master`



### server 操作：

   更新server，git仓库状态并检出文件

   ```
   cd /apps/web/html/git
   git update-server-info
   git checkout -f
   ```


   检查是不是将文件更新进来了`git status`


   设置服务器端更新hook：post-update
   `cd .git/hooks`
   新建 post-receive 或将 post-receive.sample 重命名为post-receive`touch post-receive`或则
   ```
   mv post-receive.sample post-receive
   vim post-receive
   ```
   将如下内容复制到文件中：
   ```
   #!/bin/sh
   unset GIT_DIR
   cd ..
   git checkout -f
   ```


注: 第3步的操作将post-receive 替换为 post-update也可以，
不过需要先将post-update中的`exec git update-server-info`这一行删掉


### 后续代码的更新：
   github 有更新的时候pull更新local

   然后本地先push到test-server进行测试

   然后在本地测试，测试通过之后push到prd上线

   代码的回滚：
   * 服务器端回滚：推荐`git reset --hard HEAD^`
   * 本地仓库回滚：无需登陆服务器即可实现代码回滚，`git reset HEAD^`保留代码回滚，

   使用`git push remote_name local_branch_name -f` 强制推送



### 使用过程中需要注意的问题：

   * 需要约定好git不能更新的操作要怎么处理，比如新增数据库的字段，新安装必要的扩展等
   * 充分利用好git的hook功能，比如pre-commit可用于提交代码前进行单元测试等，但是hook做的操作要尽量简单


### git其他操作

存储当前的版本号，如：`git log –pretty -1 | awk ‘/^commit/{print $2}’`



