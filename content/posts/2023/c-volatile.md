---
title: "C语言: volatile关键字"
date:  2023-02-11T19:19:34+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
categories:
- C/C++

tags:
- C/C++

---


# volatile关键字

## 来自维基百科的介绍:

>在程序设计中，尤其是在C语言、C++、C#和Java语言中，使用**volatile**关键字声明的变量或对象通常具有与优化、多线程相关的特殊属性。通常，**volatile**关键字是用来阻止（伪）编译器因误认某段代码无法被代码本身所改变，而造成的过度优化。如在C语言中，**volatile**关键字可以用来提醒编译器它后面所定义的变量随时有可能改变，因此编译后的程序每次需要存储或读取这个变量的时候，都会直接从变量地址中读取数据。如果没有**volatile**关键字，则编译器可能优化读取和存储，可能暂时使用寄存器中的值，如果这个变量由别的程序更新了的话，将出现不一致的现象。
>
>在C环境中，**volatile**关键字的真实定义和适用范围经常被误解。虽然C++、C#和Java都保留了C中的volatile关键字，但在这些编程语言中**volatile**的用法和语义却大相径庭。

## 示例:

`以下使用的编译器是(x86 msvc v19.14)`

### 比如下面代码：

```c
#include <stdio.h>


int main() {
  int i=11;
  int a=i;
  printf("i=%d\n",a);
  __asm{
      mov dword ptr [ebp-4], 10h
  }
  int d=i;
  printf("i=%d\n",d);
 return 0;
}
```

其未进行优化的汇编代码:

```assembly
_DATA   SEGMENT
COMM    ?_OptionsStorage@?1??__local_stdio_printf_options@@9@9:QWORD                                                    ; `__local_stdio_printf_options'::`2'::_OptionsStorage
_DATA   ENDS
_DATA   SEGMENT
$SG7395 DB        'i=%d', 0aH, 00H
        ORG $+2
$SG7396 DB        'i=%d', 0aH, 00H
_DATA   ENDS

_d$ = -12                                         ; size = 4
_a$ = -8                                                ; size = 4
_i$ = -4                                                ; size = 4
_main   PROC
        push    ebp
        mov     ebp, esp
        sub     esp, 12                             ; 0000000cH
        mov     DWORD PTR _i$[ebp], 11                    ; 0000000bH
        mov     eax, DWORD PTR _i$[ebp]
        mov     DWORD PTR _a$[ebp], eax
        mov     ecx, DWORD PTR _a$[ebp]
        push    ecx
        push    OFFSET $SG7395
        call    _printf
        add     esp, 8
        mov     DWORD PTR [ebp-4], 16           ; 00000010H
        mov     edx, DWORD PTR _i$[ebp]
        mov     DWORD PTR _d$[ebp], edx
        mov     eax, DWORD PTR _d$[ebp]
        push    eax
        push    OFFSET $SG7396
        call    _printf
        add     esp, 8
        xor     eax, eax
        mov     esp, ebp
        pop     ebp
        ret     0
_main   ENDP
```

输出：

```c
i=11
i=16
```

开启O2优化的汇编代码：

```assembly
_DATA   SEGMENT
COMM    ?_OptionsStorage@?1??__local_stdio_printf_options@@9@9:QWORD                                                    ; `__local_stdio_printf_options'::`2'::_OptionsStorage
_DATA   ENDS
??_C@_05BKKKKIID@i?$DN?$CFd?6?$AA@ DB 'i=%d', 0aH, 00H  ; `string'

_main   PROC                                      ; COMDAT
        push    11                                  ; 0000000bH
        push    OFFSET ??_C@_05BKKKKIID@i?$DN?$CFd?6?$AA@
        call    _printf
        add     esp, 8
        mov     DWORD PTR [ebp-4], 16           ; 00000010H
        push    11                                  ; 0000000bH
        push    OFFSET ??_C@_05BKKKKIID@i?$DN?$CFd?6?$AA@
        call    _printf
        add     esp, 8
        xor     eax, eax
        ret     0
_main   ENDP
```

输出：

```c
i=11
i=11
```

### 对代码进行修改，添加`volatile`后

```c
#include <stdio.h>


int main() {
  volatile int i=11;
  int a=i;
  printf("i=%d\n",a);
  __asm{
      mov dword ptr [ebp-4], 10h
  }
  int d=i;
  printf("i=%d\n",d);
 return 0;
}
```

开启O2优化的汇编代码

```assembly
_DATA   SEGMENT
COMM    ?_OptionsStorage@?1??__local_stdio_printf_options@@9@9:QWORD                                                    ; `__local_stdio_printf_options'::`2'::_OptionsStorage
_DATA   ENDS
??_C@_05BKKKKIID@i?$DN?$CFd?6?$AA@ DB 'i=%d', 0aH, 00H  ; `string'

_i$ = -4                                                ; size = 4
_main   PROC                                      ; COMDAT
        push    ebp
        mov     ebp, esp
        push    ecx
        mov     DWORD PTR _i$[ebp], 11                    ; 0000000bH
        mov     eax, DWORD PTR _i$[ebp]
        push    eax
        push    OFFSET ??_C@_05BKKKKIID@i?$DN?$CFd?6?$AA@
        call    _printf
        add     esp, 8
        mov     DWORD PTR [ebp-4], 16           ; 00000010H
        mov     eax, DWORD PTR _i$[ebp]
        push    eax
        push    OFFSET ??_C@_05BKKKKIID@i?$DN?$CFd?6?$AA@
        call    _printf
        add     esp, 8
        xor     eax, eax
        mov     esp, ebp
        pop     ebp
        ret     0
_main   ENDP
```

输出:

```c
i=11
i=16
```