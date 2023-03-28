# Sqlite 数据库的基本使用

### 0-0 数据库是什么

> 借用百科上的话，为数据库是“按照数据结构来组织、存储和管理数据的仓库”。是一个长期存储在计算机内的、有组织的、可共享的、统一管理的大量数据的集合。
> 人话来说就是很多个数据存储在一个文件里

## 数据库都分为哪几个类型

### 1-0 在搞清楚什么是非关系型和关系型之前我们需要搞清一个概念 -字段-

> 字段就如同 python 的键值对一样，比如有一个字段 name，理所当然他的属性是 str，这就是字段其实就是键值对

### 1-1 正式开始

> 关系型数据库：存储的格式可以直观地反映实体间的关系。关系型数据库和常见的表格比较相似，关系型数据库中表与表之间是有很多复杂的关联关系的。
> 人话来说就是像表格一样每个表和数据都有关系，例如 MySQL 还有本教程教的 sqlite

> 非关系型数据库：例如，以键值对存储，且结构不固定，每一个元组可以有不一样的字段，每个元组可以根据需要增加一些自己的键值对，
> 这样就不会局限于固定的结构，可以减少一些时间和空间的开销。

> 举个栗子，比如我向非关系型数据库存储了一个对象，他的字段分别为，name，grade，并给了他一个对象 a，这时我又存储进了一个对象 b，有 old 这个字段，这时
> 数据库里就有了 a，b 两个对象来分别访问

> 这个为关系型数据库，首先我们需要创建一张表，表就是多个相同对象的集合，创建一个表 a，分别有 name，old 两个字段，而想要向表 a 存储对象，需要实例化所
> 有表 a 所有的字段，才能存储

## 通过一个普通 python 代码来带入数据库的世界

### 2-0 创建数据库

> 在文件夹中创建一个后缀为 sqlite3 的文件
>
> 然后可以使用 pycharm 的数据库功能访问 (如图)
>
> 点击确定
> ![image](https://github.com/MRSlouzk/Nonebot-plugintutorials/blob/main/imgs/sqlite%E5%88%9B%E5%BB%BA.png)

### 2-1 创建表

> 在 data.sqlite3 下会有 main，而 main 里会有表这个，右键表，会弹出来下面的选项
> ![image](https://github.com/MRSlouzk/Nonebot-plugintutorials/blob/main/imgs/sqlite%E5%88%9B%E5%BB%BA%E8%A1%A8.png)

> 接下来创建表，这个表中有五个字段，分别为 studentID name old grade class 这五个
> 你会发现我把 studentID 设置为主键，非 null，唯一，自动增加那这些都是什么意思捏？
>
> 主键：能唯一表达这个表中的每一项
>
> 非 null：就是不为 null，什么你问我 null 是什么？百度下()
>
> 唯一：就是表面意思
>
> 自动增加：如果类型是 integer，会自动+1
> 但你又会发现 text，integer 这些类型，接下来我们会讲 sqlite 的基本类型
>
> NULL：表示值为空(null)值
>
> INTEGER：表示值是一个有符号整数，根据值的大小存储在 1,2,3,4,6 或 8 个字节中
>
> REAL：表示值是一个浮点值，存储为 8 位 IEEE 浮点数。
>
> text：表示值是一个文本字符串，使用数据库编码(utf-8，utf-16be 或 utf-16le)存储
>
> BLOB：表示值是一个数据块，与输入的数据完全相同（人话说就是二进制数据，可以存储图片，音乐等）
> ![image](https://github.com/MRSlouzk/Nonebot-plugintutorials/blob/main/imgs/sqlite%E5%88%9B%E5%BB%BA%E8%A1%A82.png)

### 2-2 创建对象（常说的表中的行）

模拟数据库网站：http://sqlfiddle.com/
在左边写创建数据库等语句
然后点击下边的 build 进行初始胡
右边是写操作语句，比如 insert，select 这些

#### 基础 sql 脚本(关系型数据库中基本通用)

> insert 语句：向一个表中添加行，格式：insert into TABLE values (VALUE1,VALUE2,VALUE3,...VALUEN); (如果表中有这个主键会报错哦，下面会教条件语句来防止)
>
> select 语句：在控制台中输出这个表中的所有行以及他的字段，格式：select \* from TABLE_NAME;
>
> drop 语句：删除表的一个语句，格式：drop table TABLE_NAME;
>
> create 语句：创建表的语句，格式：create table TABLE_NAME(NAME_1 TYPE_1,NAME_2 TYPE 2.........);
>
> where 语句：寻找一个表中符合条件的一项，例如我们要找一个主键 ID 为 3 的可以这么写 select \* from TABLE_NAME where ID=3; (注意 sqlite 中的等于是一个等于号)
>
> delete 语句：删除表中的一项，这时就可以用到 where 语句了，例如找到 ID 为 3 之后我们要删除它，可以这么写 delete from TABLE_NAME where ID=3;
>
> and 连接词，需要两边都为 true，假如在一个 student 表中需要查询 name 是王五，id 是 1 的那行，我们可以使用 select \* from student where name="王五" and id = 1;
>
> or 连接词：需要两边任意一个为 true，语法与 and 连接词一样
>
> order by：？？？等待会再画

### 2-3 使用 python 的驱动库进行对数据库的操作
