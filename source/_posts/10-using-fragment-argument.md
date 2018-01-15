---
title: "10 使用 fragment argument"
date: {{ date }}
tags: 
  - "Android"
  - "fragment argument"

top: 10
categories: 
  - "Android 开发"
---

# 前言

我们之前的应用实现了从`activity`中启动`activity`的功能。现在我们要实现从`fragment`中启动`activity`。在本例中，就是从`CrimeListFragment`中启动`CrimeActivity`实例。

![示意图](https://img.rosuh.me/wiki/wiki_201712_602156.png)

在本例中，我们将关联`CriminalIntent`应用的列表与明细部分。用户点击某个`Cirme`列表项时，会创建一个托管`CrimeFragment`的`CrimeActivity`。



#  10.1 从`fragment`中启动`activity`

- 要从`fragment`中启动`activity`，类似于`activity`中启动`activity`
  - 我们调用`Fragment.startActivity(Intent)`方法
  - 由它在后台再调用对应的`Activity`方法



本例中，我们将在`CrimeListFragment`的`CrimeHolder`类里，用启动`CrimeActivity`实例的代码，替换`toast`消息处理代码。

*启动`CrimeActivity`（`CrimeListFragment.java`）*：

```java
private class CrimeHolder extends RecyclerView.ViewHolder
implements View.OnclickListener {
	...
	@Override
    public void onClick(View view) {
    	Intent intent = new Intent(getActivity(), CrimeActivity.class);
    	startActivity(intent);
    }
}
```

上述代码和`activity`启动`activity`相当类似。

- 指定要启动的`activity`为`CrimeActivity`
  - `CrimeListFragment`创建了一个显式`itent`
- `CrimeListFragment`通过`getActivity()`方法传入他托管的`activity`来获取构造`Intent`所需要的`Context`对象

####  附加`extra`信息

为了实现能分辨是哪一个`fragmen`启动的`activity`，我们需要在启动`CrimeActivity`时，传递附加到`Intent extra`上的`crime ID`。

- 需要使用`CrimeActivity`中新增`newIntent`

*创建`newIntent`方法（`CrimeActivity.java`）*：

```java
public class CrimeActivity extends SingleFragmentActivity {
  public static final String EXTRA_CRIME_ID =
            "com.bignerdranch.android.criminalintent.cirme_id";

    public static Intent newIntent(Context packageContext, UUID crimeId) {
        Intent intent = new Intent(packageContext, CrimeActivity.class);
        intent.putExtra(EXTRA_CRIME_ID, crimeId);
        return intent;
    }
}
```

- 创建了显式`intent`后，调用`putExtra(...)`方法，传入匹配的`crimeId`字符串键与键值
  - 此处`UUID`是`Serializable`对象，所以调用`putExtra(String, Serializable)`方法

注意，上面我们是在`CtimreActivity`中编写的代码，还记得我们之前学习`Intent`的时候说过的知识点吗？

> 如何在使用`intent`并传递数据的时候，不考虑传递数据的细节和有效提高复用性？

啊哈！就是这里啦。我们在被调用的`activity`内创建静态方法`newIntent`，让这个被调用者自己把控要传入的数据类型和个数，让调用者们直接调用而不需考虑太多；还可以提高复用性呢！

接下来看看我们如何使用`newIntent`方法吧。

*传递`Crime`实例(`CrimeListFragment.java`）*：

```java
private class CrimeHolder extends RecyclerView.ViewHolder
implements View.OnclickListener {
	...
	@Override
    public void onClick(View view) {
    	Intent intent = CrimeActivity.newIntent(getActivity(), mCrime.getId());
    	startActivity(intent);
    }
}
```

wow! 简直绝赞！



####  获取`extra`信息

我们已经把`crimeId`信息安全地存储到`CrimeActivity`的`intent`中了。不过，要使用和获取`extra`信息的是`CrimeFragment`类。

`fragment`有两种方法获取`intent`中的数据：

- 简单直接的方法
  - `CrimeFragment`直接使用`getActivity()`方法获取`CrimeActivity`的`intent`
  - 回到`CrimeFragment.java`文件，取到`CrimeActivity`的`intent`内的`extra`信息
  - 再用它获取`Crime`对象

```java
public class CrimeFragment extends Fragment {
...
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
  		// mCrime = new Crime();
        UUID crimeId = (UUID)getActivity().getIntent()
                .getSerializableExtra(CrimeActivity.EXTRA_CRIME_ID);
        mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);
    }

```

我们需要理解的是，`CrimeFragment`是由`CrimeActivity`创建出来的。

所以其使用`getActivity()`就可以获取到`CrimeActivity`实例。接着通过该实例获取`intent`，由`intent`里的`crimeId`来从`CrimeLab`单例中获取特定`Crime`对象。



####  使用`Crime`数据更新`CrimeFragment`视图

获取了`Crime`对象，`CrimeFragment`视图便可以显示该`Crime`对象的数据了。

我们需要更新`onCreateView(...)`方法，显示`Crime`对象的标题以及解决状态。

*更新视图对象（`CrimeFragment.java`）*：

```java
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
                         Bundle savedInstanceState) {
  ...
  mTitleField.setText(mCrime.getTitle());
  ...
  mSolvedCheckBox.setChecked(mCrime.isSolver);
  ...
}
```



####  直接获取 extra 信息的缺点

在上述方式中，我们简单使用几行代码，便可以让`fragment`直接从托管`activity`的`intent`中获取信息。

然而，这样的方式破坏了`fragment`的封装。

- `CrimeFragment`不再是可复用的构建单元
  - 因为它现在由某个特定的`activity`托管着
  - 该特定的`activity`的`Intent`又定义了名为`com.bignerdranch.android.criminalintent.crime_id`的`extra`
- `CrimeFragment`再也无法被其他`activity`使用

一个更好的做法，是将`crimeID`存储在属于`CrimeFragment`的某个地方，而不是保存在`CrimeActivity`的私有空间里。这样，无需依赖`CrimeActivity`的`intent`内的`extra`，`CrimeFragment`就能获取自己所需的`extra`数据。

一般，那个某个地方就是实际上就是`fragment`的`argument bundle`。



#  10.2 `fragment argument`

- 每一个`fragment`实例都可以附带一个`Bundle`对象
  - 该`Bundle`包含键值对，方便我们像`Activity`的`Intent`中那样使用它们
  - 一个键值对就是一个`argument`

**创建`fragment argument`的方法：**

- 创建`Bundle`对象
- 使用`Bundle`限定类型的的`put`方法，将`argument`添加到`bundle`中

```java
Bundle args = new Bundle();
args.putSerializable(ARGS_MY_OBJECT, myObject);
args.putInt(ARGS_MY_INT, myInt);
args.outCharSequence(ARG_MY_STRING, myString);
```



####  附加`argument`到`fragment`

- 调用`Fragment.setArguement(Bundle)`方法来附加`argument bundle`给`fragment`
  - 附加时机需要在 `fragment`创建后，添加给`acticity`之前
  - 比如本例子中，我们使用的是`activity` 继承`SingleFragmentActivity.java`来创建具体`fragment`的方法；那么你查看一下`SingleFragmentActivity`就知道，内部先调用`activity`实现的`createFragment()`，然后才在`FragmentManager`事务中使用`add`操作添加给`activity`
- 合适的做法：`newInstance()`
  - 添加一个名为`newInstance()`的静态方法给`Fragment`类
  - 使用该方法完成`fragment`实例以及`Bundle`对象的创建
  - 然后将`argument`放入`bundle`中
  - 最后附加给`fragment`

我们之前需要使用`fragment`实例时，使用 `activity`调用`Fragment`构造方法。现在我们转而调用`newInstance()`方法，既可以创建`fragment`实例，`activity`又可以给`newInstance()`方法传入任何需要的参数。

*`CrimeFragment.java`*

```java
public static CrimeFragment newInstance(UUID crimeId) {
        Bundle args = new Bundle();
        args.putSerializable(ARG_CRIME_ID, crimeId);

        CrimeFragment fragment = new CrimeFragment();
        fragment.setArguments(args);
        return fragment;
    }
```

这里：

- 创建了一个属于`CrimeFrgament.java`的静态方法`newInstance()`
- 在里面完成了附加`argument`到`fragment`的一系列工作

使用：

- `CrimeActivity`调用`CrimeFragment.newInstance(UUID)`并传入从它的`extra`获取的`UUID`参数值

回到`CrimeActivity`类中，在`createFragment()`方法里，从`CrimeActivity`的`intent`中获取`extra`数据：

*`CrimeActivity.java`*

```java
@Override
    protected Fragment createFragment() {
        UUID crimeId = (UUID) getIntent()
                .getSerializableExtra(EXTRA_CRIME_ID);
        return CrimeFragment.newInstance(crimeId);
    }
```

这里：

- 托管`activity`知道`CrimeFragment`内部细节，这是必须的
- `fragment`不一定需要知道`activity`内部细节，特别是保持`fragment`通用独立的时候



####  获取`argument`

- `fragment`要获取`argument`，会先调用`Fragment`类的`getArgument()`方法
- 再调用`Bundle`限定类型的`get`方法

*`CrimeFragment.java`*

```java
@Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        UUID crimeId = (UUID) getArguments().getSerializable(ARG_CRIME_ID);
        mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);
    }
```



#  10.3 刷新显示列表项

我们进入了具体列表项，然后修改数据（或者本身进入的这个行为已经是一种数据），退出去之后需要更新列表项以反映出变化。

常见的例子，比如点击过的列表项，先改变其颜色来表示已访问；修改列表项中的某些状态，比如本例子，那就要在列表中反映出来。

- 模型层保存的数据若有变化，应该通知`RecyclerView`的`Adapter`，以便其及时获取最新数据并刷新显示列表项
  - 在恰当的时机，与系统的`ActicityManager`回退栈协同运作 ，可实现列表项的刷新功能

本例子中的回退栈流程为：

- `CrimeListFragment`启动`CrimeActivity`实例后
  - `CrimeActivity`被置于回退栈顶
  - `CrimeListActiciy`实例被暂停并停止
- 用户点击后退键回到列表项界面
  - `CrimeActivity`随机弹出栈并被销毁
  - `CrimeListActivity`立即重新启动并恢复运行
- `CrimeListActivity`恢复运行之后，操作系统发出调用`onResume()`生命周期方法的指令
  - `CrimeListActivity`接到指令之后 ，其`FragmentManager`调用当前被`activity`托管的`fragment`的`onResume()`方法
    - 本例中为`CrimeListFragment`
- 在`CrimeListFragment`中，覆盖`onResume()`方法，触发调用`updateUI()`方法刷新显示列表项
  - 如果已配置好`CrimeAdapter`，就调用`notifyDataSetChange()`方法来修改`updateUI()`方法



*`CrimeListFragemnt.java`*

```java
...
@Override
public void onResume() {
  super.onResume();
  updateUI();
}

private void updateUI() {
        CrimeLab crimeLab = CrimeLab.get(getActivity());
        List<Crime> crimes = crimeLab.getCrimes();

        if (mAdapter == null) {
            mAdapter = new CrimeAdapter(crimes);
            mCrimeRecyclerView.setAdapter(mAdapter);
        } else {
            mAdapter.notifyDataSetChanged();
        }
    }
```



#  10.4 通过`fragment`获取返回结果

记得我们之前使用`intent`的时候吗？传递完数据之后，我们可以在`Activity.startActivityForResult(...)`方法中获得返回结果。

对比着来看的话，`fragment argument`也是同样的道理哦。

- 调用`fragment.startActivityResult(...)`
- 覆盖`fragment.startActivityResult(...)`方法

*`CrimeListFragment.java`*

```java
@Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == REQUEST_CRIME){
            // Handle result
        }
    }
```



- 结果处理
  - `fragment`能够从`activity`中接受返回结果，但是**其本身无法持有返回结果**
    - 只有`activity`拥有返回结果
  - `fragment`没有`setResult()`方法

我们应该让托管`activity`返回结果值：

```java
public class CrimeFragment extends Fragment {
  ...
  public void returnResult() {
    getActivity().setResult(Activity.RESULT_OK, null);
}
```

注意：结果处理指的是接受数据的`activity`和`fragment`来完成的步骤，本例中指的是被启动的`CrimeFragment.java`哦。



#  10.5 深入学习：为何使用`fragment argument`

#### 直接在`CrimeFragment`里创建一个实例变量有什么坏处吗？

在本例中：在`CrimeFragment.java`中创建一个实例变量`CrimeID`，然后`CrimeListFragment`通过`setCrimeID()`方法来传递数据。这样做的缺点：

- 操作系统再重建`fragment`时，用户暂时离开当前 应用（系统按需回收内存），任何实例变量都将不复存在
  - 尤其是内存不够的时候，操作系统强制杀掉应用

#### 那么实例状态保存机制呢？

本例中：在`CrimeFragment.java`， 将`crimeID`赋值给实例变量，然后在`onSaveInstanceState(Bundle)`方法中保存下来；要使用时再从`onCreate(Bundle)`方法中的`Bundle`中取回

- 维护成本高
  - 我们总是需要把其他`fragment`的`argument`添加到`onSaveInstanceState(Bundle)`里面，这样导致代码结构不清晰，不利于理解代码
  - 与之相比，在专门的方法(`newInstance()`)一揽子解决这些问题，以后看起来就会清晰很多了。





























