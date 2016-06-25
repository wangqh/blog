title: 设置 GitHub SSH Keys
tags:
  - SSHKey
  - GitHub
date: 2016-06-01 11:40:45
categories: 自用笔记
---

让本地git项目与远程的github建立联系，需要设置SSH keys。
<!-- more -->
## 检查 SSH keys 的设置
首先我们需要检查电脑上现有的 ssh key:
```
$ cd ~/.ssh #检查本机的ssh密钥
```
> 如果提示：No such file or directory 说你机器上存在旧的 SSH key

## 生成新的 SSH Key:
首先用 ssh-keygen 生成一个密钥

    $ ssh-keygen -t rsa -C "邮箱名@youremail.com"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):<回车就行>
> 注意：此处的 `-C` 的是大写的 `C`

然后会提示输入密码

    Enter passphrase (empty for no passphrase):<输入密码>
    Enter same passphrase again:<输入密码>
> 在回车中会提示你输入一个密码，这个密码会在你提交项目时使用，如果为空的话提交项目时则不用输入。这个设置是防止别人往你的项目里提交内容。
> **注意：**输入密码的时候没有*字样的，你直接输入就可以了。

最后看到这样的界面，就成功设置 ssh key 了

    Your identification has been saved in
    /Users/your_user_directory/.ssh/id_rsa.
    Your public key has been saved in
    /Users/your_user_directory/.ssh/id_rsa.pub.
    The key fingerprint is: f3:ee:40:05:56:ab:fa:ac:14:72:8b:a2:44:ed:75:04 user_name@email.com

## 添加 SSH key 到 ssh-agent
如果检测到本机存在 ssh key， 就要重新生成一个 SSH key, 添加到 ssh-agent
1. 确保 ssh-agent 是启动的

        #启动 ssh-agent
        $ eval "$(ssh-agent -s)"
        Agent pid 59566
  > 上面是 Git Bash 的格式，如果用其它 terminal 工具，去掉 ``""`` 即可
2. 添加 SSH key

        $ ssh-add ~/.ssh/id_rsa

## 添加 SSH Key 到 GitHub 账号
在本机设置SSH Key之后，需要添加到GitHub上，以完成SSH连接接的设置。
1. 打开本地 C:\Documents and Settings\your_user_directory\.ssh\id_rsa.pub 文件。此文件里面内容为刚才生成人密钥。如果看不到这个文件，你需要设置显示隐藏文件。准确的复制这个文件的内容，才能保证设置的成功。
2. 登陆github系统。点击右上角的 Settings ??> SSH and GPG keys ??> New SSH key
3. 把你本地生成的密钥复制到里面（key文本框中），Title 可写中文标题，以区分其它绑定的机器， 点击 add SSH key 就 ok 了

## 测试 SSH 连接
可以输入下面的命令，看看设置是否成功，git@github.com的部分不要修改：

    $ ssh -T git@github.com

如果是下面的反馈：

    The authenticity of host 'github.com (207.97.227.239)' can't be established.
    RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
    Are you sure you want to continue connecting (yes/no)?

不要紧张，输入yes就好，然后会看到：

    Hi chanh! You've successfully authenticated, but GitHub does not provide shell access.

如果出现如下错误：

    Error: Permission denied (publickey).
> 可以参考官网解决方案：<https://help.github.com/articles/error-permission-denied-publickey/#platform-windows>

## 设置 git 全局用户信息
Git 会根据用户的名字和邮箱来记录提交。GitHub 也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的，名字必须是你的真名，而不是GitHub的昵称。

      $ git config --global user.name "username"//用户名
      $ git config --global user.email  "emailName@gmail.com"//填写自己的邮箱

