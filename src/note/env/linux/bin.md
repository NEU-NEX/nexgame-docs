# 逆向工程环境 - Linux篇

前情提要：[逆向工程的介绍](../../re.md)

## Detect-It-Easy

文件格式识别

开源软件直接github下载就行：<https://github.com/horsicq/DIE-engine/releases>

## 反汇编平台

我一般只有在调试程序时使用Linux，这部分建议可能有不足之处。

Linux下能用的有：

- 任意版本的IDA free，任意正版IDA，盗版IDA pro 9, 
- Binary Ninja
- Ghidra

具体安装过程相信准备拿Linux做主系统跑反汇编平台的你一定不需要我再赘述了

## 调试器

功能比较完善的就是那几个gdb发行版(gef, pwndbg, peda)，他们有一些功能上的区别，请在使用中自行领悟。

三者共存安装：<https://github.com/apogiatzis/gdb-peda-pwndbg-gef>

实在想要图形界面的话考虑使用edb。
