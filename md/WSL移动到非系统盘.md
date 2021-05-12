\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.cnguu.cn\](https://blog.cnguu.cn/tool/ji-luwsl-congc-pan-qian-yi-zhid-pan.html)

[#](#前言) 前言
-----------

WSL 默认安装在 C 盘，随着开发时间的增长，数据越来越多，子系统数据占用高达 60 GB，对于原本 100 GB 的 C 盘，不堪重负，终于只剩下不足 300 MB 的空间，随之而来的就是 PHPStorm 无法打开

为了解决这个问题，需要迁移 WSL 默认存储位置

[#](#过程) 过程
-----------

1.  下载工具

*   [LxRunOffline](https://github.com/DDoSolitary/LxRunOffline) ：一个非常强大的管理子系统的工具

下载并解压后，在解压目录中打开 PowerShell

2.  查看已安装的子系统

```
$ ./LxRunOffline.exe list
```

1  

3.  查看子系统所在目录

```
$ ./LxRunOffline.exe get-dir -n Ubuntu-18.04
```

1  

4.  新建目标目录并授权

```
$ icacls D:\\wsl\\installed /grant "cnguu:(OI)(CI)(F)"
```

1  

*   目标目录：`D:\wsl\installed`
*   用户名：`cnguu`

5.  迁移系统

```
$ .\\LxRunOffline move -n Ubuntu-18.04 -d D:\\wsl\\installed\\Ubuntu-18.04
```

1  

然后耐心等待一大堆 `Warning` 的结束

> 如果报错：`[ERROR] The distro "Ubuntu-18.04" has running processes and can't be operated.`
> 
> 需要重启服务：`LxssManager`（快捷键：同时按 `Win + x`，再按 `g`）

[#](#结果) 结果
-----------

C 盘满血复活（不知道是不是错觉，感觉读写文件速度快了很多）
