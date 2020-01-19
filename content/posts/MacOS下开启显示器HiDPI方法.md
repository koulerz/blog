---
title: "macOS 下开启显示器 HiDPI 方法"
date: 2016-11-23T10:21:42+08:00
publishdate: 2016-11-23
lastmod: 2020-01-19
draft: false
tags: ["mac", "os"]
---
## 备份文件
In Google Cloud

## 系统环境
操作系统：MacOS

显示器型号：DELL2515H

显示器分辨率：2560 * 1440 2K

## MacOS系统获取系统权限方法
MacOS系统下，某些系统文件夹只有系统有权限读写，称为System Integrity Protection（SIP），需要提前关闭。

## 创建相应文件夹和文件
首先在终端获取显示器的 DisplayVendorID 和 DisplayProductID。
```bash
# 获取显示器 DisplayVendorID
ioreg -l | grep "DisplayVendorID"

# 获取显示器 DisplayProductID
ioreg -l | grep "DisplayProductID"
```

在桌面新建一个文件夹，名称格式为 DisplayVendorID-xxxx，其中 xxxx 为显示器 DisplayVendorID 的16进制小写

在刚创建的文件夹中新建空白文件，名称格式为 DisplayProductID-xxxx，其中 xxxx 为显示器 DisplayProductID 的16进制小写

以上步骤可使用脚本 patch-edid.rb 自动生成。

生成步骤：
1. 下载或手动创建文件 patch-edid.rb 到桌面目录
2. 打开终端进入桌面目录
3. 执行命令 `ruby patch-edid.rb`
4. 脚本会在桌面目录创建好文件夹和文件（文件内容也有，但可能没有效果）

## 编辑新建的文件
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DisplayProductID</key>
    <integer>53358</integer>
    <key>DisplayVendorID</key>
    <integer>4268</integer>
    <key>scale-resolutions</key>
    <array>
        <data>AAAKAAAABXgAAAABACAAAA==</data>
        <data>AAAUAAAACvAAAAABACAAAA==</data>
        <data>AAAHgAAABDgAAAABACAAAA==</data>
        <data>AAAPAAAACHAAAAABACAAAA==</data>
        <data>AAAGQAAAA4QAAAABACAAAA==</data>
        <data>AAAMgAAABwgAAAABACAAAA==</data>
        <data>AAAFAAAAAtAAAAABACAAAA==</data>
        <data>AAAKAAAABaAAAAABACAAAA==</data>
    </array>
</dict>
</plist>
```
该文件是XML格式的，在Mac中被称为plist（Property List）文件，即属性列表文件。可以使用工具 PlistPro 来编辑。

文件中共有 8 个 data 标签，其中每两个为一组，即（1，2）、（3，4）、（5，6）、（7，8），每一个 data 标签表示一个分辨率。

```python
# data 标签解码后
'''
00000A00 000005A0 00000001 00200000 
00001400 00000B40 00000001 00200000 

00000780 00000438 00000001 00200000 
00000F00 00000870 00000001 00200000 

00000640 00000384 00000001 00200000 
00000C80 00000708 00000001 00200000 

00000500 000002D0 00000001 00200000 
00000A00 000005A0 00000001 00200000 
'''

# 转为十进制

'''
2560 1440 1 2097152
5120 2880 1 2097152

1920 1080 1 2097152
3840 2160 1 2097152

1600 900  1 2097152
3200 1800 1 2097152

1280 720  1 2097152
2560 1440 1 2097152
'''
```
可见每组 data 标签中，第一个 data 标签为我们需要HiDPI的分辨率，第二个 data 标签是第一个标签分辨率的两倍。

可以根据自己的显示器随意填多个 data 标签组，编辑文件时需要填十六进制

## 使用新建的HiDPI分辨率
将新建的文件夹整个拷贝到目录 /System/Library/Displays/Contents/Resources/Overrides/ 中，如果有重复的文件夹，建议先把原来的文件夹备份。重启之后就可以生效了。

可以使用工具 SwitchResX 或 RDM（RetinaDisplayMenu） 来切换不同分辨率。

## 参考 & 扩展
- [Mac OS X 10.11 开启任意显示器 开启HiDPI方法](http://bbs.feng.com/read-htm-tid-9948814.html)
- [Mac 2k 屏原生分辨率完美支持教程](http://bbs.feng.com/read-htm-tid-8937471.html)
- [MacOS无法获得权限问题](http://note.youdao.com/noteshare?id=741857f7c6dccdc2b8adc5d42e33a160)
- [patch-edid.rb](https://gist.github.com/adaugherity/7435890)
- [RDM（RetinaDisplayMenu）](https://github.com/avibrazil/RDM)
- [SwitchResX](http://download.cnet.com/SwitchResX/3000-2094_4-10558576.html)
- [Enable-HiDPI-OSX](https://github.com/syscl/Enable-HiDPI-OSX)