---
title: "Ubuntu适应计划-FAQ"
date: 2017-08-08 0800
cover: "4.jpg"
category: "Linux"
tags:
    - Linux
---

# Ubuntu适应计划
## 终端无法输入中文
终端和在终端里打开的编辑器无法输入中文的问题：
```shell
gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "{'Gtk/IMModule':<'fcitx'>}"
```
## `npm i -g`权限问题
刚刚遇到的问题。在Ubuntu下使用
```shell
npm i -g <packagename>
```
或`yarn`的类似命令时，总会遇到权限不够的情况，这是因为全局安装的package都在`/usr/lib/node_modules`里面，普通用户没有写权限。这时可以：
```shell
sudo chown -R me /usr/lib/node_modules
```
来临时更改全局包文件夹的权限，然后再
```shell
npm i -g <packagename>
```
即可，之后再把权限改回root。之前使用`sudo npm i -g <packagename>`也会出现这种问题，是因为一些包比如`hexo-cli`等都是有postinstall脚本的，之后运行的这些脚本也需要root权限，就会报错。
