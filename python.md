1. 读取文件时的第一行第一个字符是"\ufeff",而不是内容

   解决方法:在读取B.txt 时,指定编码方式为 "utf-8-sig"即可. ”\ufeff“叫BOM("ByteOrder Mark")用来声明该文件的编码信息."uft-8-sig"中sig全拼为 signature 也就是"带有签名的utf-8", 因此"utf-8-sig"读取带有BOM的"utf-8文件时"会把BOM单独处理,与文本内容隔离开,也是我们期望的结果.

   ```python
   with open("B.txt", 'r', encoding='utf-8-sig') as f:
       line = f.readline()[0:-1]  #去掉末尾换行符
       print(line)
       with open("%s.txt" % line, 'r', encoding='utf-8') as f1:
           print(f1.readline())
       f1.close()
   f.close()
   ```



## python基础

注释:

单行#

多行是单引号或者双引号三个;

## 打包发布:

1. 安装程序:

   ```python
   pip install pyinstaller
   ```



2. 打包要发布的脚本:

   ```
   pyinstaller -F -w  yourscript
   ```

   参数说明:

   -F 表示生成单个可执行文件
   -w 表示去掉控制台窗口，这在GUI界面时非常有用。不过如果是命令行程序的话那就把这个选项删除吧！
   -p 表示你自己自定义需要加载的类路径，一般情况下用不到
   -i 表示可执行文件的图标

3.
## partial vs lambda
1. partial
 使用时需要引入:

```py
from functools import partial
```

主要用于固定某函数的某一个参数值, 并形成新的函数.
eg:

```py
def add(a,b)
  return a+b;
from functools import partial
#以下写法,固定参数a值为1, 新函数addOne值需要一个参数,传入的参数将认为是b的值
addOne = partial(add, 1)
addOne(1) #将打印2
```
此函数在for语句中遍历并创建tkinter的组件时制定Command函数时很有用, 如果不用将导致传入参数并非指定时的变量值;
2. lambda
eg:
```py
g = lambda x: x+1
```
相当于简写的函数:
```py
def g(x):
  return x+1
```
## Variables in Python
tkinter中提供了一些可以追踪变量值修改的变量类型, 比如StringVar
StringVar主要有以下几种方法:
1. set(new val)
2. get()
3. trace(mode,callback)
  mode可以是"r"或者"w"
eg:
```python
def callback(*args):
  print "variable changed!"

var = StringVar()
var.trace("w", callback)
var.set("hello")
```
