---
layout: post
title:  "__builtin_return_address"
date:   2022-07-21 
categories: Objective-c
tags: Objective-c
---

#  内建函数 __builtin_return_address

__builtin_return_address 函数的作用是返回所在函数被一级函数调用后，退出的地址(通常为return)。

英文原意是“When inlining the expected behavior is that the function returns the address of the function that is returned to”

代码:
```
- (void)viewDidLoad {
    [super viewDidLoad];
    g();
    // Do any additional setup after loading the view.
}

void f()
{
    printf("%p,%p" , __builtin_return_address(0), __builtin_return_address(1));
    //printk("Caller is %pS\n", __builtin_return_address(0));
}

void g()
{
    f();
}
```

输出结果(基地址 0x00000001047d0000) `0x1047d1e40,0x1047d1e28`. 使用hopper 反汇编二进制得到,
 - `- (void)viewDidLoad`汇编
 
 ```
  ; ================ B E G I N N I N G   O F   P R O C E D U R E ================

        ; Variables:
        ;    saved_fp: 0
        ;    var_8: int64_t, -8
        ;    var_10: int64_t, -16
        ;    var_18: int64_t, -24
        ;    var_20: int64_t, -32


                     -[ViewController viewDidLoad]:
00000001047d1dec         sub        sp, sp, #0x30                               ; Objective C Implementation defined at 0x1047d2440 (instance method)
00000001047d1df0         stp        fp, lr, [sp, #0x20]
00000001047d1df4         add        fp, sp, #0x20
00000001047d1df8         stur       x0, [fp, var_8]
00000001047d1dfc         str        x1, [sp, #0x20 + var_10]
00000001047d1e00         ldur       x8, [fp, var_8]
00000001047d1e04         mov        x0, sp                                      ; argument "super" for method imp___stubs__objc_msgSendSuper2
00000001047d1e08         str        x8, [sp, #0x20 + var_20]
00000001047d1e0c         adrp       x8, #0x1047d9000                            ; 0x1047d92a0@PAGE
00000001047d1e10         ldr        x8, [x8, #0x2a0]                            ; 0x1047d92a0@PAGEOFF, 0x1047d92a0,_OBJC_CLASS_$_ViewController
00000001047d1e14         str        x8, [sp, #0x20 + var_18]
00000001047d1e18         adrp       x8, #0x1047d9000
00000001047d1e1c         ldr        x1, [x8, #0x278]                            ; argument "selector" for method imp___stubs__objc_msgSendSuper2, "viewDidLoad",@selector(viewDidLoad)
00000001047d1e20         bl         imp___stubs__objc_msgSendSuper2             ; objc_msgSendSuper2
00000001047d1e24         bl         __Z1gv                                      ; g()
// g() 返回后的首地址
00000001047d1e28         ldp        fp, lr, [sp, #0x20]
00000001047d1e2c         add        sp, sp, #0x30
00000001047d1e30         ret
                        ; endp

```
- `g()`函数汇编:
```
        ; ================ B E G I N N I N G   O F   P R O C E D U R E ================

        ; Variables:
        ;    saved_fp: 0


                     __Z1gv:        // g()
00000001047d1e34         stp        fp, lr, [sp, #-0x10]!                       ; CODE XREF=-[ViewController viewDidLoad]+56
00000001047d1e38         mov        fp, sp
00000001047d1e3c         bl         __Z1fv                                      ; f()
// f()函数return后的地址
00000001047d1e40         ldp        fp, lr, [sp], #0x10
00000001047d1e44         ret
                        ; endp
```
- `f()`函数汇编:
```

        ; ================ B E G I N N I N G   O F   P R O C E D U R E ================

        ; Variables:
        ;    saved_fp: int64_t, 0


                     __Z1fv:        // f()
00000001047d1e48         sub        sp, sp, #0x20                               ; CODE XREF=__Z1gv+8
00000001047d1e4c         stp        fp, lr, [sp, #0x10]
00000001047d1e50         add        fp, sp, #0x10
00000001047d1e54         ldr        x8, [fp, saved_fp]
00000001047d1e58         ldr        x8, [x8, #0x8]
00000001047d1e5c         adrp       x0, #0x1047d3000                            ; 0x1047d32e0@PAGE
00000001047d1e60         add        x0, x0, #0x2e0                              ; 0x1047d32e0@PAGEOFF, argument "format" for method imp___stubs__printf, "%p,%p"
00000001047d1e64         mov        x9, sp
00000001047d1e68         str        lr, [x9]
00000001047d1e6c         str        x8, [x9, #0x8]
00000001047d1e70         bl         imp___stubs__printf                         ; printf
00000001047d1e74         ldp        fp, lr, [sp, #0x10]
00000001047d1e78         add        sp, sp, #0x20
00000001047d1e7c         ret
 ```
 
综上得出:
- `0x1047d1e40` 对应 f()函数return后的地址
- `0x1047d1e28` 对应 g()函数return后的地址

链接:
[iOS 内存管理思考 - autorelease](https://minosjy.com/2021/09/12/18/439/)
[How does objc_retainAutoreleasedReturnValue work?](https://www.galloway.me.uk/2012/02/how-does-objc_retainautoreleasedreturnvalue-work/)