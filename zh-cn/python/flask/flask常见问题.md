### 一.Python3出现UnicodeEncodeError: 'ascii' codec can't encode characters问题
这是由于Python环境字符集不是utf-8导致的，所以遇到中文时就不能编码了，所以现在需要将
字符集编程设为UTF-8
- 1.第一种：改变系统的环境变量
`echo 'export LANG=en_US.UTF-8' >> ~/.bashrc`
然后重新登录即可
- 2.第二种：改变Python环境变量
在python的Lib\site-packages文件夹下新建一个sitecustomize.py，内容为：
```
# encoding=utf8  
import sys  
from importlib import reload # 这句在Python2中不需要
reload(sys)  
sys.setdefaultencoding('utf8')   
```
然后重启Python解释器

> 查看当前的Python环境的字符集：`python -c "import sys; print(sys.stdout.encoding)"`,注意，以下代码返回UTF-8并不能说明没有问题:`python -c "import sys; print(sys.getdefaultencoding())"`

> 参考链接：1.https://blog.csdn.net/chenlide683424/article/details/100914599?utm_medium=distribute.pc_relevant.none-task-blog-baidujs-5   2.https://blog.csdn.net/icyfox_bupt/article/details/103300826?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase  3.https://blog.csdn.net/lezeqe/article/details/85677822