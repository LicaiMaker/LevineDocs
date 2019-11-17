>  需要集成[LevineUtils](/zh-cn/Android/LevineUtils/README)

### LogUtils的使用

```java
LogUtils.e("error");
LogUtils.d("debug");
LogUtils.i("info");
LogUtils.w("warn");
LogUtils.v("verbose");
```



![image-20191117182627385](C:\Users\summer\AppData\Roaming\Typora\typora-user-images\image-20191117182627385.png)





另外，可全区设置关闭或打开log

```java
LogUtils.setDebugable(true);
```

```java
LogUtils.setDebugable(false);
```

