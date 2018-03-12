---
title: 自建梯子，畅行无阻
copyright: true
date: 2017-12-04 10:12:53
tags: [经验]
category: "经验"
---
[自建梯子教程](https://github.com/codeyu/BTGFW "Optional title")
[自己搭建ss/ssr服务器教程（适合初学者）](https://github.com/XX-net/XX-Net/issues/6506#issuecomment-336799889 "Optional title")

###【一键部署ssr代码】
yum -y install wget
wget -N --no-check-certificate https://softs.fun/Bash/ssr.sh && chmod +x ssr.sh && bash ssr.sh

备用地址
yum -y install wget
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh
<!-- more -->
参数配置
远程端口
密码
加密方式 aes-256-cfb
协议 auth_chain_a
混淆方式 plain

###【谷歌BBR加速教程】

yum -y install wget

wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh

chmod +x bbr.sh

./bbr.sh