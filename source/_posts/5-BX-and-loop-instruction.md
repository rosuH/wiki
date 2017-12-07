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


















