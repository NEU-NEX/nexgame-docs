# 逆向工程环境 - Windows篇

前情提要：[逆向工程的介绍](../../reverse/re.md)

## Detect-It-Easy

文件格式识别

开源软件直接github下载就行：<https://github.com/horsicq/DIE-engine/releases>

## IDA pro

> A powerful disassembler, decompiler and a versatile debugger. In one tool.

完整版8599刀/年，购买地址：<https://hex-rays.com/pricing>

免费版反编译功能需联网使用，线下赛就用不了了，不推荐。而且支持的架构少。

破解版推荐使用8, 9版本，其中只有9有Linux版和Mac版（据说Mac用户都用Hopper，不清楚喵）流出。8版本有那种打包好常用插件的，9版本插件需要自己装，且有api变动，部分插件需要修改后才能用。

下面的链接来自52破解

[IDA Pro 9.0 Beta官方泄露版（Win Linux Mac）含所有Decompiler和SDK，更新破解和链接](https://www.52pojie.cn/thread-1953170-1-1.html)

[IDA Pro 8.3 绿色版（2024.2.26更新）](https://www.52pojie.cn/thread-1874203-1-1.html)

## x64dbg

调试器。为什么不直接用IDA呢？我的经验是IDA容易崩，容易卡，看内存不太方便。

开源软件直接github下载就行：<https://github.com/x64dbg/x64dbg>
