---
layout: post
title:  "mod_init_func"
date:   2022-05-01
categories: dyld
tags: dyld
---




# Section64(__Data_CONST,__mod_init_func)

## 1. 使用背景
module initializer functions，模块初始化函数, 该节的作用主要是用于dyld执行 `start` 方法时, 调用 `runDyldInitializers` 执行必要的初始化:
```
#if DYLD_INITIALIZER_SUPPORT

typedef void (*Initializer)(int argc, const char* argv[], const char* envp[], const char* apple[]);

//Section64(__Data_CONST,__mof_init_func) 节的开始地址
extern const Initializer  inits_start  __asm("section$start$__DATA$__mod_init_func");
//Section64(__Data_CONST,__mof_init_func) 节的结束地址
extern const Initializer  inits_end    __asm("section$end$__DATA$__mod_init_func");

//
// For a regular executable, the crt code calls dyld to run the executables initializers.
// For a static executable, crt directly runs the initializers.
// dyld (should be static) but is a dynamic executable and needs this hack to run its own initializers.
// We pass argc, argv, etc in case libc.a uses those arguments
//
static void runDyldInitializers(const struct macho_header* mh, intptr_t slide, int argc, const char* argv[], const char* envp[], const char* apple[])
{
    // 遍历Section64(__Data_CONST,__mof_init_func) 节中的所有函数地址, 依次调用 C++ 的初始化函数，也就是调用被 __attribute__((constructor)) 修饰的函数

	for (const Initializer* p = &inits_start; p < &inits_end; ++p) {
		(*p)(argc, argv, envp, apple);
	}
}
#endif // DYLD_INITIALIZER_SUPPORT

```
## 2. 扩展

### 2.1 获取__TEXT section 起始地址, 对于风控SDK, 可以用来上报当前代码节的内容, 判断是否被修改
```
// __TEXT 节的起始地址
typedef void (*InitializerTextSection);
extern const InitializerTextSection  text_section_start  __asm("section$start$__TEXT$__text");
extern const InitializerTextSection  text_section_end    __asm("section$end$__TEXT$__text");
```

### 2.2 调用被 `__attribute__((constructor))` 修饰的函数

```
__attribute__((constructor)) void init_funcs()
{
    printf("--------init funcs.--------\n");

    printf("--------init done--------\n");
}
__attribute__((constructor)) void init_funcs2()
{
    printf("--------init funcs2.--------\n");

    printf("--------init done2--------\n");
}

typedef void (*Initializer)(int argc, const char* argv[], const char* envp[], const char* apple[]);
extern const Initializer  inits_start  __asm("section$start$__DATA$__mod_init_func");
extern const Initializer  inits_end    __asm("section$end$__DATA$__mod_init_func");
```

在main函数中调用:
```
 NSLog(@"inits_start:%p", &inits_start);
    NSLog(@"inits_end:%p", &inits_end);
    for (const Initializer* p = &inits_start; p < &inits_end; ++p) {
        (*p)(argc, argv, NULL, NULL);
    }

```

[内嵌汇编](https://www.dllhook.com/post/249.html)
[参见](https://blog.csdn.net/majiakun1/article/details/99413403)