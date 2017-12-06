---
title: "7 UI Fragmnet 与 fragment 管理器"
date: {{ date }}
toc: true

tags: 
  - Android

top: 7

categories: 
  - "Android 开发"
---

# 7.1 UI 设计的灵活性需求

- 复杂的用户界面呈现需求
  - 平板以及大尺寸手机显示问题
  - 滑动屏幕等交互问题
- acticity 视图局限性
  - 我们需要 activity 界面可以在运行时组装甚至时重新组装，但是其本身并不具备这样的灵活性
  - activity 还得和特定的用户界面紧紧绑定



# 7.2 引入 fragment

- fragment 是一种控制器对象
  - activity 可以委托它执行任务
    - 这些任务通常是管理用户界面
    - 受管的用户界面可以时一整屏或时整屏的一部分
  - 采用 fragmnet 来管理 UI，可以绕开 android 系统 activity 使用规划的限制
- 管理用户界面的 fragment 又称为 UI fragment
  - 它自己也有产生于布局文件的视图
  - fragment 视图包含了用户可以交互的可视化 UI 元素
- activity 视图能预留位置供 fragment 视图插入
  - 多个 fragment 则需要预留多个位置以供插入
- activity 视图和 fragment 的关系和切换如下

![明细 fragment 的切换](http://img.rosuh.me/wiki/wiki_2017_1.png)

- fragment 可以带来灵活多变的布局，代价就是复杂的应用、更多的组件以及大量实现的代码



# 7.3 着手开发示例应用

*CriminalIntent 就是本示例应用的名称咯。*

我们先来梳理一下开发流程。

##### 我们先来看一下整个`CriminalIntent`项目的对象图解，以便我们更好地理解开发流程

![项目对象图解](http://img.rosuh.me/wiki/wiki_2017_d.png)

**解析**：

- `CrimeFragment`的作用与`activity`在`GeoQuiz`应用中的作用差不多，都是负责创建并管理用户界面，以及与模型对象进行交互
- `Crime`实例代表某种办公室陋习
  - `crime`有一个标题、一个标志 ID，一个日期和一个布尔值
  - 布尔值用来表示陋习是否被解决
  - 简单起见，本章使用一个`Crime`实例，并将其存放在`CrimeFragment`类的成员变量`mCrime`中
- `CrimeActivity`视图（其对应的`.xml`文件）由`FragmentLayout`组件组成，`FragmentLayout`组件为`CrimeFragment`视图安排了显示位置
- `CrimeFragment`视图由一个`LineaLayout`组件及其三个子视图组成；`CrimeFragment`类中由存储它们的成员变量，并设有监听器，会响应用户操作，更新模型数据
  - `EditText`
  - `Button`
  - `CheckBox`

#### CrimeFragment

- 首先设计一个名为`CrimeFragment`的 UI  fragment 来管理用户界面
- 再设计一个名为`CrimeActivity`的 activity 来托管`CrimeFragment`实例
  - activity 在其视图层内提供一处位置，用来放置`fragment`视图
  - `fragment`视图本身没有在屏幕上显示视图的能力；只有将它放置在 activiti 视图层级结构中，`fragment`视图才能显示在屏幕上

![CrimeActivity 托管 CrimeFragment](http://img.rosuh.me/wiki/wiki_2017_4.png)

#### 两类 fragment

- 原生版本的`fragment`
  - 内置于设备系统中，如果应用要支持各个系统版本，在不同设备上运行的`fragment`可能会有不同的表现（因为各个版本的维护有差异）
- 支持库里的`fragment`
  - 发布时，内置于应用中；使用支持库的`fragment`在不同设备上都会由相同的表现
  - 我们使用的支持库版本来自`AppCompat`库

一般选用支持库中的`fragment`实现，因为考虑到`fragment API`不断引入新特性以及支持库不断更新的现状。



####  在 Android Studio 中增加依赖关系 

要使用`AppCompat`库，项目必须加入依赖关系：

- 打开应用模块的`build.gradle`文件
  - `app/build.gradle`

```java
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:26.1.0'
}
```

*Android Studio 在 `build.gradle`中将原来的 `compile`改为了 `api`和`implementation`。*

参看：[Why Android change 'compile' task to 'implementation' task in gradle build?](https://stackoverflow.com/questions/44402024/why-android-change-compile-task-to-implementation-task-in-gradle-build)



接着我们先创建模型层的`Crime`类。



####  创建`Crime`类

```java
public class Crime {
    private UUID mId;
    private String mTitle;
    private Date mDate;
    private boolean mSolved;
    
    public Crime(){
        mId = UUID.randomUUID();
        mDate = new Date();
    }
}
// 略去 getter() 和 setter() 方法

```

*解析*：

- `UUID`类是 Android 框架里的 Java 工具类
  - 在构造方法里，调用`UUID.randomUUID()`会产生一个随机唯一 ID 值
- 使用默认的`Date`构造方法初始化`DAte`变量
  - 作为`crime`的默认发生时间，设置`mDate`变量值为当前日期



#  7.4 托管`UI fragment`

为了托管`UI fragment`，`activity`必须：

- 在其布局中为`fragment`的视图安排位置
- 管理`fragment`实例的生命周期



####  `fragment`的生命周期

下图展示了`fragment`的生命周期：

![fragment 的生命周期图解](http://img.rosuh.me/wiki/wiki_2017_724ee3.png)



- `fragment`的生命周期类似于`activity`的生命周期，它具有停止、暂停以及运行状态，也拥有覆盖方法，用来在一些关键节点完成一些任务
- `fragment`生命周期和`activity`的方法的对应关系
  - 因为`fragment`代表`activity`工作，所以它的状态应该反映`activity`的状态
- `fragment`生命周期与`activity`生命周期的一个关键不同在于：
  - `fragment`的生命周期方法由托管`activity`而不是操作系统调用的
  - 操作系统不关心`activity`用来管理视图的`fragment`；易言之，`fragment`的使用是`activity`内部的事情



####  托管的两种方式

`activity`托管`UI fragment`有如下两种方式：

- 在`activity`布局中添加`fragment`
  - 使用布局`fragment`
  - 简单但不灵活：在`activity`布局中添加`fragment`，就等同于将`fragment`及其视图与`activity`的视图绑定在一起，并且在`activity`的生命周期过程中，无法替换`fragment`视图
- 在`activity`代码中添加`fragment`
  - 比较复杂，但是也是唯一可以动态控制`fragment`的方式
  - 何时添加`fragment`以及随后可以完成何种具体任务由你自己决定；也可以移除、替换和重新添加当前`fragment`等等

为了追求真正灵活的 UI 设计，就必须通过代码的方式添加`fragment`。

接下来我们将定义`CrimeActivity`的布局。



####  定义容器布局

尽管我们选择的是在`activity`代码中添加`UI fragment`，但是我们依旧**要在`activity`视图层级结构中为`fragment`视图安排位置**。

在`CrimeActivity`的布局中，该位置就是下图所示的`FrameLayout`：

![CrimeActivity 类的 fragment 托管布局](http://img.rosuh.me/wiki/wiki_201712_d7c848.png)

- `FragmentLayout`是服务于`CrimeFragment`的容器视图
  - 此容器视图是个通用性视图，不单用于`CrimeFragment`类，你还可以用它托管其他的`fragment`

我们会在`activity_crime.xml`文件中使用`FragmentLayout`作为默认布局：

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:id="@+id/fragment_container"
             android:layout_width="match_parent"
             android:layout_height="match_parent"/>
```

- 当前的`activity_crime.xml`布局文件仅由一个服务于单个`fragment`的容器视图组成
  - 除了自身组件之外，托管`activity`布局还可定义多个容器视图



#  7.5 创建`UI fragment`

创建`UI fragment`的步骤与创建`activity`的步骤相同：

- 定义用户界面布局文件
- 创建`fragment`类并设置其视图为定义的布局
- 编写代码以实例化组件



####  定义`CrimeFragment`的布局

- `CrimeFragment`视图用来显示包含在`Crime`类实例中的信息

balabala...



####  创建`CrimeFragment`类

1. 实现`fragment`生命周期方法

`CrimeFragment`类是与模型及视图对象交互的控制器，用于显示特定的`cirme`的明确信息。并在用户修改这些信息立即进行更新。

我们上一个例子中，`activity`通过其生命周期方法完成了大部分逻辑控制工作。在本个例子中，这些工作`fragment`的生命周期方法完成的。

```java
import android.support.v4.app.Fragment;
public class CrimeFragment extends Fragment {
    private Crime mCrime;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mCrime = new Crime();
    }
}

```

*解析*：

- `Fragment.onCreate(Bundle)`是公共方法，而`Activity.onCreate(Bundle)`是受保护方法
  - `Fragment.onCreate(Bundle)`方法及其他`Fragment`生命周期方法必须是公共方法，因为托管`fragment`的`activity`要调用它们
- ​





















