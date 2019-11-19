> **这里是关于Levine的第三方库LevineUtils的文档**

#### 1.LevineUtils简介

[![stars](https://badgen.net/github/stars/LicaiMaker/LevineUtils?icon=github&color=4ab8a1)](https://github.com/LicaiMaker/LevineUtils) [![forks](https://badgen.net/github/forks/LicaiMaker/LevineUtils?icon=github&color=4ab8a1)](https://github.com/LicaiMaker/LevineUtils)

LevineUtils是关于android中常用的工具模块集合，使用java语言.其中包含了用来全局显示和隐藏Fragment的FragmentFactory,和自定义的RecyclerListView,Recyclerview的万能适配器，RecyclerViewpager,Grdiview,ItemDecoration等许多常用的自定义view，还包含一些LogUtils等常用工具.

Github地址:[LevineUtils](https://github.com/LicaiMaker/LevineUtils)

####  2.在项目的根目录下的build.gradle文件添加maven地址

在根目录下的**build.gradle**文件中的allprojects中添加以下代码：

```groovy   
allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://jitpack.io' }
    }
}
```

新添加了一个maven仓库的地址，因为LevineUtils是放在jitpack中的.

####  3.在使用的module中的build.gradle文件中添加依赖

```groovy
 api "com.github.LicaiMaker:LevineUtils:${levine_utils_version}"
```

levine_utils_version:[![](https://jitpack.io/v/LicaiMaker/LevineUtils.svg)](https://jitpack.io/#LicaiMaker/LevineUtils)

> 使用api声明依赖的话具有依赖延续性，即可以在依赖这个module的module中使用LevineUtils；而使用implements声明依赖则只能在本module中使用.     

#### 4.常用模块

 [FragmentFactory](/zh-cn/Android/LevineUtils/FragmentFactory)

 [RecyclerView的万能适配器](/zh-cn/Android/LevineUtils/BaseRecyclerViewAdapter万能适配器)

 [recyclerview实现的ViewPager](/zh-cn/Android/LevineUtils/RecyclerViewPager)

 [RecyclerView实现的ListView](/zh-cn/Android/LevineUtils/RecyclerListView)

 [RecyclerView实现的GridRecyclerView](/zh-cn/Android/LevineUtils/GridRecyclerView)

 [自定义ItemDecorator](/zh-cn/Android/LevineUtils/ItemDecorator)

 [实现微信通讯录侧边栏indexbar](/zh-cn/Android/LevineUtils/IndexBar)

[用recyclerview自定义的可扩展列表的适配器](/zh-cn/Android/LevineUtils/BaseExpandableRecyclerViewAdapter)

 [通过注解实现butterknife的bindview和onclick功能](/zh-cn/Android/LevineUtils/LevineBindView)

[LogUtils](/zh-cn/Android/LevineUtils/LogUtils)



