## innodb 历史
## 源码版本
- Heikki Tuuri： innodb创始人，赫尔辛基大学
- mysql 创始人：Michael Widenius（monty）
- 刚开始innodb闭源，innobase 在找买家
- 被sun收购，然后sun被oracle收购
- innodb 代码行数快速增加，但内核保持稳定

innodb版本 | 代码行数
---|---
3.23.49 | 107653
5.1.60 | 164171
5.1.60（innodb plugin） | 164171
5.7.3-m13（预览版） | 318100


## 源码风格
#### 源码结构
- 存储引擎接口文件采用C++，少量汇编，大部分采用C
- 模块划分清晰，大部分模块都在.c文件中
- 每个模块放在单独文件夹下，文件命名规则“模块名0子模块名.c”
- .h统一放include文件夹
- .ic文件为每个模块定义了内联函数
#### 代码风格
- K&R风格，起始大括号放行尾
- 每一个文件开头都有简短的注释说明功能
- 每一个函数都有注释说明功能
- 变量用下划线分隔单词，用小写；宏和枚举使用大写
## 代码编译
- 支持各类型操作系统，如linux，solaris，windows，freebsd等
- windows使用visual studio
- linux使用vim + gdb
## 阅读源码次序
![innodb模块划分](pic/innodb%20%E5%86%85%E6%A0%B8%E6%A6%82%E8%A7%881.png)

#### 1. 推荐从下至上，从易到难阅读

#### 2. 最下方是最基本的模块

- File manager主要封装了 InnodB存储引擎对于文件的各类操作，如读、写、异步IO等。 

- Concurrency Manager模块主要封装了引擎内部使用的各类mutex和 latch. 

- Common Utility模块用于一些基本数据结构与算法的定义，如链表、哈希表等。

#### 3. 图中间虛线标注的部分
可以理解为 InnodB存储引擎的内核实现部分，也就是 InnodB存储引擎事务、锁、缓存、日志、存储、索引的实现模块
#### 4. 图最上面的两层是接口层
通过这些接口实现上层与存储引擎内部的互动。 Innode存储引擎可以不依赖 MySQL数据库，而作为一个嵌入式数据库存在，因此还存在嵌入式的API接口

