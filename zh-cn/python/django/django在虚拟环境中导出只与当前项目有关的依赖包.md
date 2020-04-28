> 问题引入：当需要部署时，要在服务器上安装新的python虚拟环境，想要导出django项目中用到的依赖库，常规的做法是:`pip freeze > requirements.txt`,但是，这个会把环境中所有的依赖包给列举出来，有很多不是这个环境需要的，所以此方法不可用

解决办法是：使用 `pipreqs`
> 这个工具的好处是可以通过对项目目录的扫描，自动发现使用了那些类库，自动生成依赖清单。缺点是可能会有些偏差，需要检查并自己调整下。
> `  

 1.首先安装 pipreqs# `pip install pipreqs`  
安装位置在pip所在的目录下,不要在虚拟环境中安装，在外面全局安装，这样所有环境都能用

 2.使用：
 - 生成requirements.txt文件，在命令行中输入`pipreqs 项目的目录`(不能在虚拟环境中运行)
 - 在新的虚拟环境中安装：`pip install -r 项目目录/requriements.txt`

 > 参考链接：[pipresqs](https://www.jb51.net/article/168289.htm)