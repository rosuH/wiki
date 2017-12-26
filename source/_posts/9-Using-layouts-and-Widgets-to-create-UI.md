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

可视化界面的编辑属性功能十分易用。选中组件后即可编辑。



####  动态设置列表项

为了有选择性地显示一些组件（此处是手铐图片），我们可以通过修改`ImageView`组件解决。

- 更新`ImageView`组件的 ID
- 在组件树里选中它，然后在视图属性窗口将 ID 修改为`crime_solved`

接下来要在代码中使用这个组件。

*控制图片显示（`CrimeListFragment.java`）*

```java
public class CrimeListFragment extends Fragment {
 ...
    private class CrimeHolder extends RecyclerView.ViewHolder
            implements View.OnClickListener{

        private  Crime mCrime;
        private TextView mTitleTextView;
        private TextView mDateTextView;
        private ImageView mSolvedImageView;

        public CrimeHolder(LayoutInflater inflater, ViewGroup parent) {
            super(inflater.inflate(R.layout.list_item_crime, parent,false));

            mTitleTextView = (TextView)itemView.findViewById(R.id.crime_title);
            mDateTextView = (TextView)itemView.findViewById(R.id.crime_date);
            mSolvedImageView = (ImageView)itemView.findViewById(R.id.crime_solved);
            itemView.setOnClickListener(this);
        }

        public void bind(Crime crime) {
            mCrime = crime;
            mTitleTextView.setText(mCrime.getTitle());
            mDateTextView.setText(mCrime.getDate().toString());
            mSolvedImageView.setVisibility(crime.isSolved() ? View.VISIBLE : View.GONE);
        }

    ...
}

```

可以看到，此处书里的做法是：

- 在`crime`数据中存放一个标志`solved`
  - `crimeLab`中使用`crime.setSolved(i % 2 == 0)`将偶数项的`solved`置为`1`
- 在`bind`中判断是否需要将`mSolvedImageView`设置为`View.GONE`

这样的方法比我之前自己想的使用`RecyclerView.viewType`的方法要方便、简洁和易操作。





#  9.3 深入学习布局属性



####  `dp`，`sp`以及屏幕像素密度

- 常见的视图属性值
  - 文字大小：指定设备上显示的文字的像素高度
  - 边距：指定视图组件间的举例
  - 内边距：指定视图外边框与其内容之间的举例
- 不同设备之间的兼容问题
  - Android 提供了与密度无关的尺寸单位，应用这种单位可在不通的屏幕像素密度的设备上获得相同的尺寸
- 常见的单位
  - `px`
    - 英文 pixel 的缩写，即像素
    - 无论屏幕密度多少，一个像素单位对应一个屏幕像素单位
    - **不推荐使用，因为他不会根据屏幕密度自动缩放**
  - `dp (dip)`
    - 英文 density-independent pixel 的缩写，意为**密度无关像素**
    - 在设置边距、内边距或任何不打算按像素值指定尺寸的情况下，通常都使用`dp`这种单位
    - 如果屏幕密度较高，密度无关像素会相应扩展到整个屏幕
    - **无论屏幕密度任何，总能获得同样的尺寸**
  - `sp`
    - 英文 cale-independet pixel 的缩写，意思为缩放无关像素
    - 它是一种与密度无关的像素，这种像素会受用户字体偏好设置的影响
    - 通常使用`sp`来设置屏幕上的字体大小
  - `pt, mm, in`
    - 类似于`dp`的缩放单位，允许以点、毫米或尺寸为单位指定用户界面尺寸
    - 在实际开发中不建议使用这些单位，因为并不是所有设备都能按照这些单位进行正确的尺寸缩放配置
- 对于我们常用的`sp`和`dp`，Android 会在运行时自动将他们转换为像素单位



####  边距与内边距

- **边距属性是布局参数**
  - 决定了组件之间的举例
  - 组件对外界一无所知，边距必须由该组件的父组件负责
- **内边距不是布局参数**
  - `android:padding`告诉组件：在绘制组件自身时，要比所含内容大多少



####  样式、主题及主题属性

- 样式 (style) 是`XML`资源文件，含有用来描述组件行为和外观的属性定义
- 可以自己创建自己的样式文件
  - 然后在布局文件中以`@style/my_own_style`的形式引用
- 例子中的两个`TextView`组件都有一个引用 Android 自带样式文件的`style`属性
  - 该预定义样式来自于应用主题
- 主题是各种样式的集合
  - 从结构上来说，主题本身也是一种资源，只不过他的属性指向了其他样式资源
  - 使用主题属性引用，可将预定义的应用主题样式添加给指定组件
    - 比如`?android:listSeparatorTextViewStyle`
    - 使用主题属性引用就是告诉 Android 运行资源管理器：“在应用主题里找到`listSeparatorTextViewStyle`的属性。该属性指向其他样式资源，请将其资源的值放在这里”
  - 所有 Android 主题都包括名为`listSeparatorTextViewStyle`的属性，不过他们的定义略有不同
  - 使用主题引用，可以确保`TextView`组件在应用中拥有正确一致的显示风格



####  Android 应用的设置原则

[Material Design](https://material.io/guidelines/)



#  9.4 图形布局工具使用小结



图形化很好用，直接编写`XML`也很不错，哪个方便就用哪个。













