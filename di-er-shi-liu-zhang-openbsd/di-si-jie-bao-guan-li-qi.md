# 第四节 包管理器

同其它 Unix-like 系统一样，OpenBSD 的软件安装主要有两种方式：采用官方预编译好的二进制包，以及通过源代码自己打包安装。这里我们推荐第一种方式安装。

## 二进制包

我们推荐以二进制包的方式来安装软件，以火狐浏览器为例：

- 安装软件 `pkg_add firefox`

- 删除软件 `pkg_delete firefox`

- 查询软件 `pkg_info -Q firefox`

- 升级软件 `pkg_add -iu firefox`

此外，全局的命令有：升级所有软件 `pkg_add -iu`； 删除所有软件包缓冲 `pkg_delete -a`。

## ports 

[查询网站](https://openports.se/)

OpenBSD 的 ports 安装比较复杂，这里只作一番简单介绍，学有余力的网友可进一步查看[手册](https://www.openbsd.org/faq/ports/ports.html)，获取更详细的信息。

OpenBSD 对应多个系统版本(release、stable 以及 current)，各版本间的 ports 并不通用。

## pkgsrc

pkgsrc 为 NetBSD 的软件包管理系统，不过它宣称同样支持 Linux 和 其它 BSD 系统。pkgsrc 在打包数量上似乎多过 OpenBSD 的官方包，不过唯一要担心的是 pkgsrc 与 OpenBSD 能否完美契合。以下内容仅供感兴趣的网友尝试，不能保证没有意外，我们也不推荐以 pkgsrc 为主力包管理系统。

```
$ cd ~/
$ ftp https://cdn.NetBSD.org/pub/pkgsrc/pkgsrc-2022Q1/pkgsrc.tar.gz
$ tar -xzf pkgsrc.tar.gz
$ cd pkgsrc/bootstrap
$ ./bootstrap --unprivileged
```

然后是添加路径 `~/pkg/bin` 到路径环境变量中。pkgsrc 树位于 `~/pkgsrc/` 中，其工作的所有相关文件均在`~/pkg/`中。

我们就可以在`~/pkgsrc/`中搜索软件来安装程序，之后运行`bmake install`。如在`~/pkgsrc/chat/irssi/`安装 IRC 客户端` IRSSI`。
