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

![CriminalIntent 应用对象](https://img.rosuh.me/wiki/wiki_201712_a47d61.png)





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

记得我们需要为`fragment`创建一个`activity`视图来容纳它吗？

创建托管`CrimeListFragment`的`CrimeListActivity`类之前，首先为`CrimeListActivity`创建视图。



####  通用型`fragment`托管布局

*通用的布局定义文件`activity_fragment.xml`容器视图*：

```java
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:id="@+id/fragment_container"
             android:layout_width="match_parent"
             android:layout_height="match_parent"/>

```

- 此处没有特别指定`fragment`，任何使用`activity`托管`fragment`的场景，都可以使用它


####  抽象`activity`类

可以复用`CrimeActivity`的代码来创建`CrimeListActivity`类。

*近乎通用的`CrimeActivity`类(`CrimeActivity.java`)*：

```java
public class CrimeActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_fragment);

        FragmentManager fm = getSupportFragmentManager();
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

可以看到这样的代码结构比较简单，而每一次新建一个`activity`都需要创建这样一段代码。所以我们可以将重复的代码封装为抽象类。（后文的`CrimeActivit`和`CrimeListActivity`都会用到此类代码，所以有复用价值）

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

我们要做的工作就是让`SingleFragmentActivity`的子类实现该方法，来返回`activity`托管的`fragment`实例。

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

- `RecyclerView`所做的事情，就是创建视线中的子项以及回收再利用，循环往复
- `RecyclerView`的任务仅限于回收和定位屏幕上的`View`



####  `ViewHolder`

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

#### `Adapter`

- `RecyclerView`自己不创建`ViewHolder`
  - 这个任务交给了`Adapter`
- `Adapter`是一个控制器对象，从模型层获取数据，然后提供给`RecyclerView`显示，是沟通的桥梁
- `Adapter`负责
  - 创建必要的`ViewHolder`
  - 绑定`ViewHolder`至模型层数据
- 要创建`Adapter`，首先要定义`RecyclerView.Adapter`子类
  - 然后由它封装从`CrimeLab`获取的`crime`
- `RecyclerView`需要显示视图对象时，就会找他的`Adapter`


![生动有趣的 RecyclerView-Adapter 对话](https://img.rosuh.me/wiki/wiki_201712_fb0cd7.png)



**`RecyclerView-Adapter `对话**

- 首先，调用`Adapter`的`getItemCount()`方法，`RecyclerView`询问数组列表中包含多少个对象
- 接着，`RecyclerView`调用`Adapter`的`onCreateViewHolder(ViewGroup, int)`方法创建`ViewHolder`及其要显示的视图
- 最后，`RecyclerView`会传入`ViewHolder`及其位置，调用`onBindViewHolder(ViewHolder, int)`
  - `Adapter`会找到目标位置的数据并将其绑定到`ViewHolder`视图上
  - 绑定：使用模型数据填充视图
- `onCreateViewHolder(ViewGroup, int)`方法调用并不频繁
  - 一旦有了够用的`ViewHolder`，`RecyclerView`就会停止调用`ViewHolder(...)`方法
  - 它会回收`ViewHolder`以节约时间和内存



####  使用`RecyclerView`

1. 添加依赖

你可以在`build.gradle`文件中直接写入`RecyclerView`的依赖库。不过我觉得更好的方法是在`Project Structure`-->`app`-->`Dependencies`添加依赖。这样会直接搜索最新版的依赖项。

2. 修改`.xml`文件，将根视图改为`RecyclerView`，并配置 ID 属性

*在布局文件中添加`RecyclerView`视图（`fragment_crime_list.mxl`）*

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/crime_recycler_view"
    android:layout_width="match_parent" 
    android:layout_height="match_parent"/> 
```



3. 关联视图和`fragment`

修改`CrimeListFragment.java`类文件，使用布局并找到布局中的`RecyclerView`视图。

*为`CrimeListFRagment`配置视图（`CrimeListFragment.java`）*

```java
public class CrimeListFragment extends Fragment {
    private RecyclerView mCrimeRecyclerView;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_crime_list, container, false);
        
        mCrimeRecyclerView = (RecyclerView)view
                .findViewById(R.id.crime_recycler_view);
        mCrimeRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
        return view;
    }
}
```

- `RecyclerView`视图创建完成之后，立即转交给了`LayoutManager`对象
  - `LayoutManager`对象负责在屏幕上摆放列表项还负责定义屏幕滚动行为，因此没有了它，`RecyclerView`没法正常工作，应用可能会崩溃


目前实现了一个`RecyclerView`空视图。要显示出`crime`列表项，还需要完成`Adapter`和`ViewHolder`的实现。

####  列表项视图

我们要为`RecyclerView`上的列表项创建视图层级结构。

*`list_item_crime.xml`*：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical"
              android:padding="8dp">
    
    <TextView
        android:id="@+id/crime_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Crime Title"/>
    
    <TextView
        android:id="@+id/crime_date"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Crmie Date"/>
</LinearLayout> 
```



####  实现`ViewHolder`和`Adapter`

接着我们在`CrimeListFragment`类中定义`ViewHolder`内部类，它会实例化并使用`list_item_crime`布局。

*定义`ViewHolder`内部类（`CrimeListFragment.java`）*：

```java
public class CrimeListFragment extends Fragment {
    ...
    
    private class CrimeHolder extends RecyclerView.ViewHolder{
        public CrimeHolder(LayoutInflater inflater, ViewGroup parent) {
            super(inflater.inflate(R.layout.list_item_crime, parent,false));
        }
    }
}
```



- 在`CrimeHolder`的构造方法里，我们首先实例化`list_item_crime`布局，然后传给`super(...)`方法，也就是`ViewHolder`构造方法
- 基类`ViewHolder`因而实际上引用这个视图
  - 我们可以在`itemView`变量中找到它



接下来实现`Adapter`。

*创建`Adapter`内部类（`CrimeListFragment.java`）*

```java
public class CrimeListFragment extends Fragment {
  ...
    private class CrimeAdapter extends RecyclerView.Adapter<CrimeHolder> {
      private List<Crime> mCrimes;
      
      public CrimeAdapter(List<Crime> crimes) {
        mCrimes = crimes;
      }
    }
}
```

- 需要显示新创建的`ViewHolder`或让`Crime`对象和已创建的`ViewHolder`关联时，`RecyclerView`会去找`Adapter`（调用它的方法）
  - `RecyclerView`不关心也不了解具体的`Crime`对象，这是`Adapter`要做的事



我们还需要在`Crimedapter`中实现三个方法：

*武装`CrimeAdapter`（`CrimeListFragment.java`）*

```java
...
private class CrimeAdapter extends RecyclerView.Adapter<CrimeHolder>{
        ...

        @Override
        public CrimeHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            return null;
        }

        @Override
        public void onBindViewHolder(CrimeHolder holder, int position) {

        }

        @Override
        public int getItemCount() {
            return 0;
        }
    }
```



- `RecyclerView`需要新的`ViewHolder`来显示列表时，会调用`onCreateViewHolder`方法
  - 这个方法内部，我们创建一个`LayoutInflater`，然后用它创建`CrimeHolder`
- 关联`Adapter`和`RecyclerView`
  - 实现一个设置`CrimeListFragment`用户界面的`updateUI`方法
    - 该方法创建`CrimeAdapter`，然后设置给`RecyclerView`

*设置`Adapter`（`CrimeListFragment.java`）*

```java
public class CrimeListAdapter extends Fragment {
  ...
  private CrimeAdapter mAdapter;
  
  @Override
  public View onCreateView(LayourInflater inflater, ViewGroup container,
                           Bundler savedInstanceState) {
    View view = inflater.inflate(R.layout.fragment_crime_list, container, false);
    
    mCrimeRecyclerView = (RecyclerView) view
      					.findViewById(R.id.crime_recycler_view);
    mCrimeRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
    
    updateUI();		// **
    
    return view;
  }
  
  public void updateUI() {
    CrimeLab crimeLab = CrimeLab.get(getActivity());
    List<Crime> Crimes = crimeLab.getCrimes();
    
    mAdaper = new CrimeAdapter(crimes);
    mCrimeRecyclerView.setAdapter(mAdapter);
  }
}
```



现在已经实现了基本的列表内容了。可以编译运行看看啦。



# 8.4 绑定列表项

- 绑定：让 Java 代码（`Crime`里的模型数据，或点击监听器）和组件关联起来
  - 因为`CrimeHolder`会循环使用，分开处理视图创建和绑定会有好处
  - 我们把视图绑定工作放入`CrimeHolder`类里
    - 绑定之前，首先实例化相关组件；此项工作是一次性任务，因此直接放在构造方法里处理

*在构造方法中实例化视图组件（`CrimeListFragment.java`）*

```java
private class CrimeHolder extends RecyclerView.ViewHolder {
  private TextView mTitleTextView;
  private TextView mDateTextView;
  
  public CrimeHolder(LayoutInflater inflater, ViewGroup parent) {
            super(inflater.inflate(R.layout.list_item_crime, parent,false));

            mTitleTextView = (TextView)itemView.findViewById(R.id.crime_title);
            mDateTextView = (TextView)itemView.findViewById(R.id.crime_date);
        }
}
```



- `CrimeHolder`还需要实现一个`bind(Crime)`方法
  - 每次有新的`Crime` 要在 `CrimeHolder`中显示时，都要调用它一次

*实现`bind(Crime)`方法（`CrimeListFragment.java`）*

```java
public void bind(Crime crime) {
        mCrime = crime;
        mTitleTextView.setText(mCrime.getTitle());
        mDateTextView.setText(mCrime.getDate().toString());
    }
```

- 现在只要取得一个`Crime`，`CrimeHolder`就会刷新显示`TextView`标题视图和`TextView`日期视图

  最后修改`CrimeAdapter`类，使用`bind(Crime)`方法：每次`RecyclerView`要求`CrimeHolder`绑定对应的`Crime`时，都会调用`bind(Crime)`方法。

*调用`bind(Crime)`（`CrimeListFragment.java`）*

```java
private class CrimeAdapter extends RecyclerView.Adapter<CrimeHolder> {
  ...
  @Override
        public void onBindViewHolder(CrimeHolder holder, int position) {
            Crime crime = mCrimes.get(position);
            holder.bind(crime);
        }
  ...
}
```



#  8.5 响应点击

一般来说，`RecyclerView`只处理列表项相关工作，而触摸事件是需要我们自己实现的。

实现触摸事件常用方案就是设置`OnClickListener`监听器。既然列表项视图都关联着`ViewHolder`，就可以让`ViewHolder`为它监听用户触摸事件。

*检测用户点击事件（`CrimeListFragment.java`）*

```java
private class CrimeHolder extends RecyclerView.ViewHolder
            implements View.OnClickListener{
...
            itemView.setOnClickListener(this);
        }

        @Override
        public void onClick(View view) {
            Toast.makeText(getActivity(),
                    mCrime.getTitle() + " clicked!", Toast.LENGTH_SHORT)
                    .show();
        }
    }
```











