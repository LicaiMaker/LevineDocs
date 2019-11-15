**这里是关于Levine的第三方库LevineUtils的文档**

集成**LevineUtils**的步骤：

####  1.在项目的根目录下的build.gradle文件添加maven地址

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

####  2.在使用的module中的build.gradle文件中添加依赖

```groovy
 api "com.github.LicaiMaker:CommonUtils:${levine_utils_version}"
```

levine_utils_version:[![](https://jitpack.io/v/LicaiMaker/CommonUtils.svg)](https://jitpack.io/#LicaiMaker/CommonUtils)

> 使用api声明依赖的话具有依赖延续性，即可以在依赖这个module的module中使用LevineUtils；而使用implements声明依赖则只能在本module中使用.     

#### 3.常用模块

##### [FragmentFactory](/zh-cn/Android/LevineUtils/FragmentFactory)

##### [RecyclerView的万能适配器]()

##### [recyclerview实现的ViewPager]()

##### [RecyclerView实现的ListView]()

##### [RecyclerView实现的GridRecyclerView]()

##### [实现微信通讯录侧边栏indexbar]()

##### [自定义ItemDecorator]()

##### [通过注解实现butterknife的bindview和onclick功能]()



