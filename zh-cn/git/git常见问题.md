> 1.在我已部署到服务器上的项目，再次部署的时候，设计数据库里数据的变动（数据，表或者表结构的变化），两者会有冲突，怎么解决？

> 2.再次部署时，能不能不上传数据库文件，每次在项目更新后，在已部署的项目中重新生成新增加的表或更改表结构呢？


### 1.git 添加了.gitignore没有生效怎么办？
原因是：
 > .gitignore 只能忽略那些原来没有被追踪的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未被追踪状态），然后再提交：
 ```bash
#cmd 进入.gitignore所在的目录下，删除缓存
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
 ```

### 2.Git冲突：commit your changes or stash them before you can merge
- 1.保留本地修改的方法：
git stash #保存备份本地修改到git栈里
git pull #拉取远端的内容
git stash pop #从git栈中回复

> 两者之间可能产生冲突，需要解决冲突
> 此时，可使用git mergetool来解决冲突

- 2.保留远端修改的做法
git reset --hard
git pull

### 3.每次提交都提示everything up-to-date

我的情况是，我有两个分支：master和newbranch，我默认选的是newbranch，但是每次提交选择的是master分支：`git push origin master`,这个时候如果要提交到newbranch的话，就使用`git push `就行；如果提交到master的话，先切换到master分支，再提交。
```
git checkout master
git push origin master
```
> 注：查看默认的分支（git branch）

> 另外，可能涉及到分支合并的问题
```bash
$ git checkout master
然后我们要将新分支提交的改动合并到主分支上


$ git merge newbranch
合并分支可能产生冲突这是正常的，虽然我们这是新建的分支不会产生冲突，但还是在这里记录下。下面的代码可以查看产生冲突的文件，然后做对应的修改再提交一次就可以了。


$ git diff
我们的问题就解决了，接下来就可以push代码了。


$ git push -u origin master
新建分支的朋友别忘了删除这个分支


$ git branch -D newbranch
```

> 参考地址：https://blog.csdn.net/heguixian/article/details/50883916