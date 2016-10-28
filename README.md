# UPYUNFS

### 简介

upyunfs 能让您在Linux/Mac OS X 系统中把UpYun bucket 挂载到本地文件
系统中，您能够便捷的通过本地文件系统操作UpYun 上的对象，实现数据的共享。

### 功能

upyunfs 基于s3fs 构建，具有s3fs 的全部功能。主要功能包括：

* 支持POSIX 文件系统的大部分功能，包括文件读写，目录，链接操作，权限，
  uid/gid，以及扩展属性（extended attributes）
* 通过UpYun 的multipart 功能上传大文件(暂未完成)。
* MD5 校验保证数据完整性。

### 安装

#### 源码安装

暂未提供发行版的安装包，您可以自行编译安装。编译前请先安装下列依赖库：

Ubuntu 14.04:

```
sudo apt-get install automake autotools-dev g++ git libcurl4-gnutls-dev \
                     libfuse-dev libssl-dev make pkg-config
```

CentOS 7.0:

```
sudo yum install automake gcc-c++ git libcurl-devel \
                 fuse-devel make openssl-devel
```

然后您可以在github上下载源码并编译安装：

```
git clone https://github.com/ziranwei/upyunfs.git
cd upyunfs
./autogen.sh
./configure
make
sudo make install
```

### 运行

设置username/password信息，将其存放在/etc/passwd-upyunfs 文件中，
注意这个文件的权限必须正确设置，建议设为640。

```
echo my-username:my-password > /etc/passwd-upyunfs
chmod 640 /etc/passwd-upyunfs
```

将upyun bucket mount到指定目录

```
upyunfs my-bucket my-mount-point -ourl=my-upyun-endpoint
```
#### 示例

将`my-bucket`这个bucket挂载到`/tmp/upyunfs`目录下，username是`faint`，
password是`123`，upyun endpoint是`v0.api.upyun.com`

```
echo my-bucket:faint:123 > /etc/passwd-upyunfs
chmod 640 /etc/passwd-upyunfs
mkdir /tmp/upyunfs
upyunfs my-bucket /tmp/upyunfs -o url=v0.api.upyun.com
```

卸载bucket:

```bash
umount /tmp/upyunfs # root user
fusermount -u /tmp/upyunfs # non-root user
```

#### 常用设置

- 使用`upyunfs --version`来查看当前版本，使用`upyunfs -h`来查看可用的参数

- 在linux系统中，[updatedb][updatedb]会定期地扫描文件系统，如果不想
  upyunfs的挂载目录被扫描，可参考[FAQ][FAQ-updatedb]设置跳过挂载目录
- 如果你没有使用[eCryptFs][ecryptfs]等需要[XATTR][xattr]的文件系统，可
  以通过添加`-o noxattr`参数来提升性能

- 生产环境中推荐使用[supervisor][supervisor]来启动并监控upyunfs进程，使
  用方法见[FAQ][faq-supervisor]

#### 高级设置

- 可以添加`-f -d`参数来让upyunfs运行在前台并输出debug日志

### 遇到错误

遇到错误不要慌:) 按如下步骤进行排查：

1. 如果有打印错误信息，尝试阅读并理解它
2. 查看`/var/log/syslog`或者`/var/log/messages`中有无相关信息

        grep 's3fs' /var/log/syslog
        grep 'upyunfs' /var/log/syslog

3. 重新挂载upyunfs，打开debug log：

        upyunfs ... -o dbglevel=debug -f -d > /tmp/fs.log 2>&1

    然后重复你出错的操作，出错后将`/tmp/fs.log`保留，自己查看或者发给我

### 局限性

upyunfs提供的功能和性能和本地文件系统相比，具有一些局限性。具体包括：

* 随机或者追加写文件会导致整个文件的重写。
* 元数据操作，例如list directory，性能较差，因为需要远程访问upyun服务器。
* 文件/文件夹的rename操作不是原子的。
* 多个客户端挂载同一个upyun bucket时，依赖用户自行协调各个客户端的行为。例如避免多个客户端写同一个文件等等。
* 不支持hard link。
* 不适合用在高并发读/写的场景，这样会让系统的load升高
* 目前不支持符号链接

