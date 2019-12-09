# Docker安装

## 卸载旧版本

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

## 设置Docker仓库

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker 。

```
sudo apt-get update
```

安装 apt 依赖包，用于通过HTTPS来获取仓库:

```
  apt-get instal \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common
```

添加 Docker 的官方 GPG 密钥：

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

验证秘钥

```
apt-key fingerprint 0EBFCD88
```

设置稳定版仓库

```
  add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```

## 安装 Docker Engine-Community

更新apt包索引

```
apt-get update
```

安装最新版本的 Docker Engine-Community 和 containerd ：

```
apt-get install docker-ce docker-ce-cli containerd.io
```

