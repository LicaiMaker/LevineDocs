[toc]
### 1./usr/lib/x86_64-linux-gnu/libpython3.5m.so.1.0: undefined symbol: XML_SetHashSalt
> 参考链接：https://blog.csdn.net/J_H_C/article/details/84961219?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase 

每次出现错误时，执行`source ~/.bashrc`

### 2.linux彻底卸载Python3

- 卸载pyhton3：  

 ```rpm -qa|grep python3|xargs rpm -ev --allmatches --nodeps```

- 删除所有残余文件  

 `whereis python3 |xargs rm -frv`

- 卸载完成

查看现有的已安装的python：  

` whereis python`

### 3.Python开启HTTP服务
```
Python <= 2.3
python -c "import SimpleHTTPServer as s; s.test();" 8000

Python >= 2.4
python -m SimpleHTTPServer 8000

Python 3.x
python -m http.server 8000
```
### 4.Python中的高阶函数map和reduce的
map和reduce传入的两个参数都是函数f和一个列表：
map(f,list)和reduce(f,list)，但是map函数是，将列表中的每个元素都进行f运算，返回值是一个map;reduce是需要的f满足的条件是两个参数，reduce高数是将list中的前两个数进行f运算，再将运算结果和后面一个数再进行f运算，最后得到最终结果.
- map 
```python
print([item for item in map(lambda x:x*x,[1,2,3,4])])#[1,4,9,16]
```
- reduce
```
from functools import reduce
print(reduce(lambda x,y:x+y,[1,2,3,4]))#10
#例子:计算1-100这100个数之和
print(reduce(lambda x,y:x+y,[n for n in range(100)]))#4950
```

### 5.python中的小整数对象池和intern机制
- 小整数对象池
Python中为了提高效率，将[-5,256]之间的整数置于对象池中，以至于下面的例子：
```
a=300
b=300
a==b # true
a is b # false

a=200
b=200
a==b # true
a is b # true
```
- 字符串中的intern(驻留)机制


> intern机制是，在创建新的字符串对象时，会先在缓存池里面找是否有已经存在的值相同的对象（标识符，即只包含数字、字母、下划线的字符），如果有，则直接拿过来用（引用），避免频繁的创建和销毁内存，提升效率。

```
a='abc',
b='a'+'bc'
c='ab'+'c'
d='a'+'b'+'c'

id(a)=id(b)=id(c)=id(d)# a b c d的引用的是同一个对象
```

> 值同样的字符串对象仅仅会保存一份，是共用的，这也决定了字符串必须是不可变对象. 

但是，请注意下面的例子！！！
```
#1.
'o'+'pq' is 'opq' # true

#2.
c='op'
d='opq'
c+'q' is d # false
```
> 这是因为第一个例子中是在编译时(compile-time)求值的,被替换成了‘opq’;后面的例子是在运行时拼接的(run-time)，导致没有被主动intern

> 1.intern机制的优点是，在创建新的字符串对象时，会先在缓存池里面找是否有已经存在的值相同的对象（标识符，即只包含数字、字母、下划线的字符），如果有，则直接拿过来用（引用），避免频繁的创建和销毁内存，提升效率。
> 2.缺点是在拼接字符串时，或者在改动字符串时会极大的影响性能。原因是字符串在Python当中是不可变对象，所以对字符串的改动不是inplace（原地）操作，需要新开辟内存地址，新建对象。这也是为什么拼接字符串的时候不建议用‘+’而是用join()。join()是先计算出全部字符串的长度，然后再一一拷贝，仅仅创建一次对象。
### 6.如何将str转成dict类型
最初想用json.loads(),但是这时不行的，因为你的str不一定满足json格式`{"a": "1",...}`,所以就可以采用eval函数：
```
    str="{'a':'1', 'b':'2'}"
    d = eval(str)
    type(a) # >>> <class 'dict'>
```
> 但是为了更安全的写法，需要制定eval的后两个参数：eval(str,{},{}),后两个可以这样写`{}`
> ,更多参考：https://blog.csdn.net/lu8000/article/details/44217499?utm_source=blogxgwz8

### 7.Python如何判断某一天是星期几
```
import datetime


def getWeekDay(year, month, day):
    """
    今天是星期几
    :param year: 年份
    :param month: 月份
    :param day: 某一天
    :return: 返回星期几的简写
    """
    # year,month,day都必须是int类型
    return datetime.datetime(year, month, day).strftime("%a")

```
> 更多返回格式：
```
%a 星期几的简写 Weekday name, abbr.

%A 星期几的全称 Weekday name, full

%b 月分的简写 Month name, abbr.

%B 月份的全称 Month name, full

%c 标准的日期的时间串 Complete date andtime representation

%d 十进制表示的每月的第几天 Day of the month

%H 24小时制的小时 Hour (24-hour clock)

%I 12小时制的小时 Hour (12-hour clock)

%j 十进制表示的每年的第几天 Day of the year

%m 十进制表示的月份 Month number

%M 十时制表示的分钟数 Minute number

%S 十进制的秒数 Second number

%U 第年的第几周，把星期日做为第一天（值从0到53）Week number (Sunday first weekday)

%w 十进制表示的星期几（值从0到6，星期天为0）weekday number

%W 每年的第几周，把星期一做为第一天（值从0到53） Week number (Monday first weekday)

%x 标准的日期串 Complete date representation (e.g. 13/01/08)

%X 标准的时间串 Complete time representation (e.g. 17:02:10)

%y 不带世纪的十进制年份（值从0到99）Year number within century

%Y 带世纪部分的十制年份 Year number

%z，%Z 时区名称，如果不能得到时区名称则返回空字符。Name of time zone

%% 百分号

作者：长在香蕉树上的猫
链接：https://www.jianshu.com/p/39b288f0cbc7
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
### 8.Python中生成switch  

```
class switch(object):
    def __init__(self, value):
        self.value = value
        self.fall = False

    def __iter__(self):
        """Return the match method once, then stop"""
        yield self.match
        raise StopIteration

    def match(self, *args):
        """Indicate whether or not to enter a case suite"""
        if self.fall or not args:
            return True
        elif self.value in args:  # changed for v1.5, see below
            self.fall = True
            return True
        else:
            return False

```


> 上面使用生成器，在遍历switch(value)时生成一个match函数，用法如下：


```python
from .switch import switch
value = 'a'
for case in switch(value):
    # 遍历
    if case('a'):
        break
    if case('b'):
        break
    if case('c'):
        break;
    ...  
    
```