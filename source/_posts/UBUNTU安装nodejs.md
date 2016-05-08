---
title: UBUNTU上安装nodejs
date: 2013-07-20 23:31:52
tags:
---

记录自己在UBUNTU上安装node.js和npm步骤如下：
不必自己输入很多命令，这是写好的步骤，相当与批处理了。
复制以下代码保存为.sh文件
```
echo 'export PATH=$HOME/local/bin:$PATH' >> ~/.bashrc
. ~/.bashrc
mkdir ~/local
mkdir ~/node-latest-install
cd ~/node-latest-install
curl http://nodejs.org/dist/node-latest.tar.gz | tar xz --strip-components=1
./configure --prefix=~/local
make install # ok, fine, this step probably takes more than 30 seconds...
curl https://npmjs.org/install.sh | sh
```
保存好后运行就行了。
开终端进入相应目录运行：sudo bash xxxx.sh

如果遇到以下报错信息：
make[1]: g++:命令未找到
unbuntu默认没有装g++编译器，终端输入：
```
sudo apt-get install g++
```
装好编译器之后再运行上面一段bash

或
curl 命令找不到这种，curl是一个下载器，也是你没装导致的
```
sudo apt-get install curl
```
