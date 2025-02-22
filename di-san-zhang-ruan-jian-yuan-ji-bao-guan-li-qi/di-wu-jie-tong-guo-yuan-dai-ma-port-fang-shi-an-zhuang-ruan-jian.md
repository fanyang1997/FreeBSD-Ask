# 第五节 通过源代码 port 方式安装软件

## FreeBSD ports 基本用法（仅限 FreeBSD 13. 0以前，不含 13.0）

### 首先获取 portsnap

`# portsnap fetch extract`

### 使用 whereis 查询软件地址

如 `# whereis python`

输出 `python: /usr/ports/lang/python`

### 如何安装 python3：

```
# cd /usr/ports/lang/python
# make BATCH=yes clean
```

其中 BATCH=yes 的意思是使用默认配置

### 如何使用多线程下载：

`# pkg install axel #下载多线程下载工具#`

新建或者编辑 `# ee /etc/make.conf` 文件，写入以下两行：

```
FETCH_CMD=axel
FETCH_BEFORE_ARGS= -n 10 -a
FETCH_AFTER_ARGS=
DISABLE_SIZE=yes
```

### 进阶

如果不选择 `BATCH=yes` 的方法手动配置依赖：

看看 python 的 ports 在哪：

```
# whereis python
# python: /usr/ports/lang/python
```

安装python3：

`# cd /usr/ports/lang/python`

如何设置全部所需的依赖：

`# make config-recursive`

如何一次性下载所有需要的软件包：

`# make BATCH=yes fetch-recursive`

升级 ports collection

`# portsnap fetch extract`

ports 编译的软件也可以转换为 pkg 包

`# pkg create nginx`

### FreeBSD 包升级管理工具

首先更新 Ports 树

```
# portsnap fetch update
```

然后列出过时 Ports 组件
```
# pkg_version -l '<'
```
下边分别列出 2 种 FreeBSD 手册中提及的升级工具:

一、portupgrade

```
# cd /usr/ports/ports-mgmt/portupgrade && make install clean
# portupgrade -ai #自动升级所有软件
# portupgrade -R screen #升级单个软件
```

二、portmaster （推荐）

```
# cd /usr/ports/ports-mgmt/portmaster && make install clean
# portmaster -ai #自动升级所有软件
# portmaster screen #升级单个软件
# portmaster -a -m "BATCH=yes" #或者-D -G –no-confirm 都可以免除确认
```

## FreeBSD ports 多线程编译

Linux 如 gentoo上一般是直接 `-jx` 或者 `jx+1`, `x` 为核心数。

FreeBSD ports 多线程编译

```
FORCE_MAKE_JOBS=yes
MAKE_JOBS_NUMBER=4
```

写入 `/etc/make.conf` 没有就新建。

`4` 是处理器核心数，不知道就别改。
