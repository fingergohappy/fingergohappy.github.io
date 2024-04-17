---
title: ganesha编译以及使用vscode开发ganesha
toc: true
date: 2024-04-16 16:48:56
tags:
    - ganesha
    - vscode
categories: ganesha
cover: https://source.unsplash.com/a-man-in-a-space-suit-standing-in-front-of-a-space-station-nOr0PFuQ8Qc/1200x628
---

# 简介

如何手动编译`ganesha`以及如何使用`vscode`远程开发`ganesha`源码

`FSAL`层使用`cephfs`

<!--more-->

# 环境

## 远程环境

服务器: centos 7
cmake版本: 3.12.3
vscode版本: 1.88.0

# 配置编译环境

## 下载源代码

[ganesha地址](https://github.com/nfs-ganesha/nfs-ganesha)

```bash
git clone https://github.com/nfs-ganesha/nfs-ganesha
```

编译的同时还需要[ntirpc](https://github.com/nfs-ganesha/ntirpc)这个包里面的头文件,所以还需要初始化子模块

```bash
git submodule --recursive --init
```

## 安装cmake

由于`centos7`自带的`cmake`版本过低,需要手动安装较新的版本

### Download CMake from: https://cmake.org/download/

```bash
wget https://cmake.org/files/v3.12/cmake-3.12.3.tar.gz
```

### Compile from source and install

```bash
tar zxvf cmake-3.*
cd cmake-3.*
./bootstrap --prefix=/usr/local
make -j$(nproc)
make install
```

### Validate installation

```bash
cmake --version
cmake version *.*.*
CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

## 安装ganesha编译所需依赖包


```bash
 # Common things 
 # libcpefs-devel libcephfs2 这两个是fsal层使用ceph时需要用到的包，libcephfs2这个包如果没有的话，cephfs的一些功能可能无法使用
 yum install gcc git cmake autoconf libtool bison flex libacl-devel libcephfs-devel libtirpc rpcbind
 # More nfs-ganesha specific
 yum install libgssglue-devel openssl-devel nfs-utils-lib-devel doxygen redhat-lsb gcc-c++
```

# 编译

在`ganesha`源码的同级文件夹新建一个`build`目录
`ganesha`是使用`cmake`来控制编译`FSAL`哪个模块的,由于我们是编译`cephfs`这个模块，所以要我修改一下`cmake`的命令

```bash

cd build

# 执行构建
cmake \
    -DUSE_FSAL_PROXY_V4=OFF \
    -DUSE_FSAL_PROXY_V3=OFF \
    -DUSE_FSAL_VFS=ON \
    -DUSE_FSAL_LUSTRE=OFF \
    -DUSE_FSAL_LIZARDFS=OFF \
    -DUSE_FSAL_KVSFS=OFF \
    -DUSE_FSAL_CEPH=ON \
    -DUSE_FSAL_GPFS=OFF \
    -DUSE_FSAL_XFS=OFF \
    -DUSE_FSAL_GLUSTER=OFF \
    -DUSE_FSAL_NULL=OFF \
    -DUSE_FSAL_RGW=OFF \
    -DUSE_FSAL_MEM=OFF \
    -DUSE_9P=OFF \

    ../nfs-ganesha/src

# 编译
cmake --build .
```

如果不出意外的话，在当前`build`目录下应该会出现一个`ganesha.nfsd`可执行文件


# 测试编译结果

在`build`同级目录,新建一个`test`目录

## 将编译结果复制到`test`目录

```bash
cp ../build/ganesha.nfsd ./
```

## 复制配置文件：

```bash
cp ../nfs-ganesha/src/config_samples/ceph.conf ./
```

## 复制并link so文件

由于是使用`ceph`作为`ganesha`的后端,需要复制编译好的`so`文件:

```bash
cp ../build/FSAL/FSAL_CEPH/libfsalceph.so ./

mkdir -p /usr/lib64/ganesha/

ln -s [path]/libfsalcephs.so /usr/lib64/ganesha/
 
```

## 修改一下配置文件如下:

```conf
#
# It is possible to use FSAL_CEPH to provide an NFS gateway to CephFS. The
# following sample config should be useful as a starting point for
# configuration. This basic configuration is suitable for a standalone NFS
# server, or an active/passive configuration managed by some sort of clustering
# software (e.g. pacemaker, docker, etc.).
#
# Note too that it is also possible to put a config file in RADOS, and give
# ganesha a rados URL from which to fetch it. For instance, if the config
# file is stored in a RADOS pool called "nfs-ganesha", in a namespace called
# "ganesha-namespace" with an object name of "ganesha-config":
#
# %url	rados://nfs-ganesha/ganesha-namespace/ganesha-config
#
# If we only export cephfs (or RGW), store the configs and recovery data in
# RADOS, and mandate NFSv4.1+ for access, we can avoid any sort of local
# storage, and ganesha can run as an unprivileged user (even inside a
# locked-down container).
#

NFS_CORE_PARAM
{
	# Ganesha can lift the NFS grace period early if NLM is disabled.
	Enable_NLM = false;

	# rquotad doesn't add any value here. CephFS doesn't support per-uid
	# quotas anyway.
	Enable_RQUOTA = false;

	# In this configuration, we're just exporting NFSv4. In practice, it's
	# best to use NFSv4.1+ to get the benefit of sessions.
	Protocols = 4;
	# NFS_Port = 8088;
}

NFSv4
{
	# Modern versions of libcephfs have delegation support, though they
	# are not currently recommended in clustered configurations. They are
	# disabled by default but can be re-enabled for singleton or
	# active/passive configurations.
	# Delegations = false;

	# One can use any recovery backend with this configuration, but being
	# able to store it in RADOS is a nice feature that makes it easy to
	# migrate the daemon to another host.
	#
	# For a single-node or active/passive configuration, rados_ng driver
	# is preferred. For active/active clustered configurations, the
	# rados_cluster backend can be used instead. See the
	# ganesha-rados-grace manpage for more information.
	# RecoveryBackend = rados_ng;

	# NFSv4.0 clients do not send a RECLAIM_COMPLETE, so we end up having
	# to wait out the entire grace period if there are any. Avoid them.
	Minor_Versions =  1,2;
}

# The libcephfs client will aggressively cache information while it
# can, so there is little benefit to ganesha actively caching the same
# objects. Doing so can also hurt cache coherency. Here, we disable
# as much attribute and directory caching as we can.
MDCACHE {
	# Size the dirent cache down as small as possible.
	Dir_Chunk = 0;
}

EXPORT
{
	# Unique export ID number for this export
	Export_ID=100;

	# We're only interested in NFSv4 in this configuration
	Protocols = 4;

	# NFSv4 does not allow UDP transport
	Transports = TCP;

	#
	# Path into the cephfs tree.
	#
	# Note that FSAL_CEPH does not support subtree checking, so there is
	# no way to validate that a filehandle presented by a client is
	# reachable via an exported subtree.
	#
	# For that reason, we just export "/" here.
	Path = /;

	#
	# The pseudoroot path. This is where the export will appear in the
	# NFS pseudoroot namespace.
	#
	# Pseudo = /cephfs_a/;
	Pseudo = /cephfs_a;
	# We want to be able to read and write
	Access_Type = RW;

	# Time out attribute cache entries immediately
	Attr_Expiration_Time = 0;

	# Enable read delegations? libcephfs v13.0.1 and later allow the
	# ceph client to set a delegation. While it's possible to allow RW
	# delegations it's not recommended to enable them until ganesha
	# acquires CB_GETATTR support.
	#
	# Note too that delegations may not be safe in clustered
	# configurations, so it's probably best to just disable them until
	# this problem is resolved:
	#
	# http://tracker.ceph.com/issues/24802
	#
	# Delegations = R;

	# NFS servers usually decide to "squash" incoming requests from the
	# root user to a "nobody" user. It's possible to disable that, but for
	# now, we leave it enabled.
	Squash = root;

	FSAL {
		# FSAL_CEPH export
		Name = CEPH;

		#
		# Ceph filesystems have a name string associated with them, and
		# modern versions of libcephfs can mount them based on the
		# name. The default is to mount the default filesystem in the
		# cluster (usually the first one created).
		#
		Filesystem = "cephfs";

		#
		# Ceph clusters have their own authentication scheme (cephx).
		# Ganesha acts as a cephfs client. This is the client username
		# to use. This user will need to be created before running
		# ganesha.
		#
		# Typically ceph clients have a name like "client.foo". This
		# setting should not contain the "client." prefix.
		#
		# See:
		#
		# http://docs.ceph.com/docs/jewel/rados/operations/user-management/
		#
		# The default is to set this to NULL, which means that the
		# userid is set to the default in libcephfs (which is
		# typically "admin").
		#
		# User_Id = "ganesha";

		#
		# Key to use for the session (if any). If not set, it uses the
		# normal search path for cephx keyring files to find a key:
		#
		# Secret_Access_Key = "YOUR SECRET KEY HERE";
	}
}

# Config block for FSAL_CEPH
CEPH
{
	# Path to a ceph.conf file for this ceph cluster.
	Ceph_Conf = /etc/ceph/ceph.conf;

	# User file-creation mask. These bits will be masked off from the unix
	# permissions on newly-created inodes.
	# umask = 0;
}

#
# This is the config block for the RADOS RecoveryBackend. This is only
# used if you're storing the client recovery records in a RADOS object.
#
RADOS_KV
{
	# Path to a ceph.conf file for this cluster.
	# Ceph_Conf = /etc/ceph/ceph.conf;

	# The recoverybackend has its own ceph client. The default is to
	# let libcephfs autogenerate the userid. Note that RADOS_KV block does
	# not have a setting for Secret_Access_Key. A cephx keyring file must
	# be used for authenticated access.
	# UserId = "ganesharecov";

	# Pool ID of the ceph storage pool that contains the recovery objects.
	# The default is "nfs-ganesha".
	# pool = "nfs-ganesha";
	pool = cephfs_data;	
	# Consider setting a unique nodeid for each running daemon here,
	# particularly if this daemon could end up migrating to a host with
	# a different hostname (i.e. if you're running an active/passive cluster
	# with rados_ng/rados_kv and/or a scale-out rados_cluster). The default
	# is to use the hostname of the node where ganesha is running.
	# nodeid = hostname.example.com
	nodeid = node1;
}

# Config block for rados:// URL access. It too uses its own client to access
# the object, separate from the FSAL_CEPH and RADOS_KV client.
RADOS_URLS
{
	# Path to a ceph.conf file for this cluster.
	# Ceph_Conf = /etc/ceph/ceph.conf;

	# RADOS_URLS use their own ceph client too. Authenticated access
	# requires a cephx keyring file.
	# UserId = "ganeshaurls";

	# We can also have ganesha watch a RADOS object for notifications, and
	# have it force a configuration reload when one comes in. Set this to
	# a valid rados:// URL to enable this feature.
	# watch_url = "rados://pool/namespace/object";
}

# 日志
LOG
{
 COMPONENTS{
	ALL = FULL_DEBUG;
    }
}

```
## 启动

执行一下命令，如果不出意外的话就可以启动了：
日志会输出在当前目录的`log`文件中

```bash
./ganesha.nfsd -f ceph.conf -F -L log
```
## 挂载

在另一台机器进行挂载:

```bash
mount -t nfs -o nfsvers=4.1,proto=tcp <ganesha-host-name>:<ganesha-pseudo-path> <mount-point>
```


# vscode 配置远程开发

## 下载插件

[Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)


## 配置免密登陆

### 配置密钥

如果没有`ssh`密钥，自行百度生成一份

```bash 
ssh-copy-id ${username}@${host}
```

### 配置ssh的config文件

```bash
vim ~/.ssh/config
```
添加一行配置：

```
# 缩写
Host 
# ip或者hostname
HostName 192.168.101.121
Port 22
PreferredAuthentications publickey
User user
IdentityFile ~/.ssh/id_rsa
```

## 连接

配置上以上流程，这时打开`vscode`，`cmd+shit+p`就可以看到你要链接远程的机器


## 安装必要的插件

**以下插件都是在远程安装的**

- [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
- [CMake](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)


## 配置vscode


###  c_cpp_properties.json
在`.vscode`目录下新建文件： c_cpp_properties.json

内容如下：

```json
{
    "configurations": [
        {
            "name": "Linux",
            # 这里很重要，主要是配置那些路径被include进来，可以减少报错
            "includePath": [
                "${workspaceFolder}/build/include/**",
                "${workspaceFolder}/src/**",
                "/usr/include/**",
                "/usr/include"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c11",
            "cppStandard": "c++14",
            "intelliSenseMode": "${default}"
        }
    ],
    "version": 4
}
```
### settings.json

在`.vscode`目录下新建文件： settings.json

内容如下：

```json
{
    "cmake.sourceDirectory": "/root/nfs-ganesha-code/nfs-ganesha/src",
    "cmake.options.statusBarVisibility": "visible",
    "files.associations": {
        "config.h": "c",
        "nfs4.h": "c",
        "signal.h": "c",
        "internal.h": "c"
    },
    "C_Cpp.default.configurationProvider": "ms-vscode.cmake-tools",
    # 这里就是控制编译那些选项
    "cmake.configureArgs": [
        "-DUSE_FSAL_PROXY_V4=OFF",
        "-DUSE_FSAL_PROXY_V3=OFF",
        "-DUSE_FSAL_VFS=ON",
        "-DUSE_FSAL_LUSTRE=OFF",
        "-DUSE_FSAL_LIZARDFS=OFF",
        "-DUSE_FSAL_KVSFS=OFF",
        "-DUSE_FSAL_CEPH=ON",
        "-DUSE_FSAL_GPFS=OFF",
        "-DUSE_FSAL_XFS=OFF",
        "-DUSE_FSAL_GLUSTER=OFF",
        "-DUSE_FSAL_NULL=OFF",
        "-DUSE_FSAL_RGW=OFF",
        "-DUSE_FSAL_MEM=OFF",
        "-DUSE_9P=OFF"
    ]
}
```

## 验证是否配置成功

点击左侧的`cmake`插件，点击`build`,如果能成功的`build`,就是成功，如果不能，自行根据报错排查
