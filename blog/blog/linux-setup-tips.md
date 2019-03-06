---
title: linux 搭建 tips
date: 2018-03-09 21:13:37
type: post
tag: linux
meta:
  - name: description
    content: linux 搭建记录
  - name: keywords
    content: linux
---

### tools

```bash
# 多线程下载工具
sudo apt-get install axel

# 待补充
```

### nginx

访问[ nginx ](http://nginx.org/)选择一个版本下载到服务器

如:

```bash
axel -n 50 http://nginx.org/download/nginx-1.12.2.tar.gz
```

配置 `nginx` 安装路径

```bash
sudo ./configure --prefix=/home/ubuntu/tools/nginx
```

缺少的库, 一般而言, 有下面两个

```bash
sudo apt-get install libpcre3 libpcre3-dev
sudo apt-get install zlib1g-dev
```

make file

```bash
make && make install
```

### git

ssh-key

```bash
ssh-keygen -t rsa -C "email"
# 如果没有 .ssh 的写入权限, 运行下面的指令
mkdir ~/.ssh
chmod 777 ~/.ssh
```

直接回车安装完成

```bash
cat ~/.ssh/id_rsa.pub
```

复制公钥到 `git` 服务器的 `ssh-keys` 列表中

### python

pip

```bash
sudo apt-get install python-pip
# pip3
sudo apt-get install python3-pip
```

virtualenv

```bash
pip3 install virtualenv
# 如果出 unsupported locale setting 错, 运行下面的命令去除所有本地化设置
export LC_ALL=C
```

创建 `venv`

```bash
virtualenv --no-site-packages venv
# 如果出错, 尝试执行下面这条命令
python3 -m virtualenv venv
```
