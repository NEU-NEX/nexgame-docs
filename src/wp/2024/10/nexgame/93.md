# 【中等】答案自动获取器

<https://www.bilibili.com/video/BV1Tcx1ecEqT/>

为了让这道题不变成win32 api调用题，让同学们使用patch, hook等方式完成，特意让窗口收到点击消息后，触发一次移动，使其被点击后必不在鼠标上。

~~为了防止直接解密flag，我给printFlag函数加了vmp，副作用是IDA会变得非常卡~~

对于说没有inline hook题的同学：我这就给你准备hook解法

对于使用wine卡bug过的同学：以后再出win32题，我要加个检测到wine就用在hackergame学到的wine穿透把他的home给rm -rf了

## 通过patch使窗口不会跳动

找到消息处理函数的方法：查找SetWindowPos的引用

在Imports里找到SetWindowPos，双击进去右键List cross references to，可以看到在sub140007c10里

大概长这样

```
.text:0000000140007D73 loc_140007D73:                          ; CODE XREF: sub_140007C10+11C↑j
.text:0000000140007D73                                         ; sub_140007C10+134↑j ...
.text:0000000140007D73                 movzx   eax, [rsp+0E8h+var_24]
.text:0000000140007D7B                 test    eax, eax
.text:0000000140007D7D                 jnz     loc_140007CB7
.text:0000000140007D83                 mov     [rsp+0E8h+uFlags], 5 ; uFlags
.text:0000000140007D8B                 mov     [rsp+0E8h+cy], 0 ; cy
.text:0000000140007D93                 mov     [rsp+0E8h+var_C8], 0 ; cx
.text:0000000140007D9B                 mov     r9d, [rsp+0E8h+Y] ; Y
.text:0000000140007DA3                 mov     r8d, [rsp+0E8h+X] ; X
.text:0000000140007DAB                 xor     edx, edx        ; hWndInsertAfter
.text:0000000140007DAD                 mov     rax, [rsp+0E8h+arg_0]
.text:0000000140007DB5                 mov     rcx, [rax+8]    ; hWnd
.text:0000000140007DB9                 call    cs:__imp_SetWindowPos
```

把call这一句改成nop即可

## 通过hook调用

下面将使用frida。保存成js文件后通过`frida .\circle.exe -l .\hooking.js`调用

由于加了vmp，导致不能hook通过printFlag调用的api(GetCursorPos)，所以有两个思路。
但总之前求个image base:

```js
const the_base = Process.getModuleByName("circle.exe").base
console.log("The base address of the module is: " + the_base);
```

- 让窗口无法移动：替换SetWindowPos

```js
let fxp = Module.getExportByName(null, "SetWindowPos")
let fp = new NativeFunction(fxp,  "bool", ["int64", "int64", "int", "int", "int", "int", "uint"])
Interceptor.replace(fp, new NativeCallback((h1,h2,x,y,x1,y1,u)=>{return 1},  "bool", ["int64", "int64", "int", "int", "int", "int", "uint"]))
```

- 在窗口移动前调用printFlag

```js
const f_addr = the_base.add(0x3d91)
console.log("Func addr: ", f_addr)
const f = new NativeFunction(ptr(f_addr), 'void', ["pointer"]);
Interceptor.attach(ptr(the_base.add(0x83c0)), {
  onEnter: function (args) {
    if (args[1] == 0x200)
      f(args[0])
  },
});
```