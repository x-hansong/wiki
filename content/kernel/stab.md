---
title: stabs符号调试表
layout: page
date: 2015-10-01
---
[TOC]
> add at 2015-10-01
## what's stabs
> Stabs refers to a format for information that describes a program to a debugger.

stabs指的是调试符号表,可以通过gcc编译时加上`-gstabs -S`来生成source.s文件.
它的一般格式是`.stabs "string",type,other,desc,value`

## stabs in JOS
在MIT的JOS实验中,它使用stabs中的调试信息.

```
#define N_GSYM      0x20    // global symbol
#define N_FNAME     0x22    // F77 function name
#define N_FUN       0x24    // procedure name
#define N_STSYM     0x26    // data segment variable
#define N_LCSYM     0x28    // bss segment variable
#define N_MAIN      0x2a    // main function name
#define N_PC        0x30    // global Pascal symbol
#define N_RSYM      0x40    // register variable
#define N_SLINE     0x44    // text segment line number
#define N_DSLINE    0x46    // data segment line number
#define N_BSLINE    0x48    // bss segment line number
#define N_SSYM      0x60    // structure/union element
#define N_SO        0x64    // main source file name
#define N_LSYM      0x80    // stack variable
#define N_BINCL     0x82    // include file beginning
#define N_SOL       0x84    // included source file name
#define N_PSYM      0xa0    // parameter variable
#define N_EINCL     0xa2    // include file end
#define N_ENTRY     0xa4    // alternate entry point
#define N_LBRAC     0xc0    // left bracket
#define N_EXCL      0xc2    // deleted include file
#define N_RBRAC     0xe0    // right bracket
#define N_BCOMM     0xe2    // begin common
#define N_ECOMM     0xe4    // end common
#define N_ECOML     0xe8    // end common (local name)
#define N_LENG      0xfe    // length of preceding entry
```
上面的stabs类型JOS只用到了N_SO, N_SOL, N_FUN, N_SLINE.

其自定义的`stab`结构为:
```
/* Entries in the STABS table are formatted as follows. */
struct stab {
    uint32_t n_strx;        // index into string table of name
    uint8_t n_type;         // type of symbol
    uint8_t n_other;        // misc info (usually empty)
    uint16_t n_desc;        // description field
    uintptr_t n_value;      // value of symbol
};
```

## 例子
- 编写hello.c

        #include <stdio.h>

        int main(void)
        {
            printf("hello\n");
            return 0;
        }

- 编译

        gcc -gstabs -S ./hello.c

- 打开hello.s,可以看到

        .stabs	"main:F(0,1)",36,0,0,main

    其中,"main:F(0,1)"对应n_strx, 36对应n_type(0x24的十进制即36).
