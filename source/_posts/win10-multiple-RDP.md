---
title: "win10 LTSC 2021 开启多用户 RDP 远程登录"
url: win10-multiple-RDP.html
id: win10-multiple-RDP
index_img: /img/post/FY7smBWUcAAaaB2.jpg
categories:
  - 开发日志
date: 2023-01-18 10:40:00
tags:
---

![](/img/post/FY7smBWUcAAaaB2.jpg)

<!-- TOC -->

- [简介](#%E7%AE%80%E4%BB%8B)
- [前置条件](#%E5%89%8D%E7%BD%AE%E6%9D%A1%E4%BB%B6)
- [具体操作](#%E5%85%B7%E4%BD%93%E6%93%8D%E4%BD%9C)
- [总结](#%E6%80%BB%E7%BB%93)

<!-- /TOC -->

### 简介

win10 默认不支持多用户同时登录，需要借助软件修改 RDP 功能，win10 普通版和 LTSC 2021 版本均适用

> 参考链接 [How to Allow Multiple RDP Sessions in Windows 10 and 11? | Windows OS Hub](https://woshub.com/how-to-allow-multiple-rdp-sessions-in-windows-10/)

### 前置条件

- 下载 RDPWrap，地址 [Releases · stascorp/rdpwrap · GitHub](https://github.com/binarymaster/rdpwrap/releases)

- 下载最新配置文件 地址 [rdpwrap.ini](https://raw.githubusercontent.com/sebaxakerhtc/rdpwrap.ini/master/rdpwrap.ini)

### 具体操作

1. 配置组策略
   
   - 快捷键 `Win+R` 打开 `运行`，输入 `gpedit.msc` 打开 ` 本地组策略编辑器`
   
   - 依次点开 `本地计算机 策略 > 计算机配置 > 管理模板 > Windows 组件 > 远程桌面服务 > 远程桌面会话主机 > 连接`
   
   - 查看右侧设置项，需要修改 3 个配置，分别如下
     
     - 开启 `允许用户通过使用远程桌面服务进行远程连接`
       
       ![2023-01-08_16-10-07.png](/img/post/multi-rdp/2023-01-08_16-10-07.png)
     
     - 开启 `限制连接的数量`，并设置最大连接数，根据需求设置即可
       
       ![2023-01-08_16-10-45.png](/img/post/multi-rdp/2023-01-08_16-10-45.png)
     
     - 开启 `将远程桌面服务用户限制到单独的远程桌面服务会话`，**重要选项**，用来保持用户程序的后台运行
       
       ![2023-01-08_16-11-05.png](/img/post/multi-rdp/2023-01-08_16-11-05.png)
   
   - 修改完成后需要刷新组策略，右键管理员打开 `cmd` 或者 `powershell`，输入 `gpupdate /force`，刷新完成后即可关闭命令行

2. 安装 RDPWrap
   
   - 打开下载的 RDPWrap 文件夹，右键管理员运行 `install.bat`
   - 双击运行 `RDPConf.exe`
     
     ![2023-01-08_16-34-28.png](/img/post/multi-rdp/2023-01-08_16-34-28.png)
   - 查看右上角，如果显示红色的 `[not supported]`，先关闭 RDPConf, 再用下载的 `rdpwrap.ini` 覆盖 `C:\Program Files\RDP Wrapper\rdpwrap.ini`
   - 不用重启电脑，重新运行 `RDPConf.exe`，此时右上角应该显示绿色的 `[fully supported]`
   - **至此多用户登录设置已经完成**，如果本地账户只有一个，继续参考后续内容添加用户即可

3. *添加用户并配置远程登录权限
   
   - 右键点击桌面左下角 `Win` 标志，点击打开 `计算机管理`
   
   - 依次打开 `系统工具 > 本地用户和组 > 用户`
   
   - 右键点击 `用户`，选择 `新用户`，根据需求创建新用户，完成后即可关闭 `计算机管理`
   
   - 回到桌面，依次打开 `系统设置 > 系统 > 远程桌面 > 选择可远程访问这台电脑的用户`
     
     ![2023-01-08_16-51-50.png](/img/post/multi-rdp/2023-01-08_16-51-50.png)
   
   - 依次打开 `添加 > 高级 > 立即查找`，在下方搜索结果中找到新添加的用户，依次确定即可
     
     ![2023-01-08_16-58-19.png](/img/post/multi-rdp/2023-01-08_16-58-19.png)

### 总结

部分系统版本可能完成这些操作后，依然无法同时登录多个用户，可以尝试重启电脑，**不推荐直接覆盖** `c:\Windows\System32\termsrv.dll`


