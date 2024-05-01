# Ubuntu 18.0.4 安装 libc6 2.28 及公钥验证相关

今天打算在 window 11 上安装一个 OWT-Server 环境。照着网上的 [OWT-Server 5.0编译与运行指南](https://juejin.cn/post/7087579313758797837) 通过 docker pull registry.cn-hangzhou.aliyuncs.com/wisefeng/owt-server:v5.0 安装了已打包好的 docker 文件（即已执行了 ./scripts/pack.js -t all步骤）。但是在启动 owt-server 的时候报 GLIBC_2.28 not found 错误，经过一番尝试，现将处理过程记录如下。

## 一、添加软件源

```bash
sudo vim /etc/apt/sources.list
# 在最后一行添加
deb http://security.debian.org/debian-security buster/updates main
# 另外，我的网络使用 https 更好用，用 http 容易连不上
deb https://security.debian.org/debian-security buster/updates main
```

## 二、更新软件源

```bash
sudo apt update
# 这时应该会在终端中提示（注意 key 是否和你的一直）：由于没有公钥，无法验证下列签名： NO_PUBKEY 04EE7237B7D453EC NO_PUBKEY 648ACFD622F3D138
```

更新源失败，目前有两种解决方法：

### 2.1 安装 debian-archive-keyring

```bash
sudo apt-get install debian-archive-keyring
sudo apt update
```

如果没问题，就不用看 2.2 了。

### 2.2 手动信任 key

```bash
sudo apt-key adv --keyserver hkp://http://keyserver.ubuntu.com:80 --recv-keys 112695A0E562B32A 54404762BBB6E853
# 上面的和下面的有点差别，我用的是下面的
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 112695A0E562B32A 54404762BBB6E853
sudo apt update
```

### 三、升级 apt

当我在执行完上面的操作后，通过 

	strings /lib/x86_64-linux-gnu/libc.so.6 | grep GLIBC_

并没有发现 libc6 的 2.28 版本，就执行了

	sudo apt upgrade

然后安装 libc6 和 libc6-dev 的 2.28 版本成功。

参考：

[五步解决 Ubuntu 18.04 出现GLIBC_2.28 not found的解决方法](https://blog.csdn.net/summer_lele/article/details/136166760) （含有代理服务器 squid 更新 apt 源的解决方案）

[Ubuntu:已解决：安装18.04后报错：依赖: libc6-armel-cross (＞= 2.27) 但是 2.23-0ubuntu3cross1 已经安装](https://blog.csdn.net/GentelmanTsao/article/details/114891872)

[解决Linux Ubuntu环境(docker : Ubuntu 18.04)报错libc.so.6：version GLIBC_2.28 not found的错误](https://zhuanlan.zhihu.com/p/627165977)

[Debian apt update 提示 由于没有公钥，无法验证下列签名...](https://blog.csdn.net/markinstephen/article/details/123094575)