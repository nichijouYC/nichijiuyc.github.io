---
title: ubuntu更换内核的方法
date: 2016-03-02 14:24:34
tags:
---


1. 查看当前安装的内核
`dpkg -l|grep linux-image`

2. 查看可以更新的内核版本
`sudo apt-cache search linux-image`

3. 安装新内核
`sudo apt-get install linux-image-4.2.0-35-generic linux-image-extra-4.2.0-35-generic`

4. 卸载不要的内核
`sudo apt-get purge linux-image-3.13.0-105-generic linux-image-extra-3.13.0-105-generic`

5. 更新 grub引导
`sudo update-grub`

6. 重启系统查看。
