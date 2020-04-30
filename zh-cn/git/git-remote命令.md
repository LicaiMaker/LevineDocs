## 克隆仓库

```git clone [ssh://software@172.16.0.30/~/yafeng/.git](ssh://software@172.16.0.30/~/yafeng/.git)```

## 在新建仓库的情况下，上传本地新建项目，进入到项目根目录下：

```shell
git init

git add README.md

git commit -m "first commit" #记住一定是双引号

git remote add origin git@github.com:LicaiMaker/mydocs.git

git push -u origin master
```
## 本地已存在仓库的情况下，将现有仓库push到远端：
```shell
git remote add origin git@github.com:LicaiMaker/mydocs.git git push -u origin master
```