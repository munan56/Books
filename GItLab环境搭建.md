# 基于 CentOS 7 搭建 GitLab

## 准备环境

> 任务时间：5min ~ 10min

在正式开始安装之前，先更新软件包并打开相关服务的权限。

### 更新软件包

```
yum update -y

```

### 安装 sshd

安装 sshd：

```
yum install -y curl policycoreutils-python openssh-server

```

启用并启动 sshd：

```
systemctl enable sshd
systemctl start sshd

```

### 配置防火墙

打开 */etc/sysctl.conf* 文件，在文件最后添加新的一行并按 `Ctrl + S` 保存：

```
net.ipv4.ip_forward = 1

```

启用并启动防火墙：

```
systemctl enable firewalld
systemctl start firewalld

```

放通 HTTP：

```
firewall-cmd --permanent --add-service=http

```

重启防火墙：

```
systemctl reload firewalld

```

在实际使用中，可以使用 `systemctl status firewalld` 命令查看防火墙的状态。

### 安装 postfix

GitLab 需要使用 postfix 来发送邮件。当然，也可以使用 SMTP 服务器，具体步骤请参考 *官方教程*。

安装：

```
yum install -y postfix

```

打开 */etc/postfix/main.cf* 文件，在第 119 行附近找到 `inet_protocols = all`，将 `all` 改为 `ipv4` 并按 `Ctrl + S` 保存：

```
inet_protocols = ipv4

```

启用并启动 postfix：

```
systemctl enable postfix 
systemctl start postfix

```

### 配置 swap 交换分区

由于 GitLab 较为消耗资源，我们需要先创建交换分区，以降低物理内存的压力。
在实际生产环境中，如果服务器配置够高，则不必配置交换分区。

新建 2 GB 大小的交换分区：

```
dd if=/dev/zero of=/root/swapfile bs=1M count=2048

```

格式化为交换分区文件并启用：

```
mkswap /root/swapfile
swapon /root/swapfile

```

添加自启用。打开 */etc/fstab* 文件，在文件最后添加新的一行并按 `Ctrl + S` 保存：

```
/root/swapfile swap swap defaults 0 0

```

## 安装 GitLab

> 任务时间：10min ~ 15min

### 将软件源修改为国内源

由于网络环境的原因，将 repo 源修改为[[清华大学](about:blank#stage-2-step-1-tuna)]。

在 `/etc/yum.repos.d` 目录下新建 *gitlab-ce.repo* 文件并保存。内容如下：

##### 示例代码：/etc/yum.repos.d/gitlab-ce.repo

```
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1

```

> <https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/>

### 安装 GitLab

刚才修改过了 yum 源，因此先重新生成缓存：
（此步骤执行时间较长，一般需要 3~5 分钟左右，请耐心等待）

```
yum makecache

```

安装 GitLab：
（此步骤执行时间较长，一般需要 3~5 分钟左右，请耐心等待）

```
yum install -y gitlab-ce

```

## 初始化 GitLab

> 任务时间：10min ~ 15min

### 配置 GitLab 的域名（非必需）

打开 */etc/gitlab/gitlab.rb* 文件，在第 13 行附近找到 `external_url 'http://gitlab.example.com'`，将单引号中的内容改为自己的域名（带上协议头，末尾无斜杠），并按 `Ctrl + S` 保存。

例如：

```
external_url 'http://work.myteam.com'

```

记得将域名通过 A 记录解析到 `<您的 CVM IP 地址>` 哦。

### 初始化 GitLab

*特别重要！*

使用如下命令初始化 GitLab：
（此步骤执行时间较长，一般需要 5~10 分钟左右，请耐心等待）

```
sudo gitlab-ctl reconfigure

```

## GitLab 安装已完成

> 任务时间：时间未知

### 开始使用吧！

至此，我们已经成功地在 CentOS 7 上搭建了 GitLab。 现在可以在这里（[http://<您的 CVM IP 地址>/](http://xn--%3C%20cvm%20ip%20%3E-yp49ackh32qjw5g/)）访问 GitLab 了。

- 在实际生产中，建议您使用 2 核 4 GB 或更高配置的 CVM。*点击这里* 可以查看 GitLab 官方推荐的配置和可承载人数对应表。
- 再次提醒您，定期执行 `yum update -y` 以保持各软件包的最新状态。

谢谢！