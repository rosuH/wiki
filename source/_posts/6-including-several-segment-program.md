---
title: "6 包含多个段的程序"
toc: true
tags: 
- "assembly"

top: 2

categories:
- "计算机技能"
- "汇编基础"
---

#  前言

- 在操作系统环境中，合法地通过操作系统取得的空间都是合法的
  - 在操作系统允许的情况下，程序可以取得任意容量的空间
- 程序取得所需空间的方法
  - 加载程序时候为程序分配
    - 在源码这种做出说明
    - 通过定义段来获取内存空间
  - 程序执行过程中向系统申请

在本课程中，我们只讨论第一种。

- 在一个段中存放数据、代码和栈，我们先来体会一下不使用多个段的情况
- 将数据、代码和栈放入不同的段中





# 6.1 在代码段中使用数据

> 编程计算下面 8 个数据的和，结果存储在`ax`寄存器中
>
> `0123h, 0456h, 0789h, 0abch, 0defh, 0fedh, 0cbah, 0987h`

思路是将上述 8 个数据存放在一段连续的内存空间中然后使用循环累加最后将结果存放到`ax`中。

我们无法自己随意决定使用哪一段内存空间。所以我们需要在源码中向系统申请内存空间。

- 定义我们希望处理的数据
- 编译器编译这些数据、链接将之作为程序的一部分写入到可执行文件中
- 可执行文件被载入内存，这些**数据也同时被加载入内存中**

```assembly
assume cs:code
code segment
	dw 0123h, 0456h, 0789h, 0abch, 0defh, 0fedh, 0cbah, 0987h
	
	mov bx, 0
	mov ax, 0
	
	mov cx, 8

s:	add ax,cs:[bx]
	add bx, 2
	loop s
	
	mov ax, 4c00h
	int 21h
code ends
end
```

*解析*：

- `dw`（define word）：定义字型数据
  - 这里定义了 8 个字型数据，他们一共占据 16 字节的内存空间
- 字型数据的位置
  - 代码段：因为他们在代码段中，所以他们的段地址可以通过`CS`寄存器获得
  - 偏移量：因为他们处于代码段的最开始，所以偏移地址为 0
- `bx`寄存器通过自加 2 来移动到下一个字型数据的位置

使用`dw`定义的数据，在内存中也被以二进制的形式存放进去。通过`debug`调试查看上述程序。

`u`指令：

![U 指令](https://img.rosuh.me/wiki/wiki_201712_42be46.png)

可以看到，在`0b3d:0013`以前的代码几乎是“看不懂”的。因为这一段存储的是原始的二进制数据，所以`debug`无法解析出对应的汇编语句。

当我们使用`d`语句就可以看到该代码段存放的其实就是那 8 个字型数据了。

![原始二进制数据](https://img.rosuh.me/wiki/wiki_201712_035d52.png)



当然这个时候你就不能直接使用`t`指令来执行语句了。正如前面所说的，`CS:IP`指向了程序的开头，也就是存放`dw`定义的字型数据的地方，哪些地方根本不对应那些汇编语句。

我们需要手动将`IP`设置为`10h`，从而使`CS:IP`执行程序中的第一条指令。接着再用`t`或`p`或`g`命令执行。



这样的程序有显而易见的缺漏：

- 我们必须使用`debug`来启动程序
  - 在系统中直接运行可能会出现问题，因为程序的入口并不是我们期望的指令

现在我们来解决程序入口问题：

```assembly
assume cs:code
code segment
	dw 0123h, 0456h, 0789h, 0abch, 0defh, 0fedh, 0cbah, 0987h
	
start: mov bx, 0
	mov ax, 0
	
	mov cx, 8

s:	add ax,cs:[bx]
	add bx, 2
	loop s
	
	mov ax, 4c00h
	int 21h
code ends
end start
```

*解析*：

- 程序的入口指令面前加上`start`
  - 伪指令`end`后面同样加上`start`
- `end`的作用
  - 通知编译器程序结束
  - 通知编译器程序的入口在什么地方
- 单任务系统中，可执行文件的程序执行过程如下
  - 由其他程序(`Debug`, `command`或其他程序)将可执行文件中的程序加载入内存
  - 设置`CS:IP`指向程序的第一条要执行的指令（程序的入口），从而使程序得以运行
    - 根据可执行文件中的描述信息指明哪一条指令是第一条指令
    - 源程序中的“描述信息”部分，合乎上面中的`end`相关语句；易言之，`end start`指明程序的入口，被转化为一个入口地址，存储在可执行文件的描述信息中
  - 程序结束运行之后，返回到加载者

现在我们可以这样安排程序框架：

```assembly
assume cs:code
code segment
	...
	data
	...
start:
	...
	code
	...
code ends
end start
```





#  6.2 在代码段中使用栈

> 完成下面程序，利用栈，将程序中定义的数据逆序存放
>
> ```assembly
> assume cs:codeseg
> codeseg segment
> 	dw 0123h, 0456h, 0789h, 0abch, 0defh, 0fedh, 0cbah, 0987h
> 	?
> codeseg ends
> end
> ```
>
> 

*思路*：

- 程序运行时，定义数据存放在`cs:0~cs:F`单元中，共  8 个字单元
- 依次将这 8 个字单元中的数据入栈，然后再依次出栈到这 8 个字单元中，从而实现逆序存放



要解决的问题就是必须要有一段可以使用的栈空间。我们需要在程序中通过定义数据来取得一段空间，将之当做栈空间来使用。

```assembly
assume cs:codeseg
codeseg segment
	dw 0123h, 0456h, 0789h, 0abch, 0defh, 0fedh, 0cbah, 0987h
	dw 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
	; 用 dw 定义 16 个字型数据，在程序加载后，将取得的 16 个字
	; 内存空间，存放这 16 个数据
	; 在后面的程序中将这段空间当做栈来使用
start:
	mov ax, cs
	mov ss, ax
	mov sp, 30h		; 设置栈顶 ss:sp 指向 cs:30
	
	mov bx, 0
	mov cx, 8
	
s:	push cx:[bx]	; 将 0~15 单元中的 8 个字型数据依次入栈
	add bx, 2
	loop s
	
	mov bx, 0
	mov cx, 8
	
s0:	pop cs:[bx]		; 依次出栈
	add bx, 2
	loop s0
	
	mov ax, 4c00h
	int 21h
codeseg ends
end start
```



使用`dw`的真正目的是“开辟”存储空间，所以不管用什么值去申请都是可以的。申请之后我们在代码中将他作为栈来使用。



#### 检测点 6.1

> 下面的程序实现依次用内存`0:0~0:15`单元中的内容改写程序中的数据，完成程序：

```assembly
assume cs:codeseg
codeseg segment
	dw 0123h, 0456h, 0789h, 0abch, 0defh, 0fedh, 0cbah, 0987h
	
start:
	mov ax, 0
	mov ds, ax
	mov bx, 0
	
	mov cx, 8
s:	mov ax, [bx]
	_________
	add bx, 2
	loop s
	
	mov ax, 4c00h
	int 21h
codeseg ends
end start
```

填入的答案应该是`mov cs:[bx], ax`。

- `cs:ip`指向的是下一条指令的位置
  - `cs`存储着代码代码段，而`[bx]`表示偏移量，也正是代表着这个程序在内存空间中最开始的那一段



> 下面的程序实现依次用内存`0:0~0:15`单元中的内容改写程序中的数据，数据的传送用栈来进行。栈空间设置在程序内。完成程序：

```assembly
assume cs:codeseg
codeseg segment
	dw 0123h, 0456h, 0789h, 0abch, 0defh, 0fedh, 0cbah, 0987h
	dw 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0

start:
	mov ax, ____
	mov ss, ax
	mov sp, ____
	
	mov ax, 0
	mov ds, ax
	mov bx, 0
	mov cx, 8

s:	push [bx]
	_________
	add bx, 2
	loop s
	
	mov ax, 4c00h
	int 21h
codeseg ends
end start
```



*思路*：

- 继续我们上一个例子的想法，本程序源码存放在`cs`代码段，在内存中，排在前面的是两段我们申请空间，大小分别为 16 个字节和 20 个字节
  - 前者是数据段，后者用来做栈
  - 栈内空闲时，指针指向栈底部，所以`sp = 36 = 16+20 ` --> `sp = 0024h`
- 要用`0:0~0:15`的数据将我们定义的数据段内的数据重写

步骤是：

- 将 `0:0~0:15`空间数据依次入栈，后出栈到在`cs`段的数据

故而完整程序为：

```assembly
assume cs:codeseg
codeseg segment
	dw 0123h, 0456h, 0789h, 0abch, 0defh, 0fedh, 0cbah, 0987h
	dw 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0

start:
	mov ax, cs		; 指向程序存放的代码段，以便找到数据段和栈段
	mov ss, ax
	mov sp, 24h		;设置栈底
	
	mov ax, 0
	mov ds, ax		; 指向要修改的目标内存地址
	mov bx, 0
	mov cx, 8

s:	push [bx]		; 将 0:0~0:15 数据依次入栈
	pop  cs:[bx]	; 将栈内数据弹出到我们定义的数据段内
	add bx, 2
	loop s
	
	mov ax, 4c00h
	int 21h
codeseg ends
end start
```



- 参见 [检测点6.1](http://www.cnblogs.com/Base-Of-Practice/articles/6883898.html)



















