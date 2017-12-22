---
title: "9 使用布局和组件创建用户界面"
date: {{ date }}
tags: 
  - "Android"
  - "RecyclerView"

top: 9

categories: 
  - "Android 开发"
---

# 9.1 使用图形布局工具

除了手动输入 XML 的方式创建，我们还可以使用图形布局工具来创建布局。

图形布局工具界面：

- 中间区域是界面预览窗口
- 右边是蓝图视图，可以看出各组件视图的轮廓、大小和比例
- 左边是组件面板
- 左下是组件树
- 右边是属性界面

#  9.2 引入`ConstraintLayout`

- `ConstraintLayout`工具可以为布局添加一系列约束
  - 组件位置关系因为约束而发生变化
- 在`ConstraintLayout`里布置视图，只需要添加约束就可以
  - 组件的大小调整方式有三
    - 组件自己决定（`wrap_content`
    - 手动调整
    - 让组件充满约束布局

####  引入步骤

1. 切换到`design`布局
2. 右击要转换的布局
3. 选中`convert ... to ConstraintLayout`即可

现在 Android Studio 会自动添加依赖，所以不需要手动家加入。



####  约束编辑器

在布局预览窗口顶部的工具栏上，有一些约束选项：

- 显示所有约束
- 自动连接切换开关
  - 启动后，在预览界面拖移视图时，约束会自动配置
  - Android Studio 会猜测你的布局配置意图，然后自动连接
- 清除全部约束
- 猜测约束
  - 类似自动连接
  - Android Studio 会自动帮你创建约束

我们先手动清除根视图的所有约束，然后一步一步添加回来。

#### 腾出空间

选中某个组件，右边就可以编辑该组件的属性了。

![TextView 的属性](https://img.rosuh.me/wiki/wiki_201712_b86613.png)

- 组件的水平和竖直方向的尺寸是分别由宽度设置和高度设置决定的

能设置的值有以下三种：

![三种视图尺寸设置](https://img.rosuh.me/wiki/wiki_201712_d75c2a.png)

| 设置类型 |     设置值      |            用法            |
| :--: | :----------: | :----------------------: |
| 固定大小 |     Xdp      |    以 dp 为单元，为视图指定固定值     |
| 包裹内容 | wrap_content | 设置视图想要的尺寸（随内容走），大到足够容纳内容 |
| 动态适应 |     0dp      |      允许视图缩放以满足指定约束       |



至于添加约束的方式，其实就是拉动组件轮廓四周的四个圆圈，然后吸附到另一个组件轮廓上。

![设置约束](https://img.rosuh.me/wiki/wiki_201712_bcea44.png)

####  约束的 XML 格式

刚刚添加了一个`ImgaeVIew`组件，并为之三个边添加了约束，使之呆在布局的右边。

我们来看看刚刚添加的组件的约束 XML 代码：

```xml
<ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="8dp"
        android:layout_marginEnd="8dp"
        android:layout_marginTop="8dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:srcCompat="@drawable/ic_solved"/>
```

可以看到约束相关的属性皆是`app:layout_constraint`开头的：

```xml
app:layout_constraintBottom_toBottomOf="parent"
app:layout_constraintEnd_toEndOf="parent"
app:layout_constraintTop_toTopOf="parent"
```



- 属性以`layout_`开头皆属于布局参数（`layout parameter`）
  - 组件的布局参数是用来向其父组件做指示用的：用来告诉父布局如何安排自己

想想之前那个`layout_margin`，还有`layout_height`，都是布局参数。

- 约束的名字是`constraintTop`，这表示它是`ImageView`的顶部约束
- 属性以`toTopof="parent"`结束，表示约束是连接到父组件的



####  编辑属性





























