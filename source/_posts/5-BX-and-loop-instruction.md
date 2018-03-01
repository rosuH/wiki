---
title: "5 [BX]和 loop 指令"
toc: true
tags: 
- "assembly"

top: 1

categories:
- "计算机技能"
- "汇编基础"
---

# 前言

## 1. `[bx]`和内存单元的描述

- 描述一个内存单元所需的两种信息
  - 内存单元的地址
  - 内存单元的长度（类型）

`[0]` 表示一个内存单元时， 0 表示单元的偏移地址，段地址默认在 `ds`寄存器中，单元的长度（类型）可以由具体指令中的其他操作对象（比如寄存器）指出。

- `[bx]` 是一个内存单元的地址
  - 它的偏移地址存放在 `bx`中

```assembly
mov ax, [bx]
```

- 表示将一个内存单元的内容送入 `ax`，这个内存单元的长度位 2 字节（字单元），存放一个字节，偏移地址在`bx`中 ，段地址在 `ds`中



## 2. `loop`

循环指令。



## 3. 我们定义的描述性符号：`()`

为了描述上的简洁，在后续的学习中，我们将使用`()`来表示一个寄存器或一个内存单元中的内容：

```assembly
(ax) ; 表示 ax 中的内容
(al) ; 表示 al 中的内容
```



## 4. 约定符号 `idata`表示常量

```assembly
mov ax, [idata] ; 代表 mov ax,[1]、mov ax,[2]、或 mov ax,[3] 等
mov bx, idata	; 代表 mov bx 1、mov bx, 2 或 mov bx, 3 等
```



# 5.1 `[BX]`

2 个示例：

```assembly
mov ax, [bx]
```

- `bx`中存放的数据作为一个偏移地址`EA`，段地址`SA`默认在`ds`中，将`SA:EA`处的数据送入`ax`中。即：`(ax) = ((ds)* 16 + (bx))`。

```assembly
mov [bx], ax
```

- `bx`中存放的数据作为一个偏移地址`EA`，段地址`SA`默认在`ds`中，将`ax`中的数据送入内存`SA:EA`处。即：`((ds)*16 + (bx) = ax)`



#  5.2 `Loop`指令

- `loop`指令的格式

```assembly
loop 标号
```

- `loop`被执行的过程

  - `(cx) = (cx) - 1`
  - 判断`cx`中的值，如果不为零则转至标号处执行程序，如果为零则向下执行
  - `cx`中存放循环次数


## 示例一

- 任务：编程计算`2^2`，结果存放在`ax`中
- 分析
  - 设`(ax) = 2`，可计算`(ax) = (ax) * 2`，最后`(ax)`中为`2^2`的值
  - `N*2` 可用`N+N`实现

程序如下：

```assembly
assume cs:code
code segment
	mov ax, 2
	add ax, ax
	
	mov ax, 4c00h
	int 21h
code ends
end
```



## 示例二

- 任务：计算 `2^3`
- 分析：
  - `2^3 = 2*2*2`，若设`(ax) = 2`，可计算`(ax) = (ax) * 2 * 2`，最后`(ax)`中为`2^3`的值
  - `N*2`可用`N+N`实现

```assembly
assume  cs:code
code segment
	mov ax, 2
	add ax, ax
	add ax, ax
	
	mov ax, 4c00h
	int 21h
code ends
end
```



##  示例三

- 任务：编程计算`2^12`
- 分析
  - 类似之前的例子

```assembly
assume cs:code
code segment
	mov ax, 2
	; 作 11 次 add ax, ax
	mov ax, 4c00h
	int 21h
code ends
end

```



我们可以使用`loop`来简化我们的程序。

```assembly
assume cs:code
code segment
	mov ax, 2
	mov cx, 11
s: 	add ax, ax
	loop s
	mov ax, 4c00h
	int 21h
code ends
end
```



*分析*：

- 标号`s`
  - 这个标号标识了一个地址，这个地址处有一条指令`add ax, ax`
- `loop s`
  - CPU **执行`loop s`的时候**，要进行两步操作
    - `(cx) = (cx) - 1`
    - 判断`cx`中的值，不为`0`则转至标号`s`所标识的地址；如果为零则执行下一条
- 以下 3 条指令

```assembly
	mov cx, 11
s:  add ax, ax
	loop s
```

- 执行`loop s`时，首先要将`(cx)`减 1，然后若`(cx)`不为 0，则向前转至`s`处执行`add ax, ax`
  - 利用`cx`可以用来控制`add ax,ax`的执行次数


#  5.3 在 Debug 中跟踪用`loop`指令实现的循环程序

- 问题

计算`ffff:0006`单元中的数乘以 3，结果存储在`dx`中。

- 考虑三个事项
  - 运算后结果会否超出`dx`所能存储的范围？
    - `ffff:0006`单元中的数是一个字节型的数据，范围是`0~255`之间，则用它和 3 相乘结果不会大于 65535，可以在`dx`中存放下
  - 用循环累加来实现乘法，用哪个寄存器进行累加
    - 将`ffff:0006`单元中的赋值给`ax`，用`dx`进行累加。
    - 先设`(dx) = 0`，然后作 3 次`(dx) = (dx)+(ax)`
  - `ffff:0006`单元是一个字节单元，`ax`是一个 16 位寄存器，数据长度不一样，如何赋值？
    - 要想实现`ffff:0006`单元向`ax`赋值，应该令`(ah) = 0, (al) = (ffff6h)`
- 代码实现

```assembly
assume cs:code
code segment
	mov ax, 0ffffh
	mov ds, ax
	mov bx, 6		; 以上，设置 ds:bx 指向 ffff:6
	
	mov al, [bx]
	mov ah, 0		; 以上，设置(al)=((ds*16)+(bx)), (ah)=0
	
	mov dx, 0		; 累计寄存器清 0
	
	mov cx, 3		; 循环 3 次 
s:	add dx, ax
	loop s			; 以上累加计算(ax)*3
	
	mov ax, 4c00h
	int 21h			; 返回
code ends
end
```

- 在汇编源程序中，数据不能以字母开头，所以要在前面加  0


对上面的程序编译、链接之后，使用`debug.exe`对程序执行过程进行观察。

- `u`查看被`Debug`加载入内存的程序可以看到`loop s`语句

```assembly
# 源代码语句
s:	add  dx, ax
	loop s
# 载入内存的语句
0B3D:0012 ADD DX,AX
0B3D:0014 LOOP, 0012
```

可以看到`s`已经被具体的地址所替代。

- 访问内存地址时，`Debug`的显示方式

```assembly
0B44:0008 8A07	MOV AL,[BX]			DS:0006=35	
```

可以看到`debug`右边出现了内存地址`[bx]`中的内容。根据我们设置的，段地址`ds = ffff, bx=6`，所以我们可知道`ffff:6`内存单元中的值为`35h`。

接着就是`loop`指令的执行，注意`cx`寄存器中的值，当该值等于 0 时，就跳出循环。



现在我们知道，修改`cx`寄存器中的内容可以间接实现乘法算术，对于在`debug`中的跟踪技巧而言，我们可以使用`g`指令，来快速执行到某一指令。

```shell
g 0012
```

这样就可以略过`ds:0012`之前的代码了。对于循环语句，就可以直接将地址改为循环后一句地址即可：

```shell
g 0016	# 假设循环语句地址为 0014
```

此外，我们也可以使用`p`指令来自动执行循环。

*遇到`int 21h`语句时应该使用`p`指令来正常结束程序哦。*





#  5.4 `Debug`和汇编编译器`masm`对指令的不同处理

- 在汇编源程序中，如果用指令访问一个内存单元，则在指令中必须用`[...]`来表示内存单元；如果在`[]`里用一个常量`idata`直接给出内存单元的偏移地址，就要在`[]`的前面显式地给出段地址所在的寄存器

```assembly
mov al, ds:[0] 	
; 如果没有给出 ds ，编译器 masm 就会把指令中的 [iadata] 解释为 idata
```

- 如果在`[]`里用寄存器，比如`bx`，间接给出内存单元的偏移地址，则段地址默认在`ds`中
  - 也可以显式地给出段地址所在的段寄存器

#  5.5 `loop`和`[bx]`的联合应用

> 考虑这样一个问题，计算`ffff:0 ~ ffff:b`单元中的数据的和，结果存储在`dx`中。

一样的要考虑三个问题：

- 运算后结果是否会超出`dx`所能存储的范围？

`ffff:0~ffff:b`内存单元中的数据是字节型数据，范围在`0~255`之间，12个这样的数据相加，结果不会大于`65535`，可以在`dx`中存放下。

- 我们能否将`ffff:0~ffff:b`中的数据直接累加到`dx`中？

不行。`ffff:0~ffff:b`中的数据是 8 位的，不能直接加到 16 位寄存器`dx`中。

- 我们能否将`ffff:0~ffff:b`中的数据累加到`dl`中，并设置`(dh)=0`，从而实现累加到`dx`中？

不行。`dl`是 8 位寄存器，能容纳的数据范围在`0~255`，`ffff:0~ffffLb`中的数据也是 8 位，如果仅向`dl`中累加 12 个 8 位数据，很有可能造成进位丢失的后果。

- 如何将`ffff:0~ffff:b`中的 8 位数据，累加到 16 位寄存器`dx`中呢？

矛盾点在于：

- 类型不匹配
  - `(dx)=(dx) + 内存中的 8 位数据`：类型不匹配
- 结果的超界问题
  - `(dl)=(dl) + 内存中的 8 位数据`：结果可能超界

####  解决方法

用一个 16 位寄存器作为中介。将内存单元中的 8 位数据赋值到一个 16 位寄存器`ax`中，再将`ax`中的数据加到`dx`上，从而使得两个运算对象的类型匹配并且结果不会超界。



#### 代码实现

```assembly
assume cs:code
code segment
	mov ax, 0ffffh
	mov ds,ax
	
	mov dx, 0
	
	mov al,ds:[0]
	mov ah,0
	add dx,ax
	
	mov al,ds[1]
	mov ah,0
	...			;直到 ds:[0bh]
	
	mov al, ds:[0bh]
	mov ah, 0
	add dx, ax
	
	mov ax, 4c00h
	int 21h
code ends
end
```



我们可以使用`loop`指令来优化代码：

```assembly
assume cs:code
code segment
	mov ax, 0ffffh
	mov ds, ax
	mov bx, 0		; 初始化 ds:bx 指向 ffff:0
			
	mov dx, 0		; 初始化累加器 dx, (dx) = 0
	
	mov cx, 12		; 初始化循环技术寄存器 cx, (cx) = 12
	
s:	mov al, [bx]
	mov ah, 0
	add dx, ax		; 间接向 dx 中加上 ((ds) * 16 + (bx)) 单元中的值
	inc bx			; ds:bx 指向下一个单元
	loop s
	
	mov ax, 4c00h
	int 21h
code ends
end
```





# 5.6 段前缀

- `mov ax, [bx]`中，内存单元的偏移地址由 `bx`给出，而段地址默认在`ds`中
  - 我们可以在访问内存单元的指令中显式地给出内存单元的段地址所在的寄存器

1. ​

```assembly
mov ax, ds:[bx]
```

将一个内存单元的内容送入`ax`，这个内存单元的长度为 2 字节（字单元），存放在一个字，偏移地址在`bx`中，段地址在`ds`中。

2. ​

```assembly
mov ax, cs:[bx]
```

将一个内存单元的内容送入`ax`，这个内存单元的长度为 2 字节（字单元），存放一个字，偏移地址在`bx`中，段地址在`cs`中。

3. ​

```assembly
mov ax, ss:[bx]
```

将一个内存单元的内容送入`ax`，这个内存单元长度为 2 字节（字单元），存放一个字，便宜地址在`bx`中，段地址`ss`中。

4. ​

```assembly
mov ax, es:[bx]
```

将一个内存单元的内容送入`ax`，这个内存单元长度为 2 字节（字单元），存放一个字，便宜地址在`bx`中，段地址`es`中。

5. ​

```java
mov ax,ss:[0]
```

将一个内存单元的内容送入`ax`，这个内存单元长度为 2 字节（字单元），存放一个字，便宜地址为 0 ，段地址`ss`中。

6. ​

```assembly
mov ax,cs:[0]
```

将一个内存单元的内容送入`ax`，这个内存单元长度为 2 字节（字单元），存放一个字，便宜地址为 0 ，段地址`cs`中。

上述文字中出现的`ds:`，`cs:`， `ss:`，`es:`，在汇编语言中称为段前缀。



#  5.7 一段安全的空间

在 8086 模式中，随意向一段内存地址空间写入内容是危险的，因为这段空间中可能存放着重要的系统数据或代码。即便是许多高级语言，也需要提前申请内容才能使用。

书中例子举出，在实模式（纯 DOS）模式下执行`mov [0026],ax`将会引起死机，因为地址`0:0026`存放着重要数据。

实际上，更多的情况我们都会在 CPU 实模式下进行编程。这样的硬件已被操作系统利用 CPU 保护模式所提供的功能全面而严格地管理着。

在后续的编程实践中，我们需要向内存直接写入内容，但是又不希望发生危险事故。所以我们需要找到一段安全的空间供我们使用。

- 在一般的 PC 机中，DOS 方式下，DOS 和其他合法的程序一般不会使用`0:200~0:2ff`的 256 个字节的空间

使用`deubg`查看`0:200~0:2ff`内存段如果都是 0 的话就表示没有人用过了。



#  5.8 段前缀的使用

考虑一个问题：将内存`ffff:0~ffff:b`单元中的数据复制到`0:200~0:20b`单元中。

*分析*：

- `0:200~0:20b`单元等同于`0020:0~0020:b`单元，他们描述的是同一段内存空间


- 复制的过程应用循环实现

```assembly
; 初始化
x = 0
; 循环 12 次
; 将 ffff:x 单元中的数据送入 0020:x 
X = X + 1
```

- 在循环中，原始单元`ffff:x`和目标单元`0020:x`的偏移地址`X`是变量
  - 我们使用`bx`来存放
- 将`0:200~0:20b`用`0020:0~0020:b`来描述，就是为了使目标单元的偏移地址和源始单元的偏移地址从同一数值 0 开始

程序如下：

```assembly
assume cs:code
code segment
	mov bx, 0		;(bx) = 0, 偏移地址从 0 开始
	mov cx, 12		;(cx) = 12, 循环 12 次

s:	mov ax, 0ffffh	
	mov ds, ax		;(ds) = 0fffh
	mov dl, [bx]	;(dl) = ((ds) * 16 + (bx)), 将 ffff:bx 中的数据送入 dl
	
	mov ax,0020h
	mov ds,ax		;(ds) = 0020h
	mov	[bx],dl		;((ds) * 16 + (bx)) = (dl), 将 dl 中数据送入 0020:bx
	
	inc bx			; (bx) = (bx) + 1
	loop s
	mov ax, 4c00h
	int 21h
code ends
end
```



*解析*：

- 源始单元`ffff:x`和目标单元`0020:x`相距大于 64KB ，在不同的`64KB`段里，程序需要设置两次`ds`
- 我们可以使用两个段寄存器分别存放源始单元`ffff:x`和目标单元`0020:x`的段地址

这样就可以省略需要重复 12 次的设置 `ds`程序段：

```assembly
assume cs:code
code segment
	mov ax, 0ffffh
	mov ds, ax
	
	mov ax, 0020h
	mov es, ax
	
	mov bx, 0
	
	mov cx, 12
	
s: 	mov dl,[bx]
	mov es:[bx], dl
	inc bx
	loop s
	
	mov ax,4c00h
	int 21h
code ends
end
```







#  实验四 `[BX]`和`loop`的使用

#### 题目一

> 编程，向内存`0:200~0:23f`依次传送数据`0~63(3FH)`

```assembly
assume cs:code
code segment
	mov ax, 0
	mov ds, ax
	mov bx, 0200h		;使 ds:bx=0:200h
	
	mov cx, 64			; 初始化循环计数器
	mov dx, 0

s:  mov [bx], dx
	inc bx
	inc dx
	loop s
	
	mov ax,2c00h
	int 21h
code ends
end
```

使用`ml`编译链接之后，使用`debug`调试即可。



####  题目二

> 编程，向内存`0:200~0:23f`依次传送数据`0~63(3FH)`，程序只能使用 9 条指令，9 条指令中包括`mov ax, 4c00h`和`int 21h`

















