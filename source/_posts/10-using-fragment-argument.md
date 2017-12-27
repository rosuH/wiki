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









































