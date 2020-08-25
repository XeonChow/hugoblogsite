---
title: "用 Hugo 创建 Blog 站点并部署到 VPS"
date: 2020-05-03T19:15:17+08:00
draft: false
slug: deploy-hugo-on-vps
tags: ["blog", "web"]
indent: false
dropCap: false
comments: true
toc: true
---

之前的 Blog 站点托管在 GitHub 的 page 上，尽管免费且方便部署，但毕竟还是用的 `*.github.io` 的域名，看起来不够个性，加上 GitHub 在国内的访问比较抽风，对于不能科学上网的朋友，有时候会访问不了或很慢（说的好像这个 Blog 有很多人看一样），于是决定将 Blog 迁移到自己的一台 Bandwagon VPS 上，并且自己申请域名进行解析。

<!--more-->

Blog 的静态网站生成引擎从之前的 Hexo 换成了 Hugo，相比于 Hexo，Hugo 的生成速度很快（尽管在我这个文章很少的情况下两者相差并不大）。且 Hugo 基于 Go 语言，相比于基于 Node.js 的 Hexo，依赖环境少，安装方便。但 Hugo 诞生时间相对较短，目前主题数量不如 Hexo 多，也比较缺少 Hexo 上诸如 [NexT](https://theme-next.iissnan.com/) 这种成熟且文档齐全的主题，配置起来对我这样的小白有一定难度。

尽管关于如何用 Hugo 搭建 Blog 的教程网上也多如牛毛，作为一个非程序员的小白，自己把整个流程走一遍的过程也十分艰难，因此这篇文章将整个折腾的过程进行一个记录，也希望能对其他想要建站的朋友有个帮助。由于我并非程序员，很多步骤、命令只不过依葫芦画瓢，不求甚解。想要更详细的说明，只能靠大家自学相关  Linux、Web 等知识了。

## 准备工作

由于是将 Blog 部署在 VPS 上并且使用域名访问，必不可少的两样东西就是 VPS 和域名。我用来建站的 VPS 是 [Bandwagon](https://bwh1.net/) 年付 30 刀的一个 VPS，这原本是我用于科学上网工具部署的 VPS，但由于 cn2 gt 网络访问速度实在太慢，于是转用免费的谷歌云 VPS 科学上网（免费且速度飞快，谷歌真是慈善公司）。域名是在 [namesilo](https://www.namesilo.com/) 上购买的年付 1 刀的域名。VPS 和域名的购买网上有大把教程，选择很多，甚至可以不花一分钱（谷歌云+Freenom），这里就不赘述了。

VPS 我选装的系统是 Debian 9，因此以下在 VPS 上的命令都是基于 Debian 的（Ubuntu 和 Debian 类似，CentOS 的包管理工具不同，其余命令也一样，毕竟都是 Linux）。需要知道 VPS的 IP、SSH 端口（一般默认是22，Bandwagon 初始给的是一个随机的高位端口）和 root 密码，这些在 VPS 服务商的管理后台都能获取。

域名需要解析到 VPS 的 IP 上，在域名服务商的 DNS 服务里添加一条 A 记录，指向 VPS 的 IP 即可，也可以用 CloudFlare 这样的专业域名解析服务，方便添加 CDN。

## 基本思路

Blog 网站整个的搭建流程大致如下：

1. Windows 本地用 Hugo 进行网站的配置和生成；
2. 使用 Git 工具将 Windows 本地生成的网站文件夹远程部署到 VPS 服务器上；
3. VPS 上使用 [Nginx](https://www.nginx.com/) 作为 Web 网页服务器。

另外，为了让用户在浏览网页时，浏览器不会提示「不安全」，需要为网页申请 SSL 证书并强制开启 HTTPS。我选用的证书申请工具是 [acme](https://github.com/acmesh-official)。

## Windows 本地 Hugo 设置

### 安装 Hugo

从 Hugo 官方 GitHub 的 [release](https://github.com/gohugoio/hugo/releases) 页面下载 Hugo 的 Windows 可执行文件，建议下载 extended 版本，能支持更多的特性。下载完成后解压，会获得一个 `.exe` 文件，将其重命名为 `hugo.exe`，并且新建一个文件夹，比如路径为 `D:\Hugo\bin` ，将 `hugo.exe` 放入这个文件夹，并将 `D:\Hugo\bin`  文件夹路径添加到 Windows 的 PATH 路径中，方法如下：

右键「我的电脑」&rarr;「高级系统设置」&rarr;「环境变量」&rarr;「系统变量」，找到 Path，编辑，在最后加入`D:\Hugo\bin;`。

在任意路径新建一个 Blog 站点文件夹，比如就使用 `D:\Hugo`，打开这个文件夹，`shift`+鼠标右键选择「在此处打开命令窗口」，输入`hugo version`，如果能显示 Hugo 的版本信息，则添加 Path 路径成功。

### 新建站点

在 `D:\Hugo` 下新建一个文件夹 `D:\Hugo\Sites` 作为站点文件夹，进入这个文件夹，打开命令窗口，输入 `hugo new site blog`，即新建了一个名为 `blog` 的站点，并生成了`D:\Hugo\Sites\blog` 文件夹。

### 添加主题

与 Hexo 不同，Hugo 默认没有主题文件夹，需要自己添加主题文件夹。[这个页面](https://themes.gohugo.io/)列出了 Hugo 的所有可用主题，可以上去挑选一个喜欢的主题。每个主题都在 GitHub 上有相应的项目页面，可以手动将主题文件夹下载下来，也可以在 Windows 本地安装 git 后用 `git clone` 命令拉取下来。推荐使用后一种方法，更加方便升级主题，正好后续也需要 git 进行网站文件的远程推送。

以我选用的「[MemE](https://github.com/reuixiy/hugo-theme-meme)」主题为例

#### 手动安装

打开主题的 GitHub 页面（https://github.com/reuixiy/hugo-theme-meme），点击「Clone or download」按钮，选择「Download ZIP」。将 zip 文件解压后得到的文件夹名字修改为 `meme`，并移动到 `D:\Hugo\Sites\blog\themes` 文件夹中。将 `D:\Hugo\Sites\blog\themes\meme\exampleSite` 文件夹里的 `config.toml` 配置文件移动到 `D:\Hugo\Sites\blog` 并覆盖原配置文件。

#### Git 方式安装

Windows 安装 Git，点击[这里](https://git-scm.com/download/win)下载 Git 的安装包，一路默认安装即可，安装完成后在任意文件路径下鼠标右键会出现 `Git Bash here`。在 ` D:\Hugo\Sites\blog` 下右键打开 `Git Bash here`，出现 Git 的命令窗口，输入以下命令：

```bash
git init
git submodule add --depth 1 https://github.com/reuixiy/hugo-theme-meme.git themes/meme
```

即安装主题完成。如果要升级主题，输入以下命令：

```bash
git submodule update --rebase --remote
```

### 配置站点

站点的配置主要靠修改 `config.toml` 配置文件，详细的配置方法因不同主题作者的设定而异，详细可以参照主题作者的文档，我使用的「[MemE](https://github.com/reuixiy/hugo-theme-meme)」主题配置方法可参考作者的[这篇文章](https://io-oi.me/tech/documentation-of-hugo-theme-meme/)。

### 常用的 Hugo 命令

Hugo 有很多命令，详细的命令教程可以参考官方文档或者自行搜索，以下列出常用的几个命令。

#### 新建文章

```bash
hugo new "posts/hello-world.md"
hugo new "about/_index.md"
```

#### 在本地运行 Hugo 生成预览页面

```bash
hugo server -D
```

这个命令会生成预览页面，在本地浏览器访问：localhost:1313，就可以看到 blog 的预览页面了。

#### 生成站点静态网页文件

```bash
hugo
```

这个命令会生成 blog 的网页文件，放在 `D:\Hugo\Sites\blog\public` 文件夹中，我们需要上传到服务器上的就是这个文件夹中的文件。

## Windows 本地 Git 设置

在 Windows 本地安装 Git 的方法参考前文。

### 新建 Git 用户

安装后需要在本地新建一个 git 用户，设置用户名和邮箱（仅作用户识别使用，可填写任意邮箱地址）。在任意路径下打开 Git Bash 命令窗口，输入以下指令：

```bash
git config --global user.name "name"
git config --global user.email "emailaddress"
```

其中 `"name"` 和 `"emailaddress"` 字段填入自己的用户名和邮件地址（均可任意填写）。

为了后续远程推送文件方便，在本地生成 ssh 密钥备用：

```bash
ssh-keygen -t rsa -C "emailaddress"
```

bash 会要求输入 Windows 用户的密码，无密码直接 enter。从而在 ` /c/Users/Xeon/.ssh/` 路径下会生产 ssh 密钥，有公钥 `id_rsa.pub` 和密钥 `id_rsa` 两个文件。

### 新建本地仓库

在 Hugo 生成的静态网页文件夹，即  `D:\Hugo\Sites\blog\public` 文件夹中打开 Git Bash 窗口，输入以下命令：

```bash
git init
git add --all
git commit -m "Description"
```

上面三个命令的意思分别是：

1. 创建 git 仓库；
2. 添加全部文件；
3. 提交文件，其中 `-m` 表示本次更新的描述，描述内容为 `"Description"` 中的内容，按情况自行填写。

从而本地的 Git 仓库搭建完成，远程提交静态网页文件之前需要现在 VPS 上进行设置。

## VPS Git 设置

通过 SSH 工具用 root 账户连接自己的 VPS，我使用的是 [FinalShell](http://www.hostbuf.com/downloads/finalshell_install.exe)，集成了简单的 ftp 工具，方便新手修改配置文件。

### 安装 Git

```bash
apt-get update
apt-get install git-core
```

### 添加 git 用户

为了安全，新建一个 git 用户进行 git 相关的操作

```bash
useradd git
```

给 git 用户添加 sudo 权限，修改以下文件

```bash
vi /etc/sudoers
```

在 `# User privilege specification` 配置字段下添加

```text
git	ALL=(ALL:ALL) ALL
```

从而使得 git 用户拥有 sudo 权限

给 `sudoer` 文件添加权限

```bash
chmod 740 /etc/sudoers
```

编辑 `/etc/passwd` 文件

```bash
vi /etc/passwd
```

将最后一行 git 用户的 `/bin/sh` 改为 `/bin/bash`，是 git 用户也拥有 bash 脚本解释器。

创建 git 用户密码

```bash
passwd git
```

创建 git 的用户目录

 ```bash
mkdir /home/git
 ```

给 git 的用户目录添加 git 的用户权限

```bash
chown -R git:git /home/git
```

### 新建 VPS 端的 git 仓库

在 VPS 上新建一个 git 仓库作为远程仓库。

切换到 git 用户并转到 git 用户文件夹

```bash
su git
cd /home/git
```

新建 `blog.git` 文件夹并进入

```bash
mkdir blog.git
cd blog.git
```

初始化仓库

```bash
git init --bare
```

在 `/home/git` 路径下创建存放 SSH 公钥的文件夹 `.ssh` 并进入，在该文件夹下新建公钥文件并编辑

```bash
cd /home/git
mkdir .ssh
cd .ssh
vi authorized_keys
```

将前文 Windows 本地  ` /c/Users/Xeon/.ssh/`  位置生成的 SSH 公钥文件  `id_rsa.pub` 内容复制粘贴到 `authorized_keys` 文件中并保存。

### 新建网页文件存放目录

退出 git 用户，新建  `/var/www`  路径，并转到 `/var/www` 文件夹

```bash
exit
mkdir -p /var/www
cd /var/www
```

在该路径下创建 `blog` 文件夹

```bash
mkdir blog
```

修改 `/var/www/blog` 文件夹权限

```bash
chown -R git:git /var/www/blog/
```

### 设置 git 钩子脚本

创建一个 git 钩子脚本，主要目的是当 Windows 本地的文件被远程推送到 VPS 的 `blog.git` 仓库时，触发脚本自动将 `blog.git` 里的内容复制到网页文件存放的目录  `/var/www/blog`  中。

切换为 git 用户和文件夹

```bash
su git
cd
```

转到 `blog.git/hooks/` 文件夹，新建 `post-receive` 文件

```bash
cd blog.git/hooks/
vi post-receive
```

写入以下内容

```text
#!/bin/bash

GIT_REPO=/home/git/blog.git
TMP_DIR_CLONE=/tmp/blog
PUBLIC_WWW=/var/www/blog
rm -rf ${TMP_DIR_CLONE}
git clone $GIT_REPO $TMP_DIR_CLONE
rm -rf $PUBLIC_WWW/*
cp -rf $TMP_DIR_CLONE/* $PUBLIC_WWW
```

给 `post-receive` 脚本文件添加可执行权限

```bash
chmod +x post-receive
```

## VPS 证书申请

使用 acme 申请 SSL 证书，在此之前先要做好域名的解析。我使用 CloudFlare 进行域名解析，因此可以直接使用 CloudFlare 的 API 进行申请，比较方便，若使用其他解析服务，需要手动添加两条 txt 解析，可自行搜索教程。以下步骤均在 root 用户下。

### 安装 acme

新建一个文件夹 `/home/tls` 存放证书和密钥

```bash
rm -rf /home/tls && mkdir -p /home/tls
```

安装 acme

```bash
curl https://get.acme.sh | sh
source ~/.bashrc
```

### 导入 CloudFlare 的 API

在 CloudFlare 的个人账户页面，找到 API Tokens-Global API Key，点击 view 查看，将 API Key 复制保持下来。在 VPS 上输入以下命令：

```bash
export CF_Key="Your_API_Key"
export CF_Email="Your_CloudFlare_Email"
```

其中 `CF_Key` 和 `CF_Email` 填入自己的 API Key 和 CloudFlare 账户邮箱。

### 申请证书

```bash
~/.acme.sh/acme.sh --issue --dns dns_cf -d your_domain.com -d *.your_domain.com -k ec-256
```

以上命令中的 `your_domain.com` 替换为自己的域名，这个命令会生成 `*.your_domain.com` 泛域名的 SSL 证书，即  `your_domain.com` 下所有的二级、三级域名都会受到证书保护。所申请到的证书文件存放在 `~/.acme.sh/your_domain.com_ecc` 文件夹下，`~` 代表当前用户目录，若当前是 root 用户，则 `~` 代表 `/root`。

### 安装证书并启用自动更新

将证书安装到 `/home/tls` 路径下

```bash
~/.acme.sh/acme.sh --installcert -d your_domain.com -d *.your_domain.com --fullchainpath /home/tls/your_domain.com.crt --keypath /home/tls/your_domain.com.key --ecc
```

同样将以上命令中的 `your_domain.com` 替换为自己的域名，此时在 `/home/tls` 路径下会生成证书文件 `your_domain.com.crt` 和密钥文件 `your_domain.com.key`。输入以下命令设置证书到期自动更新：

```bash
acme.sh --upgrade --auto-upgrade
```

## VPS 上 Nginx 设置

安装 Nginx

```bash
apt-get install nginx -y
```

修改 Nginx 配置文件

```bash
vi /etc/nginx/sites-enabled/default
```

用以下内容覆盖原配置

```
server {
  # 当 http 协议被请求时，统一转发到 https 协议上
  listen 80;
  listen [::]:80; # IPV6 协议
  server_name your_domain.com;
  rewrite ^(.*)$ https://$host$1 permanent;
}

server {
  listen 443 ssl;
  listen [::]:443 ssl;
  ssl_certificate /home/tls/your_domain.com.crt; # 证书的绝对路径
  ssl_certificate_key /home/tls/your_domain.com.key; # 证书密钥的绝对路径

  server_name your_domain.com;

  location / {
    root /var/www/blog/; #站点文件的路径
    index index.html;
  }
  
  # 证书验证 location
  location /.well-known/acme-challenge/ {
    root /home/tls/; # 证书文件夹的路径
    log_not_found off;
  }
}
```

同样，将 `your_domain.com` 替换为自己的域名。以上 Nginx 配置文件的大概含义见注释，详细含义见 Nginx 官方文档（说实话这一块我也没搞懂，需要有一定的 Web 知识，我完全依葫芦画瓢）。

修改配置文件后需要重启 Nginx

```bash
systemctl restart nginx
```

## Windows 本地推送

首先在 Windows 本地测试一下能否以 git 用户 ssh 登陆到 VPS

```bash
ssh -p "port" git@VPS_ip
```

若提示 ` WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! `，可将 Windows 本地 ` /c/Users/Xeon/.ssh/` 下的 `known_hosts` 文件删除。

### 本地推送

回到已经创建为本地仓库的  `D:\Hugo\Sites\blog\public` 文件夹中打开 Git Bash 窗口，输入以下命令添加远程仓库：

```bash
git remote add origin git@your_VPS_IP:/home/git/blog.git
git remote set-url origin ssh://git@your_VPS_IP:SSH_Port/home/git/blog.git
```

以上命令中的 `your_VPS_IP` 为你 VPS 的 IP，`SSH_Port` 为你的 VPS SSH 登陆端口（一般默认为22）。

以上命令仅在初次推送前设置，以后无需设置。

推送本地仓库到 VPS：

```bash
git push origin master
```

不出意外，在 VPS 的站点文件路径 `/var/www/blog/` 下应该有了和 Windows 本地  `D:\Hugo\Sites\blog\public` 文件夹中一样的内容。此时用浏览器访问域名 `your_domain.com`，即可打开自己的 blog 站点，并且自动开启了 HTTPS 加密。

以后 Windows 本地更新内容后，只需在 `D:\Hugo\Sites\blog\public` 路径下打开 Git Bash 输入以下命令：

```bash
git init
git add --all
git commit -m "description"
git push origin master
```

即可将更新内容推送到 VPS。

