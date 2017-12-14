---
title: "8 使用 RecyclerView 显示列表"
date: {{ date }}
tags: 
  - "Android"
  - "RecyclerView"

top: 8

categories: 
  - "Android 开发"
---

#  前言

为了实现我们的本次的项目，也就是`CriminalIntent`应用中显示`crime`列表的功能，我们需要用`RecyclerView`来实现列表显示。

- 应用模型层将新增一个`CrimeLab`对象，该对象是一个数据集中存储池，用来存储`Crime`对象
- 显示`crime`列表需要在应用控制器层新增一个`activity`和一个`fragment`
  - `CrimeListActivity`和`CrimeListFragment`

![CriminalIntent 应用对象](http://img.rosuh.me/wiki/wiki_201712_a47d61.png)





# 8.1 升级`CriminalIntent`应用的模型

我们需要将应用的模型层，从容纳单个`Crime`对象变为可容纳一组`Crime`对象。



#### 单例与数据集中存储

- `Crime`数组对象将存储在一个单例里
  - 单例是特殊的`Java`类，在创建实例时，一个单例类仅允许创建一个实例
  - 应用能在内存中存活多久，单例就存活多久
- 单例
  - 要创建单例，需要创建一个带有私有构造方法及`get()`方法的类
    - 如果实例已经存在，`get()`方法就直接返回它
    - 如果实例不存在，`get()`方法就会调用构造方法创建它

*`CrimeLab.java`* 

```java
public class CrimeLab {
    private static CrimeLab sCrimeLab;
    
    public static CrimeLab get(Context context) {
        if (sCrimeLab == null) {
            sCrimeLab = new CrimeLab(context);
        }
        return sCrimeLab;
    }
    
    private CrimeLab(Context context){
    }
}
```

- `sCrimeLab`变量带有`s`前缀，这是 Android 开发的命名约定，一看到此前缀，我们就直到`sCrimeLab`是一个静态变量
- `CrimeLab`的构造方法是私有的，其他类无法创建`CrimeLab`对象，只能通过`get()`方法
- `get()`方法中，我们传入`Context`对象




接着，我们往`CrimeLab`中存储`Crime`对象：

- 在`CrimeLab`的构造方法里，创建一个空`List`用来保存`Crime`对象
- `getCrime()`用来返回数组列表
- `getCrime(UUID)`返回带指定`ID`的`Crime`对象



*创建可容纳`Crime`对象的`List`（`CrimeLab.java`）*

```java
public class CrimeLab {
  ...
  private List<Crime> mCrimes;
  ...
  private CrimeLab(Context context) {
  	mCrimes = new ArrayList<>();  
  }
  public List<Crime> getCrimes() {
    return	mCrimes;
  }
  
  public Crime getCrime(UUID id) {
    for (Crime crime: mCrimes)   {
      if (crime.getID().equals(id)) {
        return crime;
      }
    }
    return null;
  }
}
```

*解析*：

- `List<E>`是一个泛型类，支持存放特定数据类型的有序列对象，拥有获取、新增和删除列表元素的方法
  - `mCrimes`含有`ArrayList`：鉴于此，推荐在声明变量的时候使用`List`接口类型；这样在有需要的时候还可以使用其他`List`
- `mCrimes`实例化语句使用的`<>`是在 Java 7 中引入的
  - 该符号告诉编译器，`List`中的元素类型可以基于变量声明传入的抽象参数来确定
  - 变量声明语句`private List<Crime> mCrimes`中指定了`Crime`参数，所以编译器可据此推测出`ArrayList`里可放入`Crime`对象

下面先批量存入 100 个`Crime`对象。

```java
private CrimeLab(Context context){
        mCrimes = new ArrayList<>();
        for (int i = 0; i < 100; i++){
            Crime crime = new Crime();
            crime.setTitle("Crime #" + i);
            crime.setSolved(i % 2 == 0);    //Every other one
            mCrimes.add(crime);
        }
    }
```





#  8.2 使用抽象`activity`托管`fragment`

创建托管`CrimeListFragment`的`CrimeListActivity`类之前，首先为`CrimeListActivity`创建视图。



####  通用型`fragment`托管布局

记得我们需要为`fragment`创建一个`activity`视图来容纳它吗？

*通用的布局定义文件`activity_fragment.xml`容器视图*：

```java
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:id="@+id/fragment_container"
             android:layout_width="match_parent"
             android:layout_height="match_parent"/>

```

- 此处没有特别指定`fragment`，任何托管`activity`托管`fragment`的场景，都可以使用它



####  抽象`activity`类

可以复用`CrimeActivity`的代码来创建`CrimeListActivity`类。

*近乎通用的`CrimeActivity`类(`CrimeActivity.java`)*：

```java
public class CrimeActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_fragment);

        android.support.v4.app.FragmentManager fm = getSupportFragmentManager();
        Fragment fragment = fm.findFragmentById(R.id.fragment_container);

        if (fragment == null) {
            fragment = new CrimeFragment();
            fm.beginTransaction()
                    .add(R.id.fragment_container, fragment)
                    .commit();
        }

    }
}
```

可以看到这样的代码结构比较简单，而每一次新建一个`activity`都需要创建这样一段代码。所以我们可以将重复的代码封装为抽象类。

创建一个`SingleFragmentActivity`的抽象类。设置超类为`AppCompatActivity`类。

*创建一个 `Activity`抽象类（`SingleFragmentActivity.java`）*：

```java
public abstract class SingleFragmentActivity extends AppCompatActivity {
    protected abstract Fragment createFragment();
    
    @Override
    protected void onCreate(Bundle saveInstanceState) {
        super.onCreate(saveInstanceState);
        setContentView(R.layout.activity_fragment);

        FragmentManager fm = getSupportFragmentManager();
        Fragment fragment = fm.findFragmentById(R.id.fragment_container);

        if (fragment == null) {	//*
            fragment = createFragment();
            fm.beginTransaction()
                    .add(R.id.fragment_container, fragment)
                    .commit();
        }
    }
}

```

可以看到上述方法除了在`*`号处将`new Fragment()`改为`createFragment()`抽象类方法外，其余和`CrimeActivity.java`中没什么区别。

我们要做的工作就是让`SingleFragmentActivity`的子类会实现该方法，来返回`activity`托管的`fragment`实例。

1. 使用抽象类


改动一下`CrimeActivity`类，将它的超类改为`SingleFragmentActivity`。然后删除`onCreate(Bundle)`方法，再添加`createFragment`。

*清理 `CrimeActivity 类(CrimeActivity.java)`*：

```java
public class CrimeActivity extends SingleFragmentActivity {
  @Override
  protected Fragment createFragment() {
    return new CrimeFragment();
  }
}
```

2. 新建控制类

类似的，使用`SingleFragmentActivity`类来创建控制类。

*实现`CrimeListActivity`(`CrimeListActivity.java`)*

```java
public class CrimeListActivity extends SingleFragmentActivity() {
  @Override
  protected Fragment createFragment() {
    return new CrimeListFragment();
  }
}
```

当然我们还要实现`CrimeListActivity.java`，这样才能使用构造器创建。不过现在我们对这个控制类先留空：

*CrimeListFragment.java*：

```java
public class CrimeListFragment extends Fragment{
  // Nothing yet
}
```



3. 在配置文件中声明`CrimeListActivity`

- 创建完`CrimeListActivity`，记得要在配置文件中声明它
- 因为本程式启动时的主界面应该是`crime`列表，因此还要在`AndroidManifest.xml`中声明为`launch activity`

```xml
<activity android:name=".CrimeListActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
</activity>
```



# 8.3 `RecyclerView`, `ViewHolder`和`Adapter`

- `RecyclerView`是`ViewGroup`的子类
  - 每一个列表只显示`Crime`的标题和日期
  - `View`是一个包含两个`TextView`的`LinearLayout`



![带有子 View 的 RecyclerView](https://img.rosuh.me/wiki/wiki_201712_3ab907.png)



仔细看看图片里的`RecyclerView`和`View`的关系。

当前屏幕只显示了 12 个子`View`。不在视线中的子`View`，就不会被创建出来。

你滑动屏幕就会显示更多子项目，然后之前出现过的，而不在视线中的子`View`就会被回收。

`RecyclerView`所做的事情，就是创建视线中的子项以及回收再利用，循环往复。



####  `ViewHolder`和`Adapter`

- `RecyclerView`的任务仅限于回收和定位屏幕上的`View`
- `ViewHolder`只做一件事：容纳`View`视图

![被吐嘈的 ViewHolder](https://img.rosuh.me/wiki/wiki_201712_ef918e.png)

下面是*典型的`ViewHolder`子类*：

```java
public class ListRow extends RecyclerView.ViewHolder {
  public ImageView mThumbnail;
  
  public ListRow(View view) {
    super(view);
    
    mThumbnail = (ImageView) view.findViewById(R.id.thumbnail);
  }
}
```

我们可以创建`ListRow`来获取自定义的`mThumbnail`和`RecyclerView.ViewHolder`超类传入的`itemView`。

- `ViewHolder`为`itemView`而生：它引用着传给`super(view)`的整个视图

*`ViewHolder`的使用示例*：

```java
ListRow row = new ListRow(inflater.inflate(R.layout.list_row, parent, false));
View view = row.itemView;
ImageView thumbnailView = row.mThumbnail;
```



- `RecyclerView`本身不会创建视图，它创建的是`ViewHolder`，而`ViewHolder`引用着`itemView`

![ViewHolder 配合 RecyclerView 使用](https://img.rosuh.me/wiki/wiki_201712_7eabb8.png)











































