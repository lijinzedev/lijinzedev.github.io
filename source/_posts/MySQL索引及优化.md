---
title: MySQL索引及优化
top: false
cover: false
toc: true
mathjax: true
categories:
  - mysql
tags:
  - mysql
date: 2022-04-08 10:15:36
password:
summary:
---

# 索引的数据结构

## 1 索引是什么

**维基百科对数据库索引的定义：**

> 数据库索引，是数据库管理系统（DBMS）中一个排序的数据结构，以协助快速查询、 更新数据库表中数据。 

**怎么理解这个定义呢？**

![image-20220408110819844](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081108916.png)

首先数据是以文件的形式存放在磁盘上面的，每一行数据都有它的磁盘地址。如果 没有索引的话，要从 500 万行数据里面检索一条数据，只能依次遍历这张表的全部数据， 直到找到这条数据。

但是有了索引之后，只需要在索引里面去检索这条数据就行了，因为它是一种特殊 的专门用来快速检索的数据结构，我们找到数据存放的磁盘地址以后，就可以拿到数据 了。

就像我们从一本 500 页的书里面去找特定的一小节的内容，肯定不可能从第一页开 始翻。那么这本书有专门的目录，它可能只有几页的内容，它是按页码来组织的，可以 根据拼音或者偏旁部首来查找，只要确定内容对应的页码，就能很快地找到我们想要的 内容。

## 2 索引的优点以及缺点

### 2.1 优点

1. 类似大学图书馆建书目索引，提高数据检索的效率，降低==数据库的IO成本== ，这也是创建索引最主要的原因。 

2. 通过创建唯一索引，可以保证数据库表中每一行 ==数据的唯一性== 。

3. 在实现数据的参考完整性方面，可以 加速表和表之间的连接 。换句话说，对于有依赖关系的子表和父表联合查询时，可以提高查询速度。 

4. 在使用分组和排序子句进行数据查询时，可以显著 减少查询中分组和排序的时间 ，降低了CPU的消耗。

### 2.2 缺点

增加索引也有许多不利的方面，主要表现在如下几个方面： 

1. 创建索引和维护索引要 耗费时间 ，并且随着数据量的增加，所耗费的时间也会增加。 
2. 索引需要占 磁盘空间 ，除了数据表占数据空间之外，每一个索引还要占一定的物理空间， 存储在磁盘上 ，如果有大量的索引，索引文件就可能比数据文件更快达到最大文件尺寸。
3. 虽然索引大大提高了查询速度，同时却会 降低更新表的速度 。当对表中的数据进行增加、删除和修改的时候，索引也要动态地维护，这样就降低了数据的维护速度。

因此，选择使用索引时，需要综合考虑索引的优点和缺点。

## 3 索引的推演

### 3.1 二分查找

双十一过去之后，你女朋友跟你玩了一个猜数字的游戏。 猜猜我昨天买了多少钱，给你五次机会。 10000？低了。30000？高了。接下来你会猜多少？ 20000。为什么你不猜 11000，也不猜 29000 呢？

其实这个就是二分查找的一种思想，也叫折半查找，每一次，我们都把候选数据缩小了一半。如果数据已经排过序的话，这种方式效率比较高。

所以第一个，我们可以考虑用有序数组作为索引的数据结构。

==有序数组==的等值查询和比较查询效率非常高，但是更新数据的时候会出现一个问题， 可能要挪动大量的数据（改变index），所以只适合存储静态的数据。

为了支持频繁的修改，比如插入数据，我们需要采用链表。链表的话，如果是单链表，它的查找效率还是不够高。

所以，有没有可以使用二分查找的链表呢？

为了解决这个问题，BST（Binary Search Tree）也就是我们所说的二叉查找树诞生了。

### 3.2 二叉查找树（BST Binary Search Tree）

![image-20220408115923990](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081159070.png)

左子树所有的节点都小于父节点，右子树所有的节点都大于父节点。投影到平面以后，就是一个有序的线性表。

二叉查找树既能够实现快速查找，又能够实现快速插入。

**但是二叉查找树有一个问题：**

就是它的查找耗时是和这棵树的深度相关的，在最坏的情况下时间复杂度会退化成O(n)。

**什么情况是最坏的情况呢？**

我们打开这样一个网站来看一下，这里面有各种各样的数据结构的动态演示，包括BST 二叉查找树：

https://www.cs.usfca.edu/~galles/visualization/Algorithms.html

还是刚才的这一批数字，如果我们插入的数据刚好是有序的，2、6、11、13、17、22。

这个时候我们的二叉查找树变成了什么样了呢？

它会变成链表（我们把这种树叫做“斜树”），这种情况下不能达到加快检索速度的目的，和顺序查找效率是没有区别的。

![image-20220408135415573](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081354658.png)

**造成它倾斜的原因是什么呢？**

因为左右子树深度差太大，这棵树的左子树根本没有节点——也就是它不够平衡。

所以，我们有没有左右子树深度相差不是那么大，更加平衡的树呢？

这个就是平衡二叉树，叫做Balanced binary search trees，或者 AVL 树（AVL 是发明这个数据结构的人的名字）。

### 3.3 平衡二叉树（AVL Tree）（左旋、右旋）

AVL Trees (Balanced binary search trees)

平衡二叉树的定义：左右子树深度差绝对值不能超过 1。

是什么意思呢？比如左子树的深度是 2，右子树的深度只能是 1 或者 3。

这个时候我们再按顺序插入 1、2、3、4、5、6，一定是这样，不会变成一棵“斜树”。

![image-20220408140830633](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081408709.png)

那它的平衡是怎么做到的呢？怎么保证左右子树的深度差不能超过 1 呢？

https://www.cs.usfca.edu/~galles/visualization/AVLtree.html

插入 1、2、3。

我们注意看：当我们插入了 1、2 之后，如果按照二叉查找树的定义，3 肯定是要在2 的右边的，这个时候根节点 1 的右节点深度会变成 2，但是左节点的深度是 0，因为它没有子节点，所以就会违反平衡二叉树的定义。

那应该怎么办呢？因为它是右节点下面接一个右节点，右-右型，所以这个时候我们要把 2 提上去，这个操作叫做左旋。

![image-20220408141002946](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081410014.png)

同样的，如果我们插入 7、6、5，这个时候会变成左左型，就会发生右旋操作，把 6 提上去。

![image-20220408141041906](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081410963.png)

所以为了保持平衡，AVL 树在插入和更新数据的时候执行了一系列的计算和调整的操作。

平衡的问题我们解决了，那么平衡二叉树作为索引怎么查询数据？

在平衡二叉树中，一个节点，它的大小是一个固定的单位，作为索引应该存储什么内容？

**它应该存储三块的内容：**

第一个是索引的键值。比如我们在id 上面创建了一个索引，我在用where id =1 的条件查询的时候就会找到索引里面的id 的这个键值。

第二个是数据的磁盘地址，因为索引的作用就是去查找数据的存放的地址。

第三个，因为是二叉树，它必须还要有左子节点和右子节点的引用，这样我们才能找到下一个节点。比如大于 26 的时候，走右边，到下一个树的节点，继续判断。

![image-20220408141157077](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081411168.png)

如果是这样存储数据的话，我们来看一下会有什么问题。

在分析用AVL 树存储索引数据之前，我们先来学习一下InnoDB 的逻辑存储结构

#### 3.3.1 .InnoDB 逻辑存储结构

https://dev.mysql.com/doc/refman/5.7/en/innodb-disk-management.html 

https://dev.mysql.com/doc/refman/5.7/en/innodb-file-space.html

MySQL 的存储结构分为 5 级：表空间、段、簇、页、行。

![image-20220408141445113](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081415660.png)

##### 表空间 Table Space

表空间可以看做是 InnoDB 存储引擎逻辑结构的最高层，所有的数据都存放在表空间中。分为：系统表空间、独占表空间、通用表空间、临时表空间、Undo 表空间。

##### 段 Segment

表空间是由各个段组成的，常见的段有数据段、索引段、回滚段等，段是一个逻辑的概念。一个ibd 文件（独立表空间文件）里面会由很多个段组成。

创建一个索引会创建两个段，一个是索引段：leaf node segment，一个是数据段： non-leaf node segment。索引段管理非叶子节点的数据。数据段管理叶子节点的数据。也就是说，一个表的段数，就是索引的个数乘以 2。

##### 簇 Extent

一个段（Segment）又由很多的簇（也可以叫区）组成，每个区的大小是 1MB（64 个连续的页）。

每一个段至少会有一个簇，一个段所管理的空间大小是无限的，可以一直扩展下去， 但是扩展的最小单位就是簇。

##### 页 Page

为了高效管理物理空间，对簇进一步细分，就得到了页。簇是由连续的页（Page） 组成的空间，一个簇中有 64 个连续的页。 （1MB／16KB=64）。这些页面在物理上和逻辑上都是连续的。

跟大多数数据库一样，InnoDB 也有页的概念（也可以称为块），每个页默认 16KB。页是InnoDB 存储引擎磁盘管理的最小单位，通过innodb_page_size 设置。

一个表空间最多拥有 2^32 个页，默认情况下一个页的大小为 16KB，也就是说一个表空间最多存储 64TB 的数据。

> 注意，文件系统中，也有页的概念。
>
> 操作系统和内存打交道，最小的单位是页Page。文件系统的内存页通常是 4K。

![image-20220408142130344](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081421420.png)

```mysql
SHOW VARIABLES LIKE 'innodb_page_size';
```

假设一行数据大小是 1K，那么一个数据页可以放 16 行这样的数据。

举例：一个页放 3 行数据。

![image-20220408142215734](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081422792.png)

往表中插入数据时，如果一个页面已经写完，产生一个新的叶页面。如果一个簇的所有的页面都被用完，会从当前页面所在段新分配一个簇。

如果数据不是连续的，往已经写满的页中插入数据，会导致叶页面分裂：

![连续](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081422209.png)

![不连续](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081422668.png)

##### 行 Row

InnoDB 存储引擎是面向行的（row-oriented），也就是说数据的存放按行进行存放。

https://dev.mysql.com/doc/refman/5.7/en/innodb-row-format.html

Antelope[ˈæntɪləʊp]（羚羊）是InnoDB 内置的文件格式，有两种行格式： 

* REDUNDANT[rɪˈdʌndənt] Row Format

* ZCOMPACT Row Format（5.6 默认）

Barracuda[ˌbærəˈkjuːdə]（梭子鱼）是 InnoDB Plugin 支持的文件格式，新增了两种行格式：

* DYNAMIC Row Format（5.7 默认）
* COMPRESSED Row Format

| 文件格式        | 行格式                | 描述                                                         |
| --------------- | --------------------- | ------------------------------------------------------------ |
| Antelope        | ROW_FORMAT=COMPACT    | Compact 和 redumdant 的区别在就是在于首部的存                |
| （Innodb-base） | ROW_FORMAT=REDUNDANT  | 存内容区别。compact 的存储格式为首部为一个非NULL 的变长字    |
|                 |                       | 段长度列表                                                   |
|                 |                       | redundant 的存储格式为首部是一个字段长度偏移                 |
|                 |                       | 列表（每个字段占用的字节长度及其相应的位移）。               |
|                 |                       | 在 Antelope 中对于变长字段，低于 768 字节的，不              |
|                 |                       | 会进行 overflow page 存储，某些情况下会减少结果              |
|                 |                       | 集 IO.                                                       |
| Barracuda       | ROW_FORMAT=DYNAMIC    | 这两者主要是功能上的区别功能上的。	另外在行               |
| (innodb-plugin) | ROW_FORMAT=COMPRESSED | 里的变长字段和 Antelope 的区别是只存 20 个字节， 其它的 overflow page 存储。 |

```mysql
show variables like "%innodb_file_format%"; 
SET GLOBAL innodb_file_format=Barracuda;
```

![image-20220408142645284](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081426354.png)

**在创建表的时候可以指定行格式。**

```mysql
CREATE TABLE tf1
(c1 INT PRIMARY KEY) ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
```

**查看行格式：**

```mysql
SHOW TABLE STATUS LIKE 'student' \G;
```

![image-20220408142736649](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081427727.png)

#### 3.3.2 .AVL 树用于存储索引数据

首先，索引的数据，是放在硬盘上的。查看数据和索引的大小：

```mysql
select
CONCAT(ROUND(SUM(DATA_LENGTH/1024/1024),2),'MB') AS data_len,
CONCAT(ROUND(SUM(INDEX_LENGTH/1024/1024),2),'MB') as index_len
from information_schema.TABLES
where table_schema='test' and table_name='user_innodb';
```

当我们用树的结构来存储索引的时候，访问一个节点就要跟磁盘之间发生一次 IO。InnoDB 操作磁盘的最小的单位是一页（或者叫一个磁盘块），大小是 16K(16384 字节)。

那么，==一个树的节点就是 16K 的大小。==

如果我们一个节点只存一个键值+数据+引用，例如整形的字段，可能只用了十几个或者几十个字节，它远远达不到 16K 的容量，所以==访问一个树节点，进行一次 IO== 的时候， 浪费了大量的空间。

所以如果每个节点存储的数据太少，从索引中找到我们需要的数据，就要访问更多的节点，意味着跟磁盘交互次数就会过多。

如果是机械硬盘时代，每次从磁盘读取数据需要 10ms 左右的寻址时间，交互次数越多，消耗的时间就越多。

![](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081440628.png)

比如上面这张图，我们一张表里面有 6 条数据，当我们查询id=37 的时候，要查询 两个子节点，就需要跟磁盘交互 3 次，如果我们有几百万的数据呢？这个时间更加难以估计。

**所以我们的解决方案是什么呢？**

第一个就是让每个节点存储更多的数据。

第二个，节点上的关键字的数量越多，我们的指针数也越多，也就是意味着可以有更多的分叉（我们把它叫做“路数”）。

因为分叉数越多，树的深度就会减少（根节点是 0）。

这样，我们的树是不是从原来的高瘦高瘦的样子，变成了矮胖矮胖的样子？ 

这个时候，我们的树就不再是二叉了，而是多叉，或者叫做多路。

### 3.4.多路平衡查找树（B Tree）（分裂、合并）

**Balanced Tree**

这个就是我们的多路平衡查找树，叫做B Tree（B 代表平衡）。

跟AVL 树一样，B 树在枝节点和叶子节点存储键值、数据地址、节点引用。

它有一个特点：分叉数（路数）永远比关键字数多 1。比如我们画的这棵树，每个节点存储两个关键字，那么就会有三个指针指向三个子节点。

![image-20220408144408829](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081444919.png)

**B Tree 的查找规则是什么样的呢？** 

**比如我们要在这张表里面查找 15。**

因为 15 小于 17，走左边。

因为 15 大于 12，走右边

在磁盘块 7 里面就找到了 15，只用了 3 次IO

**这个是不是比 AVL 树效率更高呢？**

**那B Tree 又是怎么实现一个节点存储多个关键字，还保持平衡的呢？跟 AVL 树有什么区别？**

https://www.cs.usfca.edu/~galles/visualization/Algorithms.html

比如Max Degree（路数）是 3 的时候，我们插入数据 1、2、3，在插入 3 的时候， 本来应该在第一个磁盘块，但是如果一个节点有三个关键字的时候，意味着有 4 个指针，

子节点会变成 4 路，所以这个时候必须进行分裂。把中间的数据 2 提上去，把 1 和 3 变成 2 的子节点。

如果删除节点，会有相反的合并的操作。

注意这里是分裂和合并，跟 AVL 树的左旋和右旋是不一样的。我们继续插入 4 和 5，B Tree 又会出现分裂和合并的操

![image-20220408144534134](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081445192.png)

从这个里面我们也能看到，在更新索引的时候会有大量的索引的结构的调整，所以解释了为什么我们不要在频繁更新的列上建索引，或者为什么不要更新主键。

节点的分裂和合并，其实就是InnoDB 页的分裂和合并。

### 3.5.B+树（加强版多路平衡查找树）

B Tree 的效率已经很高了，为什么 MySQL 还要对 B Tree 进行改良，最终使用了B+Tree 呢？

总体上来说，这个B 树的改良版本解决的问题比B Tree 更全面。

我们来看一下InnoDB 里面的B+树的存储结构：

![image-20220408144644926](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081446002.png)

**MySQL 中的B+Tree 有几个特点：**

1、它的关键字的数量是跟路数相等的；

2、B+Tree 的根节点和枝节点中都不会存储数据，只有叶子节点才存储数据。搜索到关键字不会直接返回，会到最后一层的叶子节点。比如我们搜索 id=28，虽然在第一层直接命中了，但是全部的数据在叶子节点上面，所以我还要继续往下搜索，一直到叶子节点。

**举个例子：假设一条记录是 1K，一个叶子节点（一页）可以存储 16 条记录。非叶子节点可以存储多少个指针？**

假设索引字段是bigint 类型，长度为 8 字节。指针大小在 InnoDB 源码中设置为6 字节，这样一共 14 字节。非叶子节点（一页）可以存储 16384/14=1170 个这样的

单元（键值+指针），代表有 1170 个指针。

树 深 度 为 2 的 时 候 ， 有 1170^2  个 叶 子 节 点 ， 可 以 存 储 的 数 据 为1170*1170*16=21902400。

![image-20220408145214396](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081452457.png)

在查找数据时一次页的查找代表一次 IO，也就是说，一张 2000 万左右的表，查询数据最多需要访问 3 次磁盘。

所以在 InnoDB 中 B+ 树深度一般为 1-3 层，它就能满足千万级的数据存储。

3、B+Tree 的每个叶子节点增加了一个指向相邻叶子节点的指针，它的最后一个数据会指向下一个叶子节点的第一个数据，形成了一个有序链表的结构。

4、它是根据左闭右开的区间 [ )来检索数据。

**我们来看一下B+Tree 的数据搜寻过程：**

1） 比如我们要查找 28，在根节点就找到了键值，但是因为它不是页子节点，所以会继续往下搜寻，28 是[28,66)的左闭右开的区间的临界值，所以会走中间的子节点，然后继续搜索，它又是[28,34)的左闭右开的区间的临界值，所以会走左边的子节点，最后在叶子节点上找到了需要的数据。

2） 第二个，如果是范围查询，比如要查询从 22 到 60 的数据，当找到 22 之后，只需要顺着节点和指针顺序遍历就可以一次性访问到所有的数据节点，这样就极大地提高了区间查询效率（不需要返回上层父节点重复遍历查找）。

**总结一下，InnoDB 中的B+Tree 的特点：**

1) 它是B Tree 的变种，B Tree 能解决的问题，它都能解决。B Tree 解决的两大问题是什么？（每个节点存储更多关键字；路数更多）

2) 扫库、扫表能力更强（如果我们要对表进行全表扫描，只需要遍历叶子节点就可以了，不需要遍历整棵B+Tree 拿到所有的数据）

3) B+Tree 的磁盘读写能力相对于B Tree 来说更强（根节点和枝节点不保存数据区， 所以一个节点可以保存更多的关键字，一次磁盘加载的关键字更多）

4) 排序能力更强（因为叶子节点上有下一个数据区的指针，数据形成了链表）

5) 效率更加稳定（B+Tree 永远是在叶子节点拿到数据，所以IO 次数是稳定的）

### 3.6.为什么不用红黑树？

红黑树也是BST 树，但是不是严格平衡的。

**必须满足 5 个约束：**

1、节点分为红色或者黑色。

2、根节点必须是黑色的。

3、叶子节点都是黑色的NULL 节点。

4、红色节点的两个子节点都是黑色（不允许两个相邻的红色节点）。

5、从任意节点出发，到其每个叶子节点的路径中包含相同数量的黑色节点。插入：60、56、68、45、64、58、72、43、49

![image-20220408145440176](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081454239.png)

**基于以上规则，可以推导出：**

从根节点到叶子节点的最长路径（红黑相间的路径）不大于最短路径（全部是黑色节点）的 2 倍。

**为什么不用红黑树？**1、只有两路；2、不够平衡。

红黑树一般只放在内存里面用。例如Java 的 TreeMap。

### 3.7.索引方式：真的是用的 B+Tree 吗？

在Navicat 的工具中，创建索引，索引方式有两种，Hash 和B Tree。

HASH：以KV 的形式检索数据，也就是说，它会根据索引字段生成哈希码和指针， 指针指向数据。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081455573.png)

**哈希索引有什么特点呢？**

第一个，它的时间复杂度是O(1)，查询速度比较快。因为哈希索引里面的数据不是按顺序存储的，所以不能用于排序。

第二个，我们在查询数据的时候要根据键值计算哈希码，所以它只能支持等值查询（= IN），不支持范围查询（> <	>= <= between and）。

另外一个就是如果字段重复值很多的时候，会出现大量的哈希冲突（采用拉链法解决），效率会降低。

**问题： InnoDB 可以在客户端创建一个索引，使用哈希索引吗？**

https://dev.mysql.com/doc/refman/5.7/en/innodb-introduction.html

InnoDB utilizes hash indexes internally for its Adaptive Hash Index feature

直接翻译过来就是：InnoDB 内部使用哈希索引来实现自适应哈希索引特性。

这句话的意思是 InnoDB 只支持显式创建 B+Tree 索引，对于一些热点数据页， InnoDB 会自动建立自适应Hash 索引，也就是在B+Tree 索引基础上建立Hash 索引， 这个过程对于客户端是不可控制的，隐式的。

我们在Navicat 工具里面选择索引方法是哈希，但是它创建的还是 B+Tree 索引，这个不是我们可以手动控制的。

上次课我们说到 buffer pool 里面有一块区域是 Adaptive Hash Index 自适应哈希索引，就是这个。

这个开关默认是ON：

```mysql
show variables like 'innodb_adaptive_hash_index';
```

从存储引擎的运行信息中可以看到：

```mysql
show engine innodb status\G
```

![image-20220408145758714](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081457759.png)

因为B Tree 和B+Tree 的特性，它们广泛地用在文件系统和数据库中，例如Windows 的HPFS 文件系统，Oracel、MySQL、SQLServer 数据库。

## 4.B+Tree 落地形式

### 4.1.MySQL 架构

MySQL 是一个支持插件式存储引擎的数据库。在MySQL 里面，每个表在创建的时候都可以指定它所使用的存储引擎。

这里我们主要关注一下最常用的两个存储引擎，MyISAM 和InnoDB 的索引的实现。

### 4.2.MySQL 数据存储文件

首先，MySQL 的数据都是文件的形式存放在磁盘中的，我们可以找到这个数据目录的地址。在MySQL 中有这么一个参数，我们来看一下：

```mysql
show VARIABLES LIKE 'datadir';
```

每个数据库有一个目录，我们新建了一个叫做test 的数据库，那么这里就有一个test 的文件夹。

这个数据库里面我们又建了 5 张表：archive、innodb、memory、myisam、csv。我们进入test 的目录，发现这里面有一些跟我们创建的表名对应的文件。

在这里我们能看到，每张InnoDB 的表有两个文件（.frm 和.ibd），MyISAM 的表有三个文件（.frm、.MYD、.MYI）。

![image-20220408145946506](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081459566.png)

有一个是相同的文件，.frm。 .frm 是MySQL 里面表结构定义的文件，不管你建表的时候选用任何一个存储引擎都会生成。

我们主要看一下其他两个文件是怎么实现MySQL 不同的存储引擎的索引的。

#### 4.2.1.MyISAM

在MyISAM 里面，另外有两个文件：

一个是.MYD 文件，D 代表Data，是 MyISAM 的数据文件，存放数据记录，比如我们的user_myisam 表的所有的表数据。

一个是.MYI 文件，I 代表Index，是MyISAM 的索引文件，存放索引，比如我们在id 字段上面创建了一个主键索引，那么主键索引就是在这个索引文件里面。

也就是说，在MyISAM 里面，索引和数据是两个独立的文件。

**那我们怎么根据索引找到数据呢？**

MyISAM 的 B+Tree 里面，叶子节点存储的是数据文件对应的磁盘地址。所以从索引文件.MYI 中找到键值后，会到数据文件.MYD 中获取相应的数据记录。

![image-20220408150256993](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081502079.png)

**这里是主键索引，如果是辅助索引，有什么不一样呢？** 

在MyISAM 里面，辅助索引也在这个.MYI 文件里面。	

辅助索引跟主键索引存储和检索数据的方式是没有任何区别的，一样是在索引文件里面找到磁盘地址，然后到数据文件里面获取数据。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081503336.png)

#### 4.2.2.InnoDB

**InnoDB 只有一个文件（.ibd 文件），那索引放在哪里呢？**

在 InnoDB 里面，它是以主键为索引来组织数据的存储的，所以索引文件和数据文件是同一个文件，都在.ibd 文件里面。

在InnoDB 的主键索引的叶子节点上，它直接存储了我们的数据。

![image-20220408150426274](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081504353.png)

##### **什么叫做聚集索引（聚簇索引）？**

就是==索引键值==的逻辑顺序跟==表数据行==的物理存储顺序是一致的。（比如字典的目录是按拼音排序的，内容也是按拼音排序的，按拼音排序的这种目录就叫聚集索引）。

在InnoDB 里面，它组织数据的方式叫做叫做（聚集）索引组织表（clustered index organize table），所以主键索引是聚集索引，非主键都是非聚集索引。

**如果InnoDB 里面主键是这样存储的，那主键之外的索引，比如我们在 name 字段上面建的普通索引，又是怎么存储和检索数据的呢？**

![image-20220408150545371](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081506021.png)

InnoDB 中，主键索引和辅助索引是有一个主次之分的。

辅助索引存储的是辅助索引和主键值。如果使用辅助索引查询，会根据主键值在主键索引中查询，最终取得数据。

比如我们用 name 索引查询 name= '青山'，它会在叶子节点找到主键值，也就是id=1，然后再到主键索引的叶子节点拿到数据。

**为什么在辅助索引里面存储的是主键值而不是主键的磁盘地址呢？如果主键的数据类型比较大，是不是比存地址更消耗空间呢？**

我们前面说到B Tree 是怎么实现一个节点存储多个关键字，还保持平衡的呢？

是因为有分叉和合并的操作，这个时候键值的地址会发生变化，所以在辅助索引里面不能存储地址。

**另一个问题，如果一张表没有主键怎么办？**

1、如果我们定义了主键(PRIMARY KEY)，那么 InnoDB 会选择主键作为聚集索引。

2、如果没有显式定义主键，则 InnoDB 会选择第一个不包含有NULL 值的唯一索引作为主键索引。

3、如果也没有这样的唯一索引，则 InnoDB 会选择内置 6 字节长的ROWID 作为隐藏的聚集索引，它会随着行记录的写入而主键递增。

```mysql
select _rowid name from t2;
```

## 5.索引使用原则

我们容易有以一个误区，就是在经常使用的查询条件上都建立索引，索引越多越好， 那到底是不是这样呢？

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081507742.png)

### 5.1.列的离散（sàn）度

第 一 个 叫 做 列 的 离 散 度 ， 我 们 先 来 看 一 下 列 的 离 散 度 的 公 式 ：

count(distinct(column_name)) : count(*)，列的全部不同值和所有数据行的比例。

数据行数相同的情况下，分子越大，列的离散度就越高。

![image-20220408151232356](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081512441.png)

简单来说，如果列的重复值越多，离散度就越低，重复值越少，离散度就越高。

了解了离散度的概念之后，我们再来思考一个问题，我们在name 上面建立索引和在gender 上面建立索引有什么区别。

当我们用在gender 上建立的索引去检索数据的时候，由于重复值太多，需要扫描的行数就更多。例如，我们现在在gender 列上面创建一个索引，然后看一下执行计划。

```mysql
ALTER TABLE user_innodb DROP INDEX idx_user_gender;
ALTER TABLE user_innodb ADD INDEX idx_user_gender (gender);	-- 耗时比较久
EXPLAIN SELECT * FROM `user_innodb` WHERE gender = 0;
```

![image-20220408152611580](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081526669.png)

```mysql
show indexes from user_innodb;
```

而name 的离散度更高，比如“青山”的这名字，只需要扫描一行。

```mysql
ALTER TABLE user_innodb DROP INDEX idx_user_name; ALTER TABLE user_innodb ADD INDEX idx_user_name (name); EXPLAIN SELECT * FROM `user_innodb` WHERE name = '青山';
```

![image-20220408152636128](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721784.png)

查看表上的索引，Cardinality [kɑ:dɪ'nælɪtɪ] 代表基数，代表预估的不重复的的数量。索引的基数与表总行数越接近，列的离散度就越高。

```mysql
show indexes from user_innodb;
```

如果在B+Tree 里面的重复值太多，MySQL 的优化器发现走索引跟使用全表扫描差不了多少的时候，就算建了索引，也不一定会走索引。

https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html![image-20220408152835312](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081528411.png)

这个给我们的启发是什么？建立索引，要使用离散度（选择度）更高的字段。

### 5.2 联合索引最左匹配

前面我们说的都是针对单列创建的索引，但有的时候我们的多条件查询的时候，也会建立联合索引。单列索引可以看成是特殊的联合索引。

比如我们在user 表上面，给name 和phone 建立了一个联合索引。

```mysql
ALTER TABLE user_innodb DROP INDEX comidx_name_phone;
ALTER TABLE user_innodb add INDEX comidx_name_phone (name,phone);
```

联合索引在B+Tree 中是复合的数据结构，它是按照从左到右的顺序来建立搜索树的（name 在左边，phone 在右边）。

![image-20220408153006231](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721789.png)

从这张图可以看出来，name 是有序的，phone 是无序的。当name 相等的时候， phone 才是有序的。

这个时候我们使用where name= '青山' and phone = '136xx '去查询数据的时候， B+Tree 会优先比较 name 来确定下一步应该搜索的方向，往左还是往右。如果 name 相同的时候再比较phone。但是如果查询条件没有 name，就不知道第一步应该查哪个节点，因为建立搜索树的时候name 是第一个比较因子，所以用不到索引。

#### 5.2.1.什么时候用到联合索引

所以，我们在建立联合索引的时候，一定要把最常用的列放在最左边。比如下面的三条语句，能用到联合索引吗？

1） 使用两个字段，可以用到联合索引：

```mysql
EXPLAIN SELECT * FROM user_innodb WHERE name= '权亮' AND phone = '15204661800';
```

![image-20220408153111203](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721107.png)

2） 使用左边的name 字段，可以用到联合索引：

```mysql
EXPLAIN SELECT * FROM user_innodb WHERE name= '权亮'
```

![image-20220408153216280](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721309.png)

3） 使用右边的phone 字段，无法使用索引，全表扫描：

```mysql
EXPLAIN SELECT * FROM user_innodb WHERE phone = '15204661800'
```

![image-20220408153233216](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721919.png)

#### 5.2.2.如何创建联合索引

有一天我们的DBA 找到我，说我们的项目里面有两个查询很慢。

```mysql
SELECT * FROM user_innodb WHERE name= ? AND phone = ?;
SELECT * FROM user_innodb WHERE name= ?;
```

按照我们的想法，一个查询创建一个索引，所以我们针对这两条SQL 创建了两个索引，这种做法觉得正确吗？

```mysql
CREATE INDEX idx_name on user_innodb(name);
CREATE INDEX idx_name_phone on user_innodb(name,phone);
```

当我们创建一个联合索引的时候，按照最左匹配原则，用左边的字段 name 去查询的时候，也能用到索引，所以第一个索引完全没必要。

相当于建立了两个联合索引(name),(name,phone)。

如果我们创建三个字段的索引index(a,b,c)，相当于创建三个索引： 

* index(a)

* index(a,b)
*  index(a,b,c)

用 where b=? 和 where b=? and c=? 和where a=? and c=?是不能使用到索引的。不能不用第一个字段，不能中断。

这里就是MySQL 联合索引的最左匹配原则。

### 5.3.覆盖索引

回表：

非主键索引，我们先通过索引找到主键索引的键值，再通过主键值查出索引里面没有的数据，它比基于主键索引的查询多扫描了一棵索引树，这个过程就叫回表。

例如：select * from user_innodb where name = '青山';

![image-20220408153431970](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721982.png)

在辅助索引里面，不管是单列索引还是联合索引，如果 select 的数据列只用从索引中就能够取得，不必从数据区中读取，这时候使用的索引就叫做覆盖索引，这样就避免了回表。

我们先来创建一个联合索引：

```mysql
-- 创建联合索引
ALTER TABLE user_innodb DROP INDEX comixd_name_phone;
ALTER TABLE user_innodb add INDEX `comixd_name_phone` (`name`,`phone`);
```

这三个查询语句都用到了覆盖索引：

```mysql
EXPLAIN SELECT name,phone FROM user_innodb WHERE name= '青山' AND phone = ' 13666666666'; 
EXPLAIN SELECT name FROM user_innodb WHERE name= ' 青 山 ' AND phone = ' 13666666666'; 
EXPLAIN SELECT phone FROM user_innodb WHERE name= '青山' AND phone = ' 13666666666';
```

Extra 里面值为“Using index”代表使用了覆盖索引。

![image-20220408153519050](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721700.png)

select * ，用不到覆盖索引。

**如果改成只用where phone = 查询呢？动手试试？**

很明显，因为覆盖索引减少了IO 次数，减少了数据的访问量，可以大大地提升查询效率。

### 5.4.索引条件下推（ICP）

https://dev.mysql.com/doc/refman/5.7/en/index-condition-pushdown-optimization.html

再来看这么一张表，在last_name 和first_name 上面创建联合索引。

```mysql
drop table employees;
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NULL,
`first_name` varchar(14) NOT NULL,
 `last_name` varchar(16) NOT NULL,
`gender` enum('M','F') NOT NULL,
`hire_date` date NULL, PRIMARY KEY (`emp_no`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

alter table employees add index idx_lastname_firstname(last_name,first_name);

INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (1, NULL, '698', 'liu', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (2, NULL, 'd99', 'zheng', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (3, NULL, 'e08', 'huang', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (4, NULL, '59d', 'lu', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (5, NULL, '0dc', 'yu', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (6, NULL, '989', 'wang', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (7, NULL, 'e38', 'wang', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (8, NULL, '0zi', 'wang', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (9, NULL, 'dc9', 'xie', 'F', NULL);
INSERT INTO `employees` (`emp_no`, `birth_date`, `first_name`, `last_name`, `gender`, `hire_date`) VALUES (10, NULL, '5ba', 'zhou', 'F', NULL);
```

**关闭ICP：**

```mysql
set optimizer_switch='index_condition_pushdown=off';
```

**查看参数：**

```mysql
show variables like 'optimizer_switch';
```

现在我们要查询所有姓 wang，并且名字最后一个字是 zi 的员工，比如王胖子，王瘦子。查询的SQL

```mysql
select * from employees where last_name='wang' and first_name LIKE '%zi' ;
```

**这条SQL 有两种执行方式：**

1、根据联合索引查出所有姓 wang 的二级索引数据，然后回表，到主键索引上查询全部符合条件的数据（3 条数据）。然后返回给 Server 层，在 Server 层过滤出名字以zi 结尾的员工。

2、根据联合索引查出所有姓wang 的二级索引数据（3 个索引），然后从二级索引中筛选出first_name 以zi 结尾的索引（1 个索引），然后再回表，到主键索引上查询全部符合条件的数据（1 条数据），返回给Server 层。

![image-20220408153803466](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081720320.png)

很明显，第二种方式到主键索引上查询的数据更少。

注意，==索引的比较是在存储引擎进行的，数据记录的比较，是在Server 层进行的==。而当 first_name 的条件不能用于索引过滤时，Server 层不会把 first_name 的条件传递给存储引擎，所以读取了两条没有必要的记录。

这时候，如果满足 last_name='wang'的记录有 100000 条，就会有 99999 条没有必要读取的记录。

**执行以下SQL，Using where：**

```mysql
explain select * from employees where last_name='wang' and first_name LIKE '%zi' ;
```

![image-20220408153850516](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721038.png)

Using Where 代表从存储引擎取回的数据不全部满足条件，需要在Server 层过滤。先用last_name 条件进行索引范围扫描，读取数据表记录，然后进行比较，检查是否符合first_name LIKE '%zi' 的条件。此时 3 条中只有 1 条符合条件。

**开启ICP：**

```mysql
set optimizer_switch='index_condition_pushdown=on';
```

此时的执行计划，Using index condition：

把first_name LIKE '%zi'下推给存储引擎后，只会从数据表读取所需的 1 条记录。

索引条件下推（Index Condition Pushdown），5.6 以后完善的功能。只适用于==二级索引==。ICP 的目标是减少访问表的完整行的读数量从而减少 I/O 操作。

## 6 索引的创建与使用

因为索引对于改善查询性能的作用是巨大的，所以我们的目标是尽量使用索引。

### 6.1 索引的创建

1、在用于where 判断order 排序和join 的（on）字段上创建索引

2、索引的个数不要过多。——浪费空间，更新变慢。

3、区分度低的字段，例如性别，不要建索引。——离散度太低，导致扫描行数过多。

4、频繁更新的值，不要作为主键或者索引。——页分裂

5、组合索引把散列性高（区分度高）的值放在前面。

6、创建复合索引，而不是修改单列索引。

7、过长的字段，怎么建立索引？

8、为什么不建议用无序的值（例如身份证、UUID ）作为索引？

### 6.2.什么时候用不到索引？

1、索引列上使用函数（replace\SUBSTR\CONCAT\sum count avg）、表达式、计算（+ - * /）：

```mysql
explain SELECT * FROM `t2` where id+1 = 4;
```

2、字符串不加引号，出现隐式转换

```MYSQL
ALTER TABLE user_innodb DROP INDEX comidx_name_phone;
ALTER TABLE user_innodb add INDEX comidx_name_phone (name,phone);

explain SELECT * FROM `user_innodb` where name = 136;
explain SELECT * FROM `user_innodb` where name = '136';
```

3、like 条件中前面带%

where 条件中 like abc%，like %2673%，like %888 都用不到索引吗？为什么？

```mysql
explain select *from user_innodb where name like 'wang%'; 
explain select *from user_innodb where name like '%wang';
```

过滤的开销太大，所以无法使用索引。这个时候可以用全文索引。

4、负向查询

NOT LIKE 不能

```mysql
explain select *from employees where last_name not like 'wang'
```

!= （<>）和NOT IN 在某些情况下可以：

```mysql
explain select *from employees where emp_no not in (1) 
explain select *from employees where emp_no <> 1
```

注意一个SQL 语句是否使用索引，跟数据库版本、数据量、数据选择度都有关系。

其实，用不用索引，最终都是优化器说了算。

基于cost 开销（Cost Base Optimizer），它不是基于规则（Rule-Based Optimizer），也不是基于语义。怎么样开销小就怎么来。

[https://docs.oracle.com/cd/B10501_01/server.920/a96533/rbo.htm#38960](https://docs.oracle.com/cd/B10501_01/server.920/a96533/rbo.htm)

https://dev.mysql.com/doc/refman/5.7/en/cost-model.html



# 索引优化

## 1 优化思路

作为架构师或者开发人员，说到数据库性能优化，你的思路是什么样的？

或者具体一点，如果在面试的时候遇到这个问题：你会从哪些维度来优化数据库， 你会怎么回答？

我们在第一节课开始的时候讲了，这四节课的目标是为了让大家建立数据库的知识体系，和正确的调优的思路。

我们说到性能调优，大部分时候想要实现的目标是让我们的查询更快。一个查询的动作又是由很多个环节组成的，每个环节都会消耗时间，我们在第一节课讲SQL 语句的执行流程的时候已经分析过了。

我们要减少查询所消耗的时间，就要从每一个环节入手。

![image-20220408165953817](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081659990.png)

## 2 连接——配置优化

第一个环节是客户端连接到服务端，连接这一块有可能会出现什么样的性能问题？ 有可能是服务端连接数不够导致应用程序获取不到连接。比如报了一个 Mysql: error 1040: Too many connections 的错误。

我们可以从两个方面来解决连接数不够的问题：

1、从服务端来说，我们可以增加服务端的可用连接数。

如果有多个应用或者很多请求同时访问数据库，连接数不够的时候，我们可以：

（1）修改配置参数增加可用连接数，修改max_connections 的大小：

```mysql
show variables like 'max_connections'; -- 修改最大连接数，当有多个应用连接的时候
```

（2）或者，或者及时释放不活动的连接。交互式和非交互式的客户端的默认超时时间都是 28800 秒，8 小时，我们可以把这个值调小。

```mysql
show global variables like 'wait_timeout'; --及时释放不活动的连接，注意不要释放连接池还在使用的连接
```

2、从客户端来说，可以减少从服务端获取的连接数，如果我们想要不是每一次执行SQL 都创建一个新的连接，应该怎么做？

这个时候我们可以引入连接池，实现连接的重用。

我们可以在哪些层面使用连接池？ORM 层面（MyBatis 自带了一个连接池）；或者使用专用的连接池工具（阿里的Druid、Spring Boot 2.x 版本默认的连接池Hikari、老牌的DBCP 和C3P0）。

当客户端改成从连接池获取连接之后，连接池的大小应该怎么设置呢？大家可能会有一个误解，觉得连接池的最大连接数越大越好，这样在高并发的情况下客户端可以获取的连接数更多，不需要排队。

实际情况并不是这样。连接池并不是越大越好，只要维护一定数量大小的连接池， 其他的客户端排队等待获取连接就可以了。有的时候连接池越大，效率反而越低。

Druid 的默认最大连接池大小是 8。Hikari 的默认最大连接池大小是 10。

**为什么默认值都是这么小呢？**

在Hikari 的github 文档中，给出了一个 PostgreSQL 数据库建议的设置连接池大小的公式：

https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing

它的建议是机器核数乘以 2 加 1。也就是说，4 核的机器，连接池维护 9 个连接就够了。这个公式从一定程度上来说对其他数据库也是适用的。这里面还有一个减少连接池大小实现提升并发度和吞吐量的案例。

**为什么有的情况下，减少连接数反而会提升吞吐量呢？为什么建议设置的连接池大小要跟CPU 的核数相关呢？**

每一个连接，服务端都需要创建一个线程去处理它。连接数越多，服务端创建的线程数就会越多。

**问题：CPU 是怎么同时执行远远超过它的核数大小的任务的？时间片。上下文切换。**

而CPU 的核数是有限的，频繁的上下文切换会造成比较大的性能开销。

我们这里说到了从数据库配置的层面去优化数据库。不管是数据库本身的配置，还是安装这个数据库服务的操作系统的配置，对于配置进行优化，最终的目标都是为了更好地发挥硬件本身的性能，包括CPU、内存、磁盘、网络。

在不同的硬件环境下，操作系统和 MySQL 的参数的配置是不同的，没有标准的配置。

默认配置可以满足大部分情况的需求，除非有特殊的需求，在清楚参数的含义的情况下再去修改它。修改配置的工作一般由专业的DBA 完成。

至于硬件本身的选择，比如使用固态硬盘，搭建磁盘阵列，选择特定的CPU 型号这些，更不是我们开发人员关注的重点，这个我们就不做过多的介绍了。

如果想要了解一些特定的参数的含义，官网有一份系统的参数列表可以参考：

https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html

除了合理设置服务端的连接数和客户端的连接池大小之外，我们还有哪些减少客户端跟数据库服务端的连接数的方案呢？

我们可以引入缓存。

## 3 缓存——架构优化

### 3.1 缓存

在应用系统的并发数非常大的情况下，如果没有缓存，会造成两个问题：一方面是会给数据库带来很大的压力。另一方面，从应用的层面来说，操作数据的速度也会受到影响。

我们可以用第三方的缓存服务来解决这个问题，例如Redis。

![image-20220408170919326](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081709443.png)

运行独立的==缓存服务==，属于架构层面的优化。

**为了减少单台数据库服务器的读写压力，在架构层面我们还可以做其他哪些优化措施？**

### 3.2主从复制

如果单台数据库服务满足不了访问需求，那我们可以做数据库的集群方案。

集群的话必然会面临一个问题，就是不同的节点之间数据一致性的问题。如果同时读写多台数据库节点，怎么让所有的节点数据保持一致？

这个时候我们需要用到复制技术（replication），被复制的节点称为master，复制的节点称为slave。slave 本身也可以作为其他节点的数据来源，这个叫做级联复制。

![image-20220408171115086](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721368.png)

主从复制是怎么实现的呢？更新语句会记录binlog，它是一种逻辑日志。

有了这个binlog，从服务器会获取主服务器的 binlog 文件，然后解析里面的 SQL 语句，在从服务器上面执行一遍，保持主从的数据一致。

这里面涉及到三个线程，连接到 master 获取binlog，并且解析 binlog 写入中继日志，这个线程叫做I/O 线程。

Master 节点上有一个log dump 线程，是用来发送binlog 给slave 的。

从库的SQL 线程，是用来读取relay log，把数据写入到数据库的。

![image-20220408171146133](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081721566.png)

做了主从复制的方案之后，我们只把数据写入 master 节点，而读的请求可以分担到slave 节点。我们把这种方案叫做读写分离。

![image-20220408171153168](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081711264.png)

读写分离可以一定程度低减轻数据库服务器的访问压力，但是需要特别注意主从数据一致性的问题。如果我们在master 写入了，马上到slave 查询，而这个时候 slave 的数据还没有同步过来，怎么办？

所以，基于主从复制的原理，我们需要弄明白，主从复制到底慢在哪里？

#### 3.2.1单线程

在早期的MySQL 中，slave 的SQL 线程是单线程。master 可以支持SQL 语句的并行执行，配置了多少的最大连接数就是最多同时多少个SQL 并行执行。

而slave 的 SQL 却只能单线程排队执行，在主库并发量很大的情况下，同步数据肯定会出现延迟。

为什么从库上的SQL Thread 不能并行执行呢？举个例子，主库执行了多条SQL 语句，首先用户发表了一条评论，然后修改了内容，最后把这条评论删除了。这三条语句在从库上的执行顺序肯定是不能颠倒的。

```mysql
insert into user_comments (10000009,'nice');
update user_comments set content ='very good' where id =10000009; delete from user_comments where id =10000009;
```

**怎么解决这个问题呢？怎么减少主从复制的延迟？**

首先我们需要知道，在主从复制的过程中，MySQL 默认是==异步复制==的。也就是说， 对于主节点来说，写入 binlog，事务结束，就返回给客户端了。对于 slave 来说，接收到binlog，就完事儿了，master 不关心slave 的数据有没有写入成功。

![image-20220408171420319](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081714444.png)

如果要减少延迟，是不是可以等待全部从库的事务执行完毕，才返回给客户端呢？ 这样的方式叫做全同步复制。从库写完数据，主库才返会给客户端。

这种方式虽然可以保证在读之前，数据已经同步成功了，但是带来的副作用大家应该能想到，事务执行的时间会变长，它会导致master 节点性能下降。

有没有更好的办法呢？既减少slave 写入的延迟，又不会明显增加 master 返回给客户端的时间？

#### 3.2.2半同步复制

介于异步复制和全同步复制之间，还有一种半同步复制的方式。半同步复制是什么样的呢？

主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到binlog 并写到relay log 中才返回给客户端。master 不会等待很长的时间，但是返回给客户端的时候，数据就即将写入成功了，因为它只剩最后一步了：就是读取relay log，写入从库。

![image-20220408171605640](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081716799.png)

如果我们要在数据库里面用半同步复制，必须安装一个插件，这个是谷歌的一位工程师贡献的。这个插件在mysql 的插件目录下已经有提供：

```bash
cd /usr/lib64/mysql/plugin/
```

主库和从库是不同的插件，安装之后需要启用：

```mysql
-- 主库执行
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
set global rpl_semi_sync_master_enabled=1; show variables like '%semi_sync%';

-- 从库执行
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so'; 
set global rpl_semi_sync_slave_enabled=1;
show global variables like '%semi%';
```

相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，它需要等待一个 slave 写入中继日志，这里多了一个网络交互的过程，所以，半同步复制最好在低延时的网络中使用。

这个是从主库和从库连接的角度，来保证slave 数据的写入。

另一个思路，如果要减少主从同步的延迟，减少SQL 执行造成的等待的时间，那有没有办法在从库上，让多个SQL 语句可以并行执行，而不是排队执行呢？

#### 3.2.3多库并行复制

怎么实现并行复制呢？设想一下，如果 3 条语句是在三个数据库执行，操作各自的数据库，是不是肯定不会产生并发的问题呢？执行的顺序也没有要求。当然是，所以如果是操作三个数据库，这三个数据库的从库的 SQL 线程可以并发执行。这是 MySQL 5.6 版本里面支持的多库并行复制。

![image-20220408172236979](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081722059.png)

但是在大部分的情况下，我们都是单库多表的情况，在一个数据库里面怎么实现并行复制呢？或者说，我们知道，数据库本身就是支持多个事务同时操作的；为什么这些事务在主库上面可以并行执行，却不会出现问题呢？

因为他们本身就是互相不干扰的，比如这些事务是操作不同的表，或者操作不同的行，不存在资源的竞争和数据的干扰。那在主库上并行执行的事务，在从库上肯定也是可以并行执行，是不是？比如在 master 上有三个事务同时分别操作三张表，这三个事务是不是在slave 上面也可以并行执行呢？

#### 3.2.4异步复制之 GTID 复制

https://dev.mysql.com/doc/refman/5.7/en/replication-gtids.html

所以，我们可以把那些在主库上并行执行的事务，分为一个组，并且给他们编号， 这一个组的事务在从库上面也可以并行执行。这个编号，我们把它叫做 GTID（Global Transaction Identifiers），这种主从复制的方式，我们把它叫做基于GTID 的复制。

![image-20220408171849770](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081718884.png)

如果我们要使用GTID 复制，我们可以通过修改配置参数打开它，默认是关闭的：

```mysql
show global variables like 'gtid_mode';
```

无论是优化master 和slave 的连接方式，还是让从库可以并行执行 SQL，都是从数据库的层面去解决主从复制延迟的问题。

除了数据库本身的层面之外，在应用层面，我们也有一些减少主从同步延迟的方法。

我们在做了主从复制之后，如果单个 master 节点或者单张表存储的数据过大的时候，比如一张表有上亿的数据，单表的查询性能还是会下降，我们要进一步对单台数据库节点的数据分型拆分，这个就是分库分表。

### 3.3分库分表

垂直分库，减少并发压力。水平分表，解决存储瓶颈。

垂直分库的做法，把一个数据库按照业务拆分成不同的数据库：

![image-20220408172324364](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081723518.png)

![image-20220408172353982](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081723119.png)

水平分库分表的做法，把单张表的数据按照一定的规则分布到多个数据库。

![image-20220408172449395](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081724100.png)**通过主从或者分库分表可以减少单个数据库节点的访问压力和存储压力，达到提升数据库性能的目的，但是如果master 节点挂了，怎么办？**

所以，高可用（High Available）也是高性能的基础。

### 3.4高可用方案

https://dev.mysql.com/doc/mysql-ha-scalability/en/ha-overview.html

#### 3.4.1主从复制

传统的HAProxy + keepalived 的方案，基于主从复制。

#### 3.4.2NDB Cluster

https://dev.mysql.com/doc/mysql-cluster-excerpt/5.7/en/mysql-cluster-overview.html

基于NDB 集群存储引擎的MySQL Cluster。

![image-20220408172627797](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081726894.png)

#### 3.4.3Galera

https://galeracluster.com/
一种多主同步复制的集群方案。

![image-20220408172843054](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081728199.png)

#### 3.4.4MHA/MMM

https://tech.meituan.com/2017/06/29/database-availability-architecture.html

MMM（Master-Master replication manager for MySQL），一种多主的高可用架构，是一个日本人开发的，像美团这样的公司早期也有大量使用MMM。

MHA（MySQL Master High Available）。

MMM 和 MHA 都是对外提供一个虚拟IP，并且监控主节点和从节点，当主节点发生故障的时候，需要把一个从节点提升为主节点，并且把从节点里面比主节点缺少的数据补上，把VIP 指向新的主节点。

#### 3.4.5MGR

https://dev.mysql.com/doc/refman/5.7/en/group-replication.html 

https://dev.mysql.com/doc/refman/5.7/en/mysql-cluster.html

MySQL 5.7.17 版本推出的 InnoDB Cluster， 也叫 MySQL Group Replicatioin
（MGR），这个套件里面包括了mysql shell 和 mysql-route。

![image-20220408173008110](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081730234.png)

总结一下：

高可用HA 方案需要解决的问题都是当一个master 节点宕机的时候，如何提升一个数据最新的slave 成为 master。如果同时运行多个 master，又必须要解决 master 之间数据复制，以及对于客户端来说连接路由的问题。

不同的方案，实施难度不一样，运维管理的成本也不一样。

以上是架构层面的优化，可以用缓存，主从，分库分表。

第三个环节：

解析器，词法和语法分析，主要保证语句的正确性，语句不出错就没问题。由 Sever 自己处理，跳过。

第四步：优化器

## 4 优化器——SQL 语句分析与优化

优化器就是对我们的SQL 语句进行分析，生成执行计划。

问题：在我们做项目的时候，有时会收到DBA 的邮件，里面列出了我们项目上几个耗时比较长的查询语句，让我们去优化，这些语句是从哪里来的呢？

我们的服务层每天执行了这么多SQL 语句，它怎么知道哪些SQL 语句比较慢呢？ 第一步，我们要把SQL 执行情况记录下来。

### 4.1慢查询日志 slow query log

https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html

#### 4.1.1打开慢日志开关

因为开启慢查询日志是有代价的（跟 bin log、optimizer-trace 一样），所以它默认是关闭的：

```mysql
show variables like 'slow_query%';
```

![image-20220408173204908](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081732007.png)

除了这个开关，还有一个参数，控制执行超过多长时间的SQL 才记录到慢日志，默认是 10 秒。

```mysql
show variables like '%slow_query%';
```

可以直接动态修改参数（重启后失效）。

```mysql
set @@global.slow_query_log=1;	-- 1 开启，0 关闭，重启后失效
set @@global.long_query_time=3;	-- mysql 默认的慢查询时间是 10 秒，另开一个窗口后才会查到最新值

show variables like '%long_query%';
show variables like '%slow_query%';
```

或者修改配置文件my.cnf。

以下配置定义了慢查询日志的开关、慢查询的时间、日志文件的存放路径。

```mysql
slow_query_log = ON long_query_time=2
slow_query_log_file =/var/lib/mysql/localhost-slow.log
```

模拟慢查询：

```mysql
select sleep(10);
```

查询user_innodb 表的 500 万数据（检查是不是没有索引）。

```mysql
SELECT * FROM `user_innodb` where phone = '136';
```

#### 4.1.2慢日志分析

**1、日志内容**

```mysql
show global status like 'slow_queries'; -- 查看有多少慢查询
show variables like '%slow_query%'; -- 获取慢日志目录
cat /var/lib/mysql/ localhost-slow.log
```

有了慢查询日志，怎么去分析统计呢？比如SQL 语句的出现的慢查询次数最多，平均每次执行了多久？

**2、mysqldumpslow**

https://dev.mysql.com/doc/refman/5.7/en/mysqldumpslow.html

MySQL 提供了mysqldumpslow 的工具，在MySQL 的bin 目录下。

```mysql
mysqldumpslow --help
```

例如：查询用时最多的 20 条慢SQL：

```mysql
mysqldumpslow -s t -t 20 -g 'select' /var/lib/mysql/localhost-slow.log
```

![image-20220408173509953](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081735114.png)

Count 代表这个SQL 执行了多少次；

Time 代表执行的时间，括号里面是累计时间； Lock 表示锁定的时间，括号是累计；

Rows 表示返回的记录数，括号是累计。

除了慢查询日志之外，还有一个 SHOW PROFILE 工具可以使用。

### 4.2 SHOW PROFILE

https://dev.mysql.com/doc/refman/5.7/en/show-profile.html

SHOW PROFILE 是谷歌高级架构师Jeremy Cole 贡献给MySQL 社区的，可以查看SQL 语句执行的时候使用的资源，比如 CPU、IO 的消耗情况。

在SQL 中输入help profile 可以得到详细的帮助信息。

#### 4.2.1查看是否开启

```mysql
select @@profiling; 
set @@profiling=1;
```

#### 4.2.2查看 profile 统计

```mysql
# （命令最后带一个s）
show profiles;
```

![image-20220408173620387](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081736535.png)

查看最后一个SQL 的执行详细信息，从中找出耗时较多的环节（没有s）。

```mysql
show profile;
```

![image-20220408173646673](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081736815.png)

6.2E-5，小数点左移 5 位，代表 0.000062 秒。

也可以==根据ID== 查看执行详细信息，在后面带上for query + ID。

```mysql
show profile for query 1;
```

除了慢日志和 show profile，如果要分析出当前数据库中执行的慢的SQL，还可以通过查看运行线程状态和服务器运行信息、存储引擎信息来分析。

#### 4.2.3其他系统命令

##### **show processlist 运行线程**

https://dev.mysql.com/doc/refman/5.7/en/show-processlist.html

```mysql
show processlist;
```

这是很重要的一个命令，用于显示用户运行线程。可以根据id 号kill 线程。也可以查表，效果一样：

```mysql
select * from information_schema.processlist;
```

![image-20220408173818179](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081738305.png)

| 列   | 含义                                           |
| ---- | ---------------------------------------------- |
| Id   | 线程的唯一标志，可以根据它 kill 线程           |
| User | 启动这个线程的用户，普通用户只能看到自己的线程 |
| Host | 哪个 IP 端口发起的连接                         |
| db   | 操作的数据库                                   |
| Command | 线程的命令https://dev.mysql.com/doc/refman/5.7/en/thread-commands.html |
| Time    | 操作持续时间，单位秒                                         |
| State   | 线程状态，比如查询可能有 copying to tmp table，Sorting result，Sending datahttps://dev.mysql.com/doc/refman/5.7/en/general-thread-states.html |
| Info    | SQL 语句的前 100 个字符， 如果要查看完整的 SQL 语句， 用 SHOW FULLPROCESSLIST |

##### show status 服务器运行状态

https://dev.mysql.com/doc/refman/5.7/en/show-status.html

SHOW STATUS 用于查看 MySQL 服务器运行状态（重启后会清空），有 session 和global 两种作用域，格式：参数-值。

可以用like 带通配符过滤。

```mysql
SHOW GLOBAL STATUS LIKE 'com_select'; -- 查看select 次数
```

##### show engine  存储引擎运行信息

https://dev.mysql.com/doc/refman/5.7/en/show-engine.html

show engine 用来显示存储引擎的当前运行信息，包括事务持有的表锁、行锁信息； 事务的锁等待情况；线程信号量等待；文件IO 请求；buffer pool 统计信息。

例如：

```mysql
show engine innodb status;
```

如果需要将监控信息输出到错误信息error log 中（15 秒钟一次），可以开启输出。

```mysql
show variables like 'innodb_status_output%';
-- 开启输出：
SET GLOBAL innodb_status_output=ON;
SET GLOBAL innodb_status_output_locks=ON;
```

我们现在已经知道了这么多分析服务器状态、存储引擎状态、线程运行信息的命令， 如果让你去写一个数据库监控系统，你会怎么做？

其实很多开源的慢查询日志监控工具，他们的原理其实也都是读取的系统的变量和状态。

**现在我们已经知道哪些SQL 慢了，为什么慢呢？慢在哪里？**

MySQL 提供了一个执行计划的工具（在架构中我们有讲到，优化器最终生成的就是一个执行计划），其他数据库，例如Oracle 也有类似的功能。

通过 EXPLAIN 我们可以模拟优化器执行 SQL 查询语句的过程，来知道 MySQL 是怎么处理一条SQL 语句的。通过这种方式我们可以分析语句或者表的性能瓶颈。

**explain 可以分析update、delete、insert 么？**

MySQL 5.6.3 以前只能分析SELECT; MySQL5.6.3 以后就可以分析update、delete、insert 了。

### 4.3EXPLAIN 执行计划

官方链接：https://dev.mysql.com/doc/refman/5.7/en/explain-output.html

我们先创建三张表。一张课程表，一张老师表，一张老师联系方式表（没有任何索引）。

```mysql
DROP TABLE IF EXISTS course;
CREATE TABLE `course` (
`cid` int(3) DEFAULT NULL,
`cname` varchar(20) DEFAULT NULL,
`tid` int(3) DEFAULT NULL
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

DROP TABLE IF EXISTS teacher;
CREATE TABLE `teacher` (
`tid` int(3) DEFAULT NULL,
`tname` varchar(20) DEFAULT NULL,
`tcid` int(3) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

DROP TABLE IF EXISTS teacher_contact; CREATE TABLE `teacher_contact` (
`tcid` int(3) DEFAULT NULL,
`phone` varchar(200) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO `course` VALUES ('1', 'mysql', '1'); INSERT INTO `course` VALUES ('2', 'jvm', '1'); INSERT INTO `course` VALUES ('3', 'juc', '2');
INSERT INTO `course` VALUES ('4', 'spring', '3');

INSERT INTO `teacher` VALUES ('1', 'qingshan', '1'); INSERT INTO `teacher` VALUES ('2', 'jack', '2'); INSERT INTO `teacher` VALUES ('3', 'mic', '3');

INSERT INTO `teacher_contact` VALUES ('1', '13688888888');
INSERT INTO `teacher_contact` VALUES ('2', '18166669999');
INSERT INTO `teacher_contact` VALUES ('3', '17722225555');
```

explain 的结果有很多的字段，我们详细地分析一下。先确认一下环境：

```mysql
select version();
show variables like '%engine%';
```

#### 4.3.1 id

id 是查询序列编号。

##### id 值不同

id 值不同的时候，先查询id 值大的（先大后小）。

```mysql
-- 查询mysql 课程的老师手机号E
EXPLAIN SELECT tc.phone FROM teacher_contact tc WHERE tcid = (
SELECT tcid FROM teacher t WHERE t.tid = ( SELECT c.tid FROM course c
WHERE c.cname = 'mysql'));
```

查询顺序：course c——teacher t——teacher_contact tc。

![](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081744173.png)

先查课程表，再查老师表，最后查老师联系方式表。子查询只能以这种方式进行， 只有拿到内层的结果之后才能进行外层的查询。

##### id 值相同

```mysql
-- 查询课程ID 为 2，或者联系表ID 为 3 的老师
EXPLAIN
SELECT t.tname,c.cname,tc.phone
FROM teacher t, course c, teacher_contact tc WHERE t.tid = c.tid
AND t.tcid = tc.tcid AND (c.cid = 2
OR tc.tcid = 3);
```

![image-20220408174511828](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081745919.png)

id 值相同时，表的查询顺序是从上往下顺序执行。例如这次查询的id 都是 1，查询的顺序是teacher t（3 条）——course c（4 条）——teacher_contact tc（3 条）。

teacher 表插入 3 条数据后：

```mysql
INSERT INTO `teacher` VALUES (4, 'james', 4); INSERT INTO `teacher` VALUES (5, 'tom', 5); INSERT INTO `teacher` VALUES (6, 'seven', 6); COMMIT;

-- （备份）恢复语句
DELETE FROM teacher where tid in (4,5,6); COMMIT;
```

id 也都是 1，但是从上往下查询顺序变成了：teacher_contact tc（3 条）——teacher t（6 条）——course c（4 条)。

![img](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081745897.png)

为什么数据量不同的时候顺序会发生变化呢？这个是由笛卡尔积决定的。

举例：假如有a、b、c 三张表，分别有 2、3、4 条数据，如果做三张表的联合查询， 当查询顺序是 a→b→c 的时候，它的笛卡尔积是：2*3*4=6*4=24。如果查询顺序是 c→b→a，它的笛卡尔积是 4*3*2=12*2=24。

因为MySQL 要把查询的结果，包括中间结果和最终结果都保存到内存，所以 MySQL 会优先选择中间结果数据量比较小的顺序进行查询。所以最终联表查询的顺序是a→b→ c。这个就是为什么teacher 表插入数据以后查询顺序会发生变化

（小标驱动大表的思想）

##### 既有相同也有不同

如果ID 有相同也有不同，就是ID 不同的先大后小，ID 相同的从上往下。

#### 4.3.2select type 查询类型

==这里并没有列举全部==（其它：DEPENDENT UNION、DEPENDENT SUBQUERY、MATERIALIZED、UNCACHEABLE SUBQUERY、UNCACHEABLE UNION）。

下面列举了一些常见的查询类型：

##### SIMPLE

简单查询，不包含子查询，不包含关联查询union。

```mysql
EXPLAIN SELECT * FROM teacher;
```

![image-20220408174745285](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081747390.png)

再看一个包含子查询的案例：

```mysql
-- 查询mysql 课程的老师手机号EXPLAIN SELECT tc.phone FROM teacher_contact tc WHERE tcid = (
SELECT tcid FROM teacher t WHERE t.tid = ( SELECT c.tid FROM course c
WHERE c.cname = 'mysql'));
```

![image-20220408174808086](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081750980.png)

##### PRIMARY

子查询SQL 语句中的==主查询==，也就是最外面的那层查询。

##### SUBQUERY

子查询中所有的==内层查询==都是SUBQUERY 类型的。

##### DERIVED

衍生查询，表示在得到最终查询结果之前会用到临时表。例如：

```mysql
-- 查询ID 为 1 或 2 的老师教授的课程
EXPLAIN SELECT cr.cname FROM (
SELECT * FROM course WHERE tid = 1 UNION
SELECT * FROM course WHERE tid = 2
) cr;
```

![image-20220408174919762](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204081749868.png)

对于关联查询，先执行右边的 table（UNION），再执行左边的 table，类型是DERIVED。

##### UNION

用到了UNION 查询。同上例。

##### UNION RESULT

主要是显示哪些表之间存在 UNION 查询。<union2,3>代表 id=2 和 id=3 的查询存在UNION。同上例。

#### 4.3.3 type 连接类型

[https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-join-types](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)

所有的连接类型中，上面的最好，越往下越差。

**在常用的链接类型中：**==system > const > eq_ref > ref > range > index > all==

**这 里 并 没 有 列 举 全 部** （ 其 他 ： fulltext 、 ref_or_null 、 index_merger 、unique_subquery、index_subquery）。

以上访问类型除了all，都能用到索引。

##### const

> 主键索引或者唯一索引，只能查到一条数据的SQL。

```mysql
DROP TABLE IF EXISTS single_data; CREATE TABLE single_data(
id int(3) PRIMARY KEY,
content varchar(20)
);
insert into single_data values(1,'a');

EXPLAIN SELECT * FROM single_data a where id = 1;
```

![image-20220411103331486](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111033653.png)

##### system

> system 是const 的一种特例，只有一行满足条件。例如：只有一条数据的系统表。

```mysql
EXPLAIN SELECT * FROM mysql.proxies_priv;
```

![image-20220411104043773](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111040888.png)

##### eq_ref

通常出现在多表的 join 查询，表示对于前表的每一个结果,，都只能匹配到后表的一行结果。一般是唯一性索引的查询（UNIQUE 或PRIMARY KEY）。

eq_ref 是除const 之外最好的访问类型。

先删除 teacher 表中多余的数据，teacher_contact 有 3 条数据，teacher 表有 3 条数据。

```mysql
DELETE FROM teacher where tid in (4,5,6); commit;
--  备份
INSERT INTO `teacher` VALUES (4, 'james', 4);
INSERT INTO `teacher` VALUES (5, 'tom', 5);
INSERT INTO `teacher` VALUES (6, 'seven', 6); commit;
```

为teacher_contact 表的tcid（第一个字段）创建==主键索引。==

```mysql
-- ALTER TABLE teacher_contact DROP PRIMARY KEY;
ALTER TABLE teacher_contact ADD PRIMARY KEY(tcid); 
```

为teacher 表的tcid（第三个字段）创建==普通索引==。

```mysql
-- ALTER TABLE teacher DROP INDEX idx_tcid; 
ALTER TABLE teacher ADD INDEX idx_tcid (tcid);
```

执行以下SQL 语句：

```mysql
select t.tcid from teacher t,teacher_contact tc where t.tcid = tc.tcid;
```

![image-20220411104343783](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111043930.png)

此时的执行计划（teacher_contact 表是eq_ref）：

##### 小结：

以上三种 system，const，eq_ref，都是可遇而不可求的，基本上很难优化到这个状态。

##### ref

> 查询用到了非唯一性索引，或者关联操作只使用了索引的最左前缀。

例如：使用tcid 上的普通索引查询：

```mysql
explain SELECT * FROM teacher where tcid = 3;
```

![image-20220411104622444](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111046550.png)

##### range

索引范围扫描。

如果where 后面是 between and 或 <或 > 或 >= 或 <=或in 这些，type 类型就为range。

不走索引一定是全表扫描（ALL），所以先加上==普通索引==。

```mysql
-- ALTER TABLE teacher DROP INDEX idx_tid; 
ALTER TABLE teacher ADD INDEX idx_tid (tid);
```

执行范围查询（字段上有普通索引）：

```mysql
EXPLAIN SELECT * FROM teacher t WHERE t.tid <3;
-- 或
EXPLAIN SELECT * FROM teacher t WHERE tid BETWEEN 1 AND 2;
```

![image-20220411104926493](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111049602.png)

IN 查询也是range（字段有主键索引）

```mysql
EXPLAIN SELECT * FROM teacher_contact t WHERE tcid in (1,2,3);
```

![image-20220411104940774](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111049880.png)

##### index

Full Index Scan，查询全部索引中的数据（比不走索引要快）。

```mysql
EXPLAIN SELECT tid FROM teacher;
```

![image-20220411105056316](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111050438.png)

##### all

Full Table Scan，如果没有索引或者没有用到索引，type 就是ALL。代表全表扫描。

##### NULL

不用访问表或者索引就能得到结果，例如：

```mysql
EXPLAIN select 1 from dual where 1=1;
```

##### 小结：

一般来说，需要保证查询至少达到range 级别，最好能达到ref。ALL（全表扫描）和index（查询全部索引）都是需要优化的。

#### 4.3.4possible_key、key

可能用到的索引和实际用到的索引。如果是NULL 就代表没有用到索引。possible_key 可以有一个或者多个，可能用到索引不代表一定用到索引。反过来，possible_key 为空，key 可能有值吗？

表上创建联合索引：

```mysql
ALTER TABLE user_innodb DROP INDEX comidx_name_phone;
ALTER TABLE user_innodb add INDEX comidx_name_phone (name,phone);
```

执行计划（改成select name 也能用到索引）

```mysql
explain select phone from user_innodb where phone='126';
```

![image-20220411112643735](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111126851.png)

结论：是有可能的（这里是覆盖索引的情况）。

如果通过分析发现没有用到索引，就要检查SQL 或者创建索引。

#### 4.3.5key_len

索引的长度（使用的字节数）。跟索引字段的类型、长度有关。

#### 4.3.6 rows

MySQL 认为扫描多少行才能返回请求的数据，是一个预估值。一般来说行数越少越好。

#### 4.3.7 filtered

这个字段表示存储引擎返回的数据在server 层过滤后，剩下多少满足查询的记录数量的比例，它是一个百分比。

#### 4.3.8 ref

使用哪个列或者常数和索引一起从表中筛选数据。

#### 4.3.9 Extra

执行计划给出的额外的信息说明。

##### using index

用到了覆盖索引，不需要回表。

```mysql
EXPLAIN SELECT tid FROM teacher ;
```

##### using where

使用了where 过滤，表示存储引擎返回的记录并不是所有的都满足查询条件，需要在server 层进行过滤（跟是否使用索引没有关系）。

```mysql
EXPLAIN select * from user_innodb where phone ='13866667777';
```

![image-20220411114641767](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111146898.png)

##### Using index condition（索引条件下推）

https://dev.mysql.com/doc/refman/5.7/en/index-condition-pushdown-optimization.html

##### using filesort

不能使用索引来排序，用到了额外的排序（跟磁盘或文件没有关系）。需要优化。

（复合索引的前提）

```mysql
ALTER TABLE user_innodb DROP INDEX comidx_name_phone;
ALTER TABLE user_innodb add INDEX comidx_name_phone (name,phone);
```

```mysql
EXPLAIN select * from user_innodb where name ='青山' order by id;
```

（order by id 引起）

![image-20220411114922209](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111149350.png)

##### using temporary

用到了临时表。例如（以下不是全部的情况）：

 1、distinct 非索引列

```mysql
EXPLAIN select DISTINCT(tid) from teacher t;
```

2、group by 非索引列

```mysql
EXPLAIN select tname from teacher group by tname;
```

3、使用join 的时候，group 任意列

```mysql
EXPLAIN select t.tid from teacher t join course c on t.tid = c.tid group by t.tid;
```

需要优化，例如创建复合索引。

总结一下：

模拟优化器执行SQL 查询语句的过程，来知道 MySQL 是怎么处理一条SQL 语句的。通过这种方式我们可以分析语句或者表的性能瓶颈。

分析出问题之后，就是对SQL 语句的具体优化。

比如怎么用到索引，怎么减少锁的阻塞等待

### 4.4 SQL 与索引优化

当我们的SQL 语句比较复杂，有多个关联和子查询的时候，就要分析 SQL 语句有没有改写的方法。

举个简单的例子，一模一样的数据：

```mysql
-- 大偏移量的limit
select * from user_innodb limit 900000,10;

-- 改成先过滤ID，再limit
SELECT * FROM user_innodb WHERE id >= 900000 LIMIT 10;
```

对于具体的 SQL 语句的优化，MySQL 官网也提供了很多建议，这个是我们在分析具体的SQL 语句的时候需要注意的

https://dev.mysql.com/doc/refman/5.7/en/optimization.html

## 5 存储引擎

### 5.1 存储引擎的选择

为不同的业务表选择不同的存储引擎，例如：查询插入操作多的业务表，用 MyISAM。临时数据用Memeroy。常规的并发大更新多的表用InnoDB。

### 5.2 分区或者分表

分区不推荐。

交易历史表：在年底为下一年度建立 12 个分区，每个月一个分区。渠道交易表：分成当日表；当月表；历史表，历史表再做分区。

### 5.3 字段定义

原则：使用可以正确存储数据的最小数据类型。为每一列选择合适的字段类型：

#### 5.3.1整数类型

![image-20220411120043522](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111200649.png)

INT 有 8 种类型，不同的类型的最大存储范围是不一样的。性别？用TINYINT，因为 ENUM 也是整型存储。

#### 5.3.2 字符类型

变长情况下，varchar 更节省空间，但是对于 varchar 字段，需要一个字节来记录长度。

固定长度的用char，不要用varchar。

#### 5.3.3 非空 

非空字段尽量定义成NOT NULL，提供默认值，或者使用特殊值、空串代替 null。NULL 类型的存储、优化、使用都会存在问题。

#### 5.3.4 不要用外键、触发器、视图

降低了可读性；

影响数据库性能，应该把把计算的事情交给程序，数据库专心做存储； 数据的完整性应该在程序中检查。

#### 5.3.5 大文件存储

不要用数据库存储图片（比如base64 编码）或者大文件；

把文件放在NAS 上，数据库只需要存储 URI（相对路径），在应用中配置 NAS 服务器地址。

#### 5.3.6 表拆分

将不常用的字段拆分出去，避免列数过多和数据量过大。

比如在业务系统中，要记录所有接收和发送的消息，这个消息是 XML 格式的，用blob 或者text 存储，用来追踪和判断重复，可以建立一张表专门用来存储报文。

## 6 总结：优化体系

![image-20220411133016336](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202204111330520.png)

除了对于代码、SQL 语句、表定义、架构、配置优化之外，业务层面的优化也不能忽视。举几个例子：

**1） 在某一年的双十一，为什么会做一个充值到余额宝和余额有奖金的活动（充 300 送 50）？**

因为使用余额或者余额宝付款是记录本地或者内部数据库，而使用银行卡付款，需要调用接口，操作内部数据库肯定更快。

**2） 在去年的双十一，为什么在凌晨禁止查询今天之外的账单？**

账单？ 这是一种降级措施，用来保证当前最核心的业务。

**3） 最近几年的双十一，为什么提前一个多星期就已经有双十一当天的价格了？ 预售分流。**

在应用层面同样有很多其他的方案来优化，达到尽量减轻数据库的压力的目的，比如限流，或者引入MQ 削峰，等等等等。

为什么同样用MySQL，有的公司可以扛住百万千万级别的并发，而有的公司几百个并发都扛不住，关键在于怎么用。所以，用数据库慢，不代表数据库本身慢，有的时候还要往上层去优化。

当然，如果关系型数据库解决不了的问题，我们可能需要用到搜索引擎或者大数据的方案了，并不是所有的数据都要放到关系型数据库存储。
