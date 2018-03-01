---
title: "7 更灵活的定位内存地址的方法"
toc: true
tags: 
- "assembly"

top: 2

categories:
- "计算机技能"
- "汇编基础"
---

#  7.1 `and`和`or`指令

- `and`指令：逻辑与运算指令，按位进行与运算
  - 通过该指令可将操作对象的相应位设为 0 ，其他为不变

```assembly
mov al, 01100011B
and al, 00111011B
```

执行后`al = 00100011B`。



- `or`指令：逻辑或指令，按位进行或运算
  - 通过该指令可将操作对象的相应位设为 1，其他为不变

```assembly
mov al, 01100011B
or al, 00111011B
```

结果为：`01111011B`



#  7.2 关于 ASCII 码

- Ascii 码是一种编码方案
  - 他约定了计算机用此种方案来表示信息
    - 比如`61h`表示`a`， `62h`表示`b`
- 文本编辑过程中，就按照 ASCII 编码规则进行编码和解码
- 键盘输入到屏幕显示的流程
  - 按下`a`键，这个键的信息被送入计算机
  - 计算机使用 ASCII 编码规则将其转换为`61H`存储在内存的指定空间中
  - 文本编辑器软件从内存中取出`61H`，将其送到显卡的显存中
  - 工作在文本模式下的显卡用 ASCII 码的规则解释显存中的内容
  - 显卡驱动器将解释后的内容，也就是`a`的图像画到屏幕上

也就是说，要想在屏幕上显示出`a`，就要给显卡提供`a`的 ASCII 码（61H）。

我们通过向显卡显存写入数据实现。





#  7.3 以字符的形式给出数据

- 在汇编程序中，用单引号括起来的方式指明数据是以字符的形式给出的
  - 编译器将他们转换为相对应的 ASCII 码

```assembly
assume cs:code,ds:data
data segment
	db 'unIX'
	db 'foRK'
data ends
code segment
start:
	mov al, 'a'
	mov bl, 'b'
	mov ax, 4c00h
	int 21h
code ends
end start
```

*解析*：

- `db 'unIX'`相当于：`db 75H,6EH,49H,58H`
  - 因为`u,n,I,X`的 ASCII 码分别为 75H, 6EH, 49H 和 58H
- `mov al, 'a'`相当于：`mov al, 61h`
  - 'a' 的 ASCII 码为`61H`

#  7.4 大小写转换问题

> 在`codeseg`中填写代码，将`dataseg`中的第一个字符串转换为大写，第二个字符串转换为小写

```assembly
assume cs:codeseg, ds:dataseg
dataseg segment
	db 'BaSic'
	db 'iNfOrMaTiOn'
dataseg ends
codeseg segment
start:
codeseg ends
end start
```

- 分析大小写字符的 ASCII 码表规律

![字母 ASCII 码表规律](https://img.ioioi.top/wiki/wiki_201712_7fa815.png)

可以看到两个规律：

1. 小写字母的 ASCII 码总比相应大写字母的大`20H`
2. 无论是哪一个字母，只要是大写的，其二进制第 5 位总是 0，而小写字母第 5 位总是 1

所以思路就是：

- 使用与操作将第一个字符串中所有字符的第 5 位设为 0
- 使用或操作将第二个字符串中所有字符的第 5 位设为 1

```assembly
assume cs:codeseg, ds:dataseg
dataseg segment
	db 'BaSic'
	db 'iNfOrMaTiOn'
dataseg ends
codeseg segment
start:
	mov ax, dataseg
	mov bx, 0
	mov cx, 5
	
s:	mov al, [bx]
	and al, 11011111B
	mov [bx], al
	inc bx
	loop s

	mov bx, 5
	mov cx, 11

s0: mov al, [bx]
	or al, 00100000B
	mov [bx], al
	inc bx
	loop s0

	mov ax, 4c00h
	int 21h
codeseg ends
end start
```



#  7.5 `[bx+data]`

我们之前使用`[bx]`来指定一个内存单元，也可以使用`[bx + idata]`的方式指定内存单元。

- `[bx + idata]`表示一个内存单元，他的偏移地址为`(bx) + idata`
  - 也就是`bx`中的数值加上`idata`

```assembly
mov ax, [bx+200]
```

- 将一个内存单元的内容送入`ax`，这个内存单元长度为 2 个字节（1字单元），存放一个字，偏移地址为`bx`中的数值加上`200`，段地址在`ds`中
- 数字化描述：`(ax) = ((ds)*16 + (bx) + 200)`

也可作一下形式：

```assembly
mov ax, [200+bx]
mov ax, 200[bx]
mov ax, [bx].200
```



#### 补课：关于 8086 中单元、字单元、字节和`D`指令输出的迷思

1. 在`debug`中使用`D`指令输出

![D 输出](https://img.ioioi.top/wiki/wiki_201712_e86146.png)



可以看到，每一排整整齐齐地格式：

- 左边为地址
  - 十六进制格式输出
  - 每一行为 16 的整数倍地址
- 右边为地址区段单元的内容
  - 每两个数字代表一个内存单元，大小为一个字节，也就是 8 位
    - `74`代表`2000:1000H`这个内存单元所存储的值
      - 由两个十六进制的数组成
      - 每一个为 4 位，故而为 8 位
    - `79`代表`2000:1001H`这个内存单元所存储的值
- 8086 中一个字单元为 2 个字节，也就是 16 位

下面语句执行：

```assembly
mov ax, 2000h
mov ds, ax
mov bx, 1000

mov ax, ds:[bx]
```

此时，`ax =7974 `。因为 8086 中，`ax`寄存器为 16 位，那么你使用`mov`指令的话，默认移动 2 个字节，即是 16 位，所以会移动：

- `2000:1000H`
- `2000:1001H`

两个内存单元中的内容，并且先移动，到低位，后移动，到高位。





























