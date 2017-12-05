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





































