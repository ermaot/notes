__init__.py文件，这文件是干嘛的，为什么要在引用的目录下加这个文件？
python 在执行 import 语句时，到底进行了什么操作，按照 python 的文档，它执行了如下操作：
- 第 1步，创建一个新的，空的 module 对象（它可能包含多个 module）
- 第 2步，把这个 module 对象插入 sys.module 中
- 第 3步，装载 module 的代码（如果需要，首先必须编译）
- 第 4步，执行新的 module 中对应的代码。
- 在执行第 3步时，首先要找到 module 程序所在的位置，搜索的顺序是：
1. 当前路径 （以及从当前目录指定的 sys.path） 
2. 然后是 PYTHONPATH
3. 然后是 python 的安装设置相关的默认路径。
正因为存在这样的顺序，如果当前路径或PYTHONPATH中存在与标准module同样的module，则会覆盖标准module。也就是说，如果当前目录下存在xml.py，那么执行 import xml时，导入的是当前目录下的module，而不是系统标准的xml。了解了这些，我们就可以先构建一个 package，以普通 module 的方式导入，就可以直接访问此 package中的各个 module 了。python 中的 package 必须包含一个__init__.py 的文件

- _init__.py 文件中可以有内容；我们在导入一个包时，实际上导入了它的__init__.py文件