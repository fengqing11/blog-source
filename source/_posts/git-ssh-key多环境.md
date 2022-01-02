---
title: git ssh key多环境
date: 2022-01-02 23:19:39
tags: 
- git
- ssh key
---

在我们电脑使用 Git 时可能会遇到要使用 [GitLab](https://so.csdn.net/so/search?q=GitLab)、GitHub、Gitee 等不同代码托管平台的情况，之前由于项目工期紧、自己也比较懒，弄了几下多 SSH key 没有成功嫌麻烦就放弃了，改成直接使用 HTTPS 方式，这次特意抽点功夫研究一下，总结完贡献出来，不废话了直奔主题…

我使用的 Windows 系统装 Git 简单的一批，Git 官网下载个安装包一直下一步就好了，其他系统也不困难请自行安装，这里不再多说…

**1. 安装完 Git 先运行 Git Bash 看下安装成功了没有，然后设置下全局用户名和邮箱**

```bash
# 查看git版本
git --version

# 添加全局用户名和邮箱
git config --global user.name "yourname"
git config --global user.email "yourname@email.com"

# 查看当前的全局变量
git config -l

```

**2. 生成多个 ssh key，用几个平台生成几个，每条命令运行后再 2 下回车**

```bash
# 生成ssh key
ssh-keygen -t rsa -C "yourname1@email.com" -f ~/.ssh/id_rsa_gitlab
ssh-keygen -t rsa -C "yourname2@email.com" -f ~/.ssh/id_rsa_github
ssh-keygen -t rsa -C "yourname3@email.com" -f ~/.ssh/id_rsa_gitee

```

**3. 打开公钥~/.ssh/id_rsa_xxx.pub（C:\Users \ 你的用户名 \.ssh）将 ssh key 添加到 Git 平台的 SSH keys 列表中**

*   GitLab 设置：登陆后点右上角头像 -> Settings -> 左侧菜单 SSH Keys -> Title 随便写，Key 把公钥~/.ssh/id_rsa_gitlab.pub 里的 ssh key 粘贴上 -> 点 Add key 添加

*   [GitHub](https://so.csdn.net/so/search?q=GitHub) 设置：登陆后点右上角头像 -> Settings -> 左侧菜单 SSH and GPG keys -> 点右侧 New SSH key 按钮 -> Title 随便写，Key 把公钥~/.ssh/id_rsa_gitlab.pub 里的 ssh key 粘贴上 -> 点 Add SSH key 添加

*   [Gitee](https://so.csdn.net/so/search?q=Gitee) 设置：登陆后点右上角头像 -> 设置 -> 左侧菜单 SSH 公钥 -> （标题）随便写，（公钥）把公钥~/.ssh/id_rsa_gitlab.pub 里的 ssh key 粘贴上 -> 点确定添加


**4. 添加私钥**

```bash
# 先运行ssh公钥管理，再添加公钥，否则可能报错Could not open a connection to your authentication agent.
ssh-agent bash

# 添加私钥
ssh-add ~/.ssh/id_rsa_gitlab
ssh-add ~/.ssh/id_rsa_github
ssh-add ~/.ssh/id_rsa_gitee

# 查看私钥列表
ssh-add -l

```

**5. 添加配置文件**

```bash
# 添加配置文件
cat >> ~/.ssh/config <<EOF
# gitlab
Host mygitlab.com
HostName mygitlab.com
Port 22
User yourname
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_gitlab
 
# github
Host github.com
HostName github.com
User yourname
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_github

# gitee
Host gitee.com
HostName gitee.com
User yourname
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_gitee
EOF

```

**配置文件参数**  
**Host：** 指定 Host 别名，建议与 HostName 相同  
**HostName：** 指定 Git 服务器地址（域名或 IP）  
**Port：** 指定 Git 服务器端口（默认 80 端口可忽略此项）  
**User：** 指定用户名（自定义）  
**PreferredAuthentications：** 指定优先使用哪种方式验证，支持密码和密钥验证方式  
**IdentityFile：** 指定对应的私钥文件

**6. 测试（显示 successfully 即为成功）**

```bashc
# 测试
ssh -T git@mygitlab.com
ssh -T git@github.com
ssh -T git@gitee.com

```