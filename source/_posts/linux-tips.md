---
layout: post
title: Linux小操作集锦
date: 2017-09-24 09:12:00
tags: linux-tips
---

### root密码修改

```
chattr -i /etc/passwd
chattr -i /etc/shadow
passwd root
chattr +i /etc/passwd
chattr +i /etc/shadow
```
<!-- more -->
1出现permission问题时，执行`mount / -rw -o remount`即可完成root密码修改。

当然这里面的操作都是在recovery(root)模式下进行的。


### linux ssh不带密码
`ssh apps@ip`一般出现：
```
apps@ip: Permission denied (publickey,gssapi-keyex,gssapi-with-mic)
```

这种信息。直接的原因是server上apps用户的 authorized_keys权限不够。

解决方法：在server 上，用apps 执行：`chmod 644 ~/.ssh/authorized_keys`

另附：
linux清除用户密码：root下执行： `passwd -d apps`


### linux oh-my-zsh
server中，以root方式login，按照上面的链接无法生效。
只是因为没有`/root/.zshrc`文件，最直接的解决方法：
`cp /root/.oh-my-zsh/templates/zshrc.zsh-template /root/.zshrc`

ps：zsh切换到`bash-exec bash`


### linux 查看本机外网IP
1. `curl cip.cc`
2. `curl ifconfig.me`


### mysql data
mysql 在用source导入数据时，中文乱码。解决方案：
`mysql -uroot -p --default-character-set=utf8`
