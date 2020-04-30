>  1.在我已部署到服务器上的项目，再次部署的时候，设计数据库里数据的变动（数据，表或者表结构的变化），两者会有冲突，怎么解决？

>  2.再次部署时，能不能不上传数据库文件，每次在项目更新后，在已部署的项目中重新生成新增加的表或更改表结构呢？

### 1.git 添加了.gitignore没有生效怎么办？

原因是：

 **.gitignore 只能忽略那些原来没有被追踪的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未被追踪状态），然后再提交：**

\#cmd 进入.gitignore所在的目录下，删除缓存 git rm -r --cached . git add . git commit -m 'update .gitignore'

### 2.Git冲突：commit your changes or stash them before you can merge

#### 1.保留本地修改的方法：

```shell
git stash 保存备份本地修改到git栈里

git pull #拉取远端的内容 

git stash pop #从git栈中回复
```

两者之间可能产生冲突，需要解决冲突

此时，可使用**git mergetool**来解决冲突

#### 2.保留远端修改的做法



```shell
git reset --hard git pull
```