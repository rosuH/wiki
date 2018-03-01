---
title: "7 UI Fragmnet 与 fragment 管理器"
date: {{ date }}

tags: 
  - "Android"
  - "frgament"

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

![明细 fragment 的切换](https://img.ioioi.top/wiki/wiki_2017_1.png)

- fragment 可以带来灵活多变的布局，代价就是复杂的应用、更多的组件以及大量实现的代码



# 7.3 着手开发示例应用

*CriminalIntent 就是本示例应用的名称咯。*

我们先来梳理一下开发流程。

##### 我们先来看一下整个`CriminalIntent`项目的对象图解，以便我们更好地理解开发流程

![项目对象图解](https://img.ioioi.top/wiki/wiki_2017_d.png)

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

![CrimeActivity 托管 CrimeFragment](https://img.ioioi.top/wiki/wiki_2017_4.png)

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

![fragment 的生命周期图解](https://img.ioioi.top/wiki/wiki_2017_724ee3.png)



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

![CrimeActivity 类的 fragment 托管布局](https://img.ioioi.top/wiki/wiki_201712_d7c848.png)

- `FragmentLayout`是服务于`CrimeFragment`的容器视图
  - 此容器视图是个通用性视图，不单用于`CrimeFragment`类，你还可以用它托管其他的`fragment`

我们会在`activity_crime.xml`文件中使用`FragmentLayout`作为默认布局：

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="https://schemas.android.com/apk/res/android"
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

*`CrimeFragment.java`*

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
- `Fragment`同样具有保存及获取状态的`bundle`
  - 类似于使用`Activity.onSaveInstanceState(Bundle)`，我们需要覆盖`Fragment.onSaveInstanceState(Bundle)`来使用
- `fragment`的视图不是在`Fragment.onCreate(Bundle)`中生成的，虽然我们在该方法中配置了`fragment`实例，但是创建和配置`fragment`视图是在另一个`fragment`生命周期方法完成的


```java
public View onCrateView(LayoutInflater inflater, ViewGroup container, 
                       Bundle savedInstanceState) {}
```



- 该方法实例化`fragment`视图的布局，然后将实例化的`View`返回给托管的`activity`
  - `LayoutInflater, ViewGroup`是必要参数，`Bundle`用来存储恢复数据，可供该方法从保存状态下重建视图

下面我们在`CrimeFragment.java`中，添加`onCreateView`方法的实现代码，从`fragment_crime.xml`布局中实例化返回布局。

```java
@Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                         Bundle saveInstanceState) {
        View v = inflater.inflate(R.layout.fragment_crime, container, false);
        return v;
    }
```

*解析*：

- 在`onCreateView(...)`方法中，`fragment`的视图是直接通过调用`LayoutInflater.inflate(...)`方法并传入布局的资源 ID 生成的
  - 第二个参数是视图的父视图，我们通常需要父视图来正确配置组件
  - 第三个参数告诉布局生成器是否将生成的视图添加个给父视图
    - 传入`flase`表示我们将以代码的方式添加视图



2. 在`fragment`中实例化组件

`fragment`中的`EditText, CheckBox, Button`组件，也都是在`onCreateView(...)`方法里实例化的。

```java
@Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                         Bundle saveInstanceState) {
        View v = inflater.inflate(R.layout.fragment_crime, container, false);

        mTitleFiled = (EditText)v.findViewById(R.id.crime_title);
        mTitleFiled.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {
                // This space intentionally left blank
            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                mCrime.setTitle(s.toString());
            }

            @Override
            public void afterTextChanged(Editable s) {
                // This on too
            }
        });

        return v;
    }
```



- 对比`Activity`中实例化组件，`Fragment`中需要手动调用`View.findViewById()`方法
- `onTextChanged(..)`方法中，调用`CharSequence`(表示用户输入)的`toString()`方法
  - 该方法最后返回用来设置的`Crime`标题字符串

接下来设置`Button`组件，让他显示`crime`的发生日期。

```java
...
  mDateButton = (Button)v.findViewById(R.id.crime_date);
        mDateButton.setText(mCrime.getDate().toString());
        mDateButton.setEnabled(false);
...
```

此处只是显示日期，而点击功能没有启用。

接着设置`ChcekBox`组件。引用它并设置监听器，根据用户操作，更新`mSolved`状态。

```java
...
  mSolvedCheckBox = (CheckBox)v.findViewById(R.id.crime_solved);
        mSolvedCheckBox.setOnCheckedChangeListener(new OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
                mCrime.setSolved(isChecked);
            }
        });
...
```





# 7.6 向`FragmentManager`添加`UI fragment`

`fragment`自己无法在屏幕上显示视图，我们需要把`CrimeFragment`添加给`CrimeActivity`。

- `FragmentManager`类负责管理`fragment`并将它们的视图添加到`activity`的视图层级结构中
  - `Activity`类中添加了`FragmentManager`

![FragmentManager 图解](https://img.ioioi.top/wiki/wiki_201712_720ea7.png)

- `FragmentManager`具体管理
  - `fragment`队列
  - `fragment`事务回退栈




在本例中，我们只需关心`FragmentManager`管理的`fragment`队列。

- 以代码的方式将`fragment`添加给`activity`，需要直接调用`activity`的`fragmentManager`
  - 先获取`fragmentManager`本身
  - 在`CrimeActivity.java`中，在`onCreate(Bundle)`方法中添加代码取得`fragmentManager`

*获取`fragmentManager`（`CrimeActivity.java`）*

```java
public class CrimeActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crime);

        android.support.v4.app.FragmentManager fm = getSupportFragmentManager();
    }
}

```



####  `fragment`事务

获取`fragmentManager`后，再获取一个`fragment`交给它管理。

*添加一个`CrimeFragment`（`CrimeActivity.java`）*

```java
Fragment fragment = fm.findFragmentById(R.id.fragment_container);
if (fragment == null) {
            fragment = new CrimeFragment();
            fm.beginTransaction()
                    .add(R.id.fragment_container, fragment)
                    .commit();
}
```

- `new` --> `add` --> `commit`
  - 事务的创建到提交的过程
- `fragment`事务用来被添加、移除、附加、分离或替换`fragment`队列中的`fragment`
  - 这是`fragment`动态组装和重新组装用户界面的关键
- `Fragment.beginTransaction()`
  - 创建并返回`fragmentTransaction`实例
    - 该实例类支持流接口（fluent interface）的链式方法调用，以此配置`FragmentTransaction`再返回它
- `add`是整个事务的核心
  - 参数
    - 容器视图资源 ID
      - 告诉`FragmentManager`，`fragment`视图应该出现在`activity`视图的什么位置
      - 作为`FragmentManager`队列中`fragment`的唯一标志
    - 新创建的`CrimeFragment`
- 从`FragmentManager`中获取`CrimeFragment`，使用容器视图资源 ID 就行了
  - 如果要向`activity`添加多个`fragment`，通常就需要分别为每个`fragment`创建具有不同 ID 的不同容器



总结起来，其是就是：

![总结](https://img.ioioi.top/wiki/wiki_201712_04aff3.png)



#### `FragmentManafer`和`fragment`生命周期

![再探 fragment 生命周期](https://img.ioioi.top/wiki/wiki_201712_5265c7.png)

- `activity`的`FragmentManager`负责调用队列中的`fragment`的生命周期方法
  - 添加`fragment`供`FragmentManager`管理时，`onAttach(Context), onCreate(Bundle)`和`onCreateView(...)`方法会被调用
  - 托管`activity`的`onCreate(Bundle)`方法执行后，`onActivityCreated(Bundle)`方法也不会被调用
    - 因为`CrimeActivity.onCreate(Bundle)`方法中添加`CrimeFragment`，所以`fragment`被添加后，该方法会被调用
- 当`activity`处于运行状态时，添加`fragment`后
  - `FragmentManager`会立即驱赶（指让`fragment`走得快一点...）`fragment`，调用一系列必要的生命周期方法，快速赶上`activity`的步伐
  - 一旦赶上，托管的`activity`的`FragmentManager`就会边接收操作系统的指令，边调用其他生命周期方法，让`fragment`与`activity`的状态保持一致

#  7.7. 采用`fragment`应用架构

尽管`fragment`组件可以复用，但是正确使用`fragment`非常重要 ，否则就边成了滥用。

- `fragment`时用来封装关键组件以便复用
  - 关键组件：针对应用的整个屏幕来讲的
  - 单屏使用大量的`fragment`，不仅使代码充斥`fragment`事务处理，模块的职责分工也会不清晰
  - 如果由很多零碎的晓组件需要复用，比较好的架构设计时使用定制视图
- 实践证明：**应用单屏最多使用 2 ~ 3 个 `fragment`** 

![少就是多的哲学](https://img.ioioi.top/wiki/wiki_201712_e4ee9d.png)



####  使用`fragment`的理由

- 实际开发中，尽管有时候可用可不用，但是我们还是会采用`fragment`
  - 因为后期添加`fragment`是一个大坑

作者坚信的 AUF（Always Use Fragments），总是使用`fragment`。



# 7.8 深入学习：`fragment`与支持库

- `AppCompat`库没有实现`fragment`功能，它依赖于`support-4`库，是个后者实现了`fragment`功能
- `support-v4`实现了`fragment`功能
  - 其库内也有一个`Activity`子类：`FragmentActivity`
  - 而`AppCompatActivity`时`FragmentActivity`的子类，所以应用能使用支持库版本的`fragment`



![AppCompatActivity 继承树](https://img.ioioi.top/wiki/wiki_201712_b243e8.png)





# 7.9 深入学习：为什么优先使用支持库版本的`fragment`

####  支持库版本的`fragment`使用起来最方便

- Google 每年会多次更新支持库，并借此引入新特性、修复 bug
  - 支持库的本意是方便在不支持该 API 的旧版本上使用
- 支持库版本的`fragment`没有显著的缺点
  - 功能实现上和系统内置的没有不同
  - 唯一缺点是导入支持库包会占用额外空间

#### 如何使用内置版？

如果要使用内置版本的`fragment`，需要对项目作如下改动：

- 弃用`FragmentActivity`类，改用标准库中的`Activity`类（`android.app.Activity`）
- 弃用`android.support.v4.app.Fragment`类，改用`android.app.Fragment`类
- 弃用`getSupportFragmentManager()`方法，改用`getFragmentManager()`方法获取`FragmentManager`

































