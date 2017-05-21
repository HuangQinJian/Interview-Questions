[toc]

#**索引的实现方式**

1、B+树
    我们经常听到B+树就是这个概念，用这个树的目的和红黑树差不多，也是为了尽量保持树的平衡，当然红黑树是二叉树，但B+树就不是二叉树了，节点下面可以有多个子节点，数据库开发商会设置子节点数的一个最大值，这个值不会太小，所以B+树一般来说比较矮胖，而红黑树就比较瘦高了。
关于B+树的插入，删除，会涉及到一些算法以保持树的平衡，这里就不详述了。ORACLE的默认索引就是这种结构的。
如果经常需要同时对两个字段进行AND查询,那么使用两个单独索引不如建立一个复合索引，因为两个单独索引通常数据库只能使用其中一个，而使用复合索引因为索引本身就对应到两个字段上的，效率会有很大提高。

2、散列索引
    第二种索引叫做散列索引，就是通过散列函数来定位的一种索引，不过很少有单独使用散列索引的，反而是散列文件组织用的比较多。
散列文件组织就是根据一个键通过散列计算把对应的记录都放到同一个槽中，这样的话相同的键值对应的记录就一定是放在同一个文件里了，也就减少了文件读取的次数，提高了效率。
散列索引呢就是根据对应键的散列码来找到最终的索引项的技术，其实和B树就差不多了，也就是一种索引之上的二级辅助索引，我理解散列索引都是二级或更高级的稀疏索引，否则桶就太多了，效率也不会很高。

3、位图索引
    位图索引是一种针对多个字段的简单查询设计一种特殊的索引，适用范围比较小，只适用于字段值固定并且值的种类很少的情况，比如性别，只能有男和女，或者级别，状态等等，并且只有在同时对多个这样的字段查询时才能体现出位图的优势。
位图的基本思想就是对每一个条件都用0或者1来表示，如有5条记录，性别分别是男，女，男，男，女，那么如果使用位图索引就会建立两个位图，对应男的10110和对应女的01001,这样做有什么好处呢，就是如果同时对多个这种类型的字段进行and或or查询时，可以使用按位与和按位或来直接得到结果了。

为了进一步榨取MySQL的效率，就要考虑建立组合索引。就是将 name, city, age建到一个索引里：

```
ALTER TABLE mytable ADD INDEX name_city_age (name(10),city,age); 
```

建表时，usernname长度为 16，这里用 10。这是因为一般情况下名字的长度不会超过10，这样会加速索引查询速度，还会减少索引文件的大小，提高INSERT的更新速度。

如果分别在 usernname，city，age上建立单列索引，让该表有3个单列索引，查询时和上述的组合索引效率也会大不一样，远远低于我们的组合索引。虽然此时有了三个索引，但MySQL只能用到其中的那个它认为似乎是最有效率的单列索引。

建立这样的组合索引，其实是相当于分别建立了下面三组组合MySQL数据库索引：

```
CREATE TABLE mytable( ID INT NOT NULL, username VARCHAR(16) NOT NULL, city VARCHAR(50) NOT NULL, age INT NOT NULL );
```

usernname,city,age usernname,city usernname 为什么没有 city，age这样的组合索引呢？这是因为MySQL组合索引“最左前缀”的结果。简单的理解就是只从最左面的开始组合。并不是只要包含这三列的查询都会用到该组合索引，下面的几个SQL就会用到这个组合MySQL数据库索引：

```
SELECT * FROM mytable WHREE username="admin" AND city="郑州" SELECT * FROM mytable WHREE username="admin"
```

 而下面几个则不会用到：

```
SELECT * FROM mytable WHREE age=20 AND city="郑州" SELECT * FROM mytable WHREE city="郑州"
```

使用索引的代价

1、索引需要占用数据表以外的物理存储空间
2、创建索引和维护索引要花费一定的时间
3、当对表进行更新操作时，索引需要被重建，这样降低了数据的维护速度


什么情况下不适合建立索引？

1.对于在查询过程中很少使用或参考的列，不应该创建索引。
2.对于那些只有很少数据值的列，不应该创建索引。
3.对于那些定义为image，text和bit数据类型的列，不应该创建索引。
4.当修改性能远大于检索性能，不应该建立索引。


----------

#**线程池中的corenum和maxnum有什么不同？**

(1) 直接提交的队列：该功能由SynchronizedQueue对象提供。SynchronizedQueue是一个特殊的阻塞队列。SynchronizedQueue没有容量，每一个插入操作都要等待一个相应的删除操作，反之每一个删除操作都需要等待对应的插入操作。使用SynchronizedQueue时提交的任务不会被真实的保存，而总是将新任务提交给线程执行，如果没有空闲的线程则尝试创建新的线程，如果线程数量达到最大值就执行决绝策略。使用SynchronizedQueue队列通常要设置很大的maxnumPoolSize，否则很容易执行拒绝策略。可以当做大小为0的队列来理解。

(2) 有界的任务队列：有界的任务队列可以使用ArrayBlockingQueue实现。当使用有界的任务队列时，若有新的任务需要执行，如果线程池的实际线程数小于核心线程数，则有优先创建新的线程，若大于核心线程数，则会将新任务加入等待队列。若队列已满，无法加入则在总线程数不大于最大线程数的前提下，创建新的线程。若大于最大线程数，则执行拒绝策略。也就是说，有界队列仅当任务队列满时才可能将线程数提升到核心线程数只上，否则确保线程数维持在核心线程数大小。

(3) 无界任务队列：无界任务队列可以通过LinkedBlockingQueue类来实现。与有界队列相比，除非系统资源耗尽，否则无界队列不存在任务入队失败的情况。当有新的任务到来，系统的线程数小于核心线程数时线程池会生成新的线程执行任务，但当系统线程数大于核心线程数后，就不会继续增加。若后续有新的任务，则将任务放入无界队列中等待。

(4) 优先任务队列：优先任务队列是带有执行优先级的队列，通过PriorityBlockingQueue实现，可以控制任务的执行先后顺序，是一个特殊的无界队列。无论是有界队列还ArrayBlockingQueue还是未指定大小的无界队列LinkedBlockingQueue，都是按照先进先出算法处理任务的，而有限队列则可以根据任务自身的优先级顺序执行，在确保系统性能的同时，也能有很好的质量保证。

回过头看Executor框架提供了几种线程池，newFixedThreadPool()返回的固定大小线程池中核心线程数和最大线程数一样，并且使用了无界队列。因为对于固定大小的线程池来说，不存在线程数量的动态变化，所以最大线程数等于核心线程数。同时，使用无界队列存放无法立即执行的任务，当任务提交非常频繁时，队列可能迅速膨胀，从而耗尽系统资源。 

newSingleThreadExecutor()返回的单线程线程池，是固定大小线程池的一种退化，只是简单的将线程池数量设置为1。 
newCachedThreadExecutor()返回核心线程数为0，最大线程数为无穷大的线程池。使用直接提交队列SynchronizedQueue。 
当Executor提供的线程池不满足使用场景时，则需要使用自定义线程池，选择合适的任务队列来作为缓冲。不同的并发队列对系统和性能的影响均不同。


----------
#**如何找出单链表中的倒数第k个元素？**

思路一：
初看题目，最容易想到的方法就是遍历。首先遍历一遍单链表，得出整个链表的长度n（元素个数从1到n），然后找到倒数第k个元素的位置n-k+1，接着从头遍历到第n-k+1元素，就是倒数第k个元素。但是该方法需要对链表进行两次遍历，遍历的元素个数为n+n-k+1=2n+1-k个。

思路二：
有了思路一的提示，是不是可以想到用两个指针，让它们之间的距离保持为k-1，同时对链表进行遍历，当第一个指针到达链表的最后一个元素（即倒数第一个元素时），第二个指针刚好停留在倒数第k个元素上。此方法看似对链表进行了一次遍历，其实是用两个指针对链表进行了同时遍历，对链表本身而言，它被遍历的元素个数仍是n+n-k+1=2n+1-k个。

思路三：
思路一和思路二是两种不同思路，但就本质而言，都是两次对链表进行2次遍历，一次遍历n个元素，另一次遍历n-k+1个，总共遍历2n+1-k个元素。此时，想想能否再减少遍历的元素个数而找到倒数第k个元素呢？我们注意到思路二，是用两个指针，保持k-1个元素的距离同时进行遍历的，可否按着每次k个元素这样的遍历下去呢。这样遍历的结果就是，每次遍历k个元素，遍历m次（m=n/k），最后一次遍历的个数为i个（i=n%k），我们只需记录最后一次遍历k个元素的起始位置，然后再遍历i个元素，此时的位置即为倒数第k个元素。此时，对链表遍历的元素个数为n+i（i为n除以k的余数）。


----------

#**多线程缺点？**

1、将给定的工作量划分给过多的线程会造成每个线程的工作量过少，因此可能导致线程启动和终止时的开销比程序实际工作的开销还要多；

2、过多并发线程的存在将导致共享有限硬件资源的开销增大。

线程相对于进程的优点：
1、开销小
2、资源共享性好。

线程相对于进程的缺点：
1、共享资源需要耗费一定的锁资源，同步相对复杂。
2、一个线程崩溃可能导致整个进程崩溃，这个当然是自己的应用程序有问题


----------

#**迭代和递归的最大区别是？**

递归与迭代都是基于控制结构：迭代用重复结构，而递归用选择结构。

递归与迭代都涉及重复：迭代显式使用重复结构，而递归通过重复函数调用实现重复。

递归与迭代都涉及终止测试：迭代在循环条件失败时终止，递归在遇到基本情况时终止。

使用计数器控制重复的迭代和递归都逐渐到达终止点：迭代一直修改计数器，直到计数器值使循环条件失败；递归不断产生最初问题的简化副本，直到达到基本情况。迭代和递归过程都可以无限进行：如果循环条件测试永远不变成false，则迭代发生无限循环；如果递归永远无法回推到基本情况，则发生无穷递归。
递归函数是通过调用函数自身来完成任务，而且在每次调用自身时减少任务量。而迭代是循环的一种形式，这种循环不是由用户输入而控制，每次迭代步骤都必须将剩余的任务减少；也就是说，循环的每一步都必须执行一个有限的过程，并留下较少的步骤。


----------
#**SQL truncate 、delete与drop区别？**

相同点：

1.truncate和不带where子句的delete、以及drop都会删除表内的数据。

2.drop、truncate都是DDL语句(数据定义语言),执行后会自动提交。


不同点：

1. truncate 和 delete 只删除数据不删除表的结构(定义)
drop 语句将删除表的结构被依赖的约束(constrain)、触发器(trigger)、索引(index)；依赖于该表的存储过程/函数将保留,但是变为 invalid 状态。


2. delete 语句是数据库操作语言(dml)，这个操作会放到 rollback segement 中，事务提交之后才生效；如果有相应的 trigger，执行的时候将被触发。

truncate、drop 是数据库定义语言(ddl)，操作立即生效，原数据不放到 rollback segment 中，不能回滚，操作不触发 trigger。

3.delete 语句不影响表所占用的 extent，高水线(high watermark)保持原位置不动
drop 语句将表所占用的空间全部释放。
truncate 语句缺省情况下见空间释放到 minextents个 extent，除非使用reuse storage；truncate 会将高水线复位(回到最开始)。

4.速度，一般来说: drop> truncate > delete

5.安全性：小心使用 drop 和 truncate，尤其没有备份的时候.否则哭都来不及
使用上,想删除部分数据行用 delete，注意带上where子句. 回滚段要足够大.
想删除表,当然用 drop；
想保留表而将所有数据删除，如果和事务无关，用truncate即可。如果和事务有关,或者想触发trigger,还是用delete。
如果是整理表内部的碎片，可以用truncate跟上reuse stroage，再重新导入/插入数据。

6.delete是DML语句,不会自动提交。drop/truncate都是DDL语句,执行后会自动提交。

7、TRUNCATE   TABLE   在功能上与不带   WHERE   子句的   DELETE   语句相同：二者均删除表中的全部行。但   TRUNCATE   TABLE   比   DELETE   速度快，且使用的系统和事务日志资源少。DELETE   语句每次删除一行，并在事务日志中为所删除的每行记录一项。TRUNCATE   TABLE   通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。 

 
8、TRUNCATE   TABLE   删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用   DELETE。如果要删除表定义及其数据，请使用   DROP   TABLE   语句。  
    
9、对于由   FOREIGN   KEY   约束引用的表，不能使用   TRUNCATE   TABLE，而应使用不带   WHERE   子句的   DELETE   语句。由于   TRUNCATE   TABLE   不记录在日志中，所以它不能激活触发器。    
 
10、TRUNCATE   TABLE   不能用于参与了索引视图的表。 


----------
#**总结常见的mysql数据库优化操作？**

1、Index索引

2、少用SELECT *

可能有的人查询数据库时，遇到要查询的都会select，这是不恰当的行为。我们应该取我们要用的数据，而不是全取，因为当我们select时，会增加web服务器的负担，增加网络传输的负载，查询速度自然就下降 。

 3、开启查询缓存

大多数的MySQL服务器都开启了查询缓存。这是提高性最有效的方法之一，而且这是被MySQL的数据库引擎处理的。当有很多相同的查询被执行了多次的时候，这些查询结果会被放到一个缓存中，这样，后续的相同的查询就不用操作表而直接访问缓存结果了。

4、使用NOT NULL

　　很多表都包含可为NULL（空值）的列，即使应用程序并不需要保存 NULL 也是如此 ，这是因为可为NULL是列的默认属性。通常情况下最好指定列为 NOT NULL，除非真的需要存储NULL值。如果查询中包含可为NULL的列，对 MySQL 来说更难优化 ，因为可为 NULL 的列使 得索引、索引统计和值比较都更复杂 。可为NULL 的列会使用更多的存储空间 ，在MySQL里也需要特殊处理 。当可为NULL 的列被索引肘，每个索引记录需要一个额 外的字节，在 MyISAM 里甚至还可能导致固定大小 的索引 （例如只有一个整数列的 索引） 变成可变大小的索引。

　　通常把可为 NULL 的列改为 NOT NULL 带来的性能提升比较小 ，所以 （调优时） 没有 必要首先在现有schema中查找井修改掉这种情况 ，除非确定这会导致问题。但是， 如果计划在列上建索引 ，就应该尽量避免设计成可为 NULL 的列。当然也有例外 ，例如值得一提的是，InnoDB 使用单独的位 （bit ） 存储 NULL 值 ，所 以对于稀疏数据由有很好的空间效率 。但这一点不适用于MyISAM 。

5、避免在 where 子句中使用 or 来连接

如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描，如：

```
select id from t where num=10 or Name = 'admin'
```

可以这样查询：

```
select id from t where num = 10
union all
select id from t where Name = 'admin'
```

6、多使用varchar/nvarchar

使用varchar/nvarchar代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

7、避免大数据量返回

这里要考虑使用limit，来限制返回的数据量，如果每次返回大量自己不需要的数据，也会降低查询速度。

8、where子句优化

where 子句中使用参数，会导致全表扫描,因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。
应尽量避免在 where 子句中对字段进行表达式操作，避免在where子句中对字段进行函数操作这将导致引擎放弃使用索引而进行全表扫描。不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。


----------

#**SQL语句中executeQuery、executeUpdate、execute的区别？**

1. ResultSet executeQuery(String sql); 执行SQL查询，并返回ResultSet 对象。
2. int executeUpdate(String sql); 可执行增，删，改，返回执行受到影响的行数。
3. boolean execute(String sql); 可执行任何SQL语句，返回一个布尔值，表示是否返回ResultSet 。

execute是executeQuery和executeUpdate的综合.

executeUpdate() 这是 PreparedStatement 接口中的方法
executeUpdate(String sql) 这是 PreparedStatement 从父接口 Statement 中继承过来的方法

executeUpdate() 中执行 SQL 语句需要在创建 PerparedStatement 时通过 Connection 的 prepareStatement(String sql) 方法中写出，因为 PerparedStatement 中的 SQL 语句数据库需要进行预编译和缓存，因此要在创建 PerparedStatement 对象时给出 SQL 语句。

而 executeUpdate(String sql) 是 Statement 中的方法，参数中的 SQL 语句只是提交给数据库去执行，并不需要预编译。

<font color=red>**如果 SQL 语句中有 ? 占位符，那么在设置好占位符中的值后，必须使用 executeUpdate() 执行。而 executeUpdate(String sql) 只是提交一个 SQL 语句，且这个语句中不能带有 ? 占位符。**


1、在Java中如何使用execute()、executeQuery()、executeUpdate()三个方法？
execute(String sql) 
          执行给定的 SQL 语句，该语句可能返回多个结果。
executeQuery(String sql) 
          执行给定的 SQL 语句，该语句返回单个 ResultSet 对象
executeUpdate(String sql) 
          执行给定 SQL 语句，该语句可能为 INSERT、UPDATE 或 DELETE 语句，或者不返回任何内容的 SQL 语句（如 SQL DDL 语句）

头2种一般在查询中使用

最后一个在插入、更新、删除时使用

2、executeQuery()是干什么用的?实现什么功能啊？
使用JDBC连接数据库需要四步，第一步加载驱动程序；第二步，连接数据库；第三步，访问数据库；第四步，执行查询；其中在第四步执行查询时，要用statement类的executeQuery()方法来下达select指令以查询数据库，executeQuery()方法会把数据库响应的查询结果存放在ResultSet类对象中供我们使用。即语句：String sql="select * from"+tableName; ResultSet rs=s.executeQuery(sql);

3、executeQuery、executeUpdate或execute方法区别？
在用纯JSP做一个页面报警功能的时候习惯性的用executeQuery来执行SQL语句，结果执行update时就遇到问题，语句能执行，但返回结果出现问题，另外还忽略了executeUpdate的返回值不是结果集ResultSet,而是数值！特收藏如下一篇文章（感谢网友们对各种信息的贡献）： 

JDBCTM中Statement接口提供的execute、executeQuery和executeUpdate之间的区别 


Statement 接口提供了三种执行 SQL 语句的方法：executeQuery、executeUpdate 和 execute。使用哪一个方法由 SQL 语句所产生的内容决定。 

方法executeQuery 
用于产生单个结果集的语句，例如 SELECT 语句。 被使用最多的执行 SQL 语句的方法是 executeQuery。这个方法被用来执行 SELECT 语句，它几乎是使用最多的 SQL 语句。 

方法executeUpdate 
用于执行 INSERT、UPDATE 或 DELETE 语句以及 SQL DDL（数据定义语言）语句，例如 CREATE TABLE 和 DROP TABLE。INSERT、UPDATE 或 DELETE 语句的效果是修改表中零行或多行中的一列或多列。executeUpdate 的返回值是一个整数，指示受影响的行数（即更新计数）。对于 CREATE TABLE 或 DROP TABLE 等不操作行的语句，executeUpdate 的返回值总为零。 

使用executeUpdate方法是因为在 createTableCoffees 中的 SQL 语句是 DDL （数据定义语言）语句。创建表，改变表，删除表都是 DDL 语句的例子，要用 executeUpdate 方法来执行。你也可以从它的名字里看出，方法 executeUpdate 也被用于执行更新表 SQL 语句。实际上，相对于创建表来说，executeUpdate 用于更新表的时间更多，因为表只需要创建一次，但经常被更新。 


方法execute： 
用于执行返回多个结果集、多个更新计数或二者组合的语句。因为多数程序员不会需要该高级功能 

execute方法应该仅在语句能返回多个ResultSet对象、多个更新计数或ResultSet对象与更新计数的组合时使用。当执行某个已存储过程 或动态执行未知 SQL 字符串（即应用程序程序员在编译时未知）时，有可能出现多个结果的情况，尽管这种情况很少见。 
因为方法 execute 处理非常规情况，所以获取其结果需要一些特殊处理并不足为怪。例如，假定已知某个过程返回两个结果集，则在使用方法 execute 执行该过程后，必须调用方法 getResultSet 获得第一个结果集，然后调用适当的 getXXX 方法获取其中的值。要获得第二个结果集，需要先调用 getMoreResults 方法，然后再调用 getResultSet 方法。如果已知某个过程返回两个更新计数，则首先调用方法 getUpdateCount，然后调用 getMoreResults，并再次调用 getUpdateCount。 
对于不知道返回内容，则情况更为复杂。如果结果是 ResultSet 对象，则方法 execute 返回 true；如果结果是 Java int，则返回 false。如果返回 int，则意味着结果是更新计数或执行的语句是 DDL 命令。在调用方法 execute 之后要做的第一件事情是调用 getResultSet 或 getUpdateCount。调用方法 getResultSet 可以获得两个或多个 ResultSet 对象中第一个对象；或调用方法 getUpdateCount 可以获得两个或多个更新计数中第一个更新计数的内容。 
当 SQL 语句的结果不是结果集时，则方法 getResultSet 将返回 null。这可能意味着结果是一个更新计数或没有其它结果。在这种情况下，判断 null 真正含义的唯一方法是调用方法 getUpdateCount，它将返回一个整数。这个整数为调用语句所影响的行数；如果为 -1 则表示结果是结果集或没有结果。如果方法 getResultSet 已返回 null（表示结果不是 ResultSet 对象），则返回值 -1 表示没有其它结果。也就是说，当下列条件为真时表示没有结果（或没有其它结果）： 

((stmt.getResultSet() == null) && (stmt.getUpdateCount() == -1)) 

如果已经调用方法 getResultSet 并处理了它返回的 ResultSet 对象，则有必要调用方法 getMoreResults 以确定是否有其它结果集或更新计数。如果 getMoreResults 返回 true，则需要再次调用 getResultSet 来检索下一个结果集。如上所述，如果 getResultSet 返回 null，则需要调用 getUpdateCount 来检查 null 是表示结果为更新计数还是表示没有其它结果。 

当 getMoreResults 返回 false 时，它表示该 SQL 语句返回一个更新计数或没有其它结果。因此需要调用方法 getUpdateCount 来检查它是哪一种情况。在这种情况下，当下列条件为真时表示没有其它结果： 

((stmt.getMoreResults() == false) && (stmt.getUpdateCount() == -1)) 

下面的代码演示了一种方法用来确认已访问调用方法 execute 所产生的全部结果集和更新计数： 

stmt.execute(queryStringWithUnknownResults); 
while (true) { 
int rowCount = stmt.getUpdateCount(); 
if (rowCount > 0) { // 它是更新计数 
System.out.println("Rows changed = " + count); 
stmt.getMoreResults();


----------

#**Spring初始化过程？**

　　在传统的Java应用中，Bean的生命周期非常简单。Java的关键词new用来实例化Bean（或许他是非序列化的）。这样就够用了。相反，Bean 的生命周期在spring容器中更加细致。理解Spring Bean的生命周期非常重要，因为你或许要利用Spring提供的机会来订制Bean的创建过程。

1.容器寻找Bean的定义信息并且将其实例化。 

2.使用依赖注入，Spring按照Bean定义信息配置Bean的所有属性。
 
3.如果Bean实现了BeanNameAware接口，工厂调用Bean的setBeanName()方法传递Bean的ID。 

4.如果Bean实现了BeanFactoryAware接口，工厂调用setBeanFactory()方法传入工厂自身。 

5.如果BeanPostProcessor和Bean关联，那么它们的postProcessBeforeInitialzation()方法将被调用。 

6.如果Bean指定了init-method方法，它将被调用。 

7.最后，如果有BeanPsotProcessor和Bean关联，那么它们的postProcessAfterInitialization()方法将被调用。 

　　到这个时候，Bean已经可以被应用系统使用了，并且将被保留在Bean Factory中知道它不再需要。有两种方法可以把它从Bean Factory中删除掉。 

1.如果Bean实现了DisposableBean接口，destory()方法被调用。 

2.如果指定了订制的销毁方法，就调用这个方法。
 
　　Bean在Spring应用上下文的生命周期与在Bean工厂中的生命周期只有一点不同，唯一不同的是，如果Bean实现了ApplicationContextAwre接口，setApplicationContext()方法被调用。 

----------
#**MySQL Hash索引和B-Tree索引的区别？**

　　MySQL Hash索引结构的特殊性，其检索效率非常高，索引的检索可以一次定位，不像B-Tree 索引需要从根节点到枝节点，最后才能访问到页节点这样多次的IO访问，所以 Hash 索引的查询效率要远高于 B-Tree 索引。
　　可能很多人又有疑问了，既然 Hash 索引的效率要比 B-Tree 高很多，为什么大家不都用 Hash 索引而还要使用 B-Tree 索引呢？任何事物都是有两面性的，Hash 索引也一样，虽然 Hash 索引效率高，但是 Hash 索引本身由于其特殊性也带来了很多限制和弊端，主要有以下这些。
（1）MySQL Hash索引仅仅能满足"=","IN"和"< >"查询，不能使用范围查询。
由于 MySQL Hash索引比较的是进行 Hash 运算之后的 Hash 值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 Hash 算法处理之后的 Hash 值的大小关系，并不能保证和Hash运算前完全一样。
（2）MySQL Hash索引无法被用来避免数据的排序操作。
由于 MySQL Hash索引中存放的是经过 Hash 计算之后的 Hash 值，而且Hash值的大小关系并不一定和 Hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算；
（3）MySQL Hash索引不能利用部分索引键查询。
对于组合索引，Hash 索引在计算 Hash 值的时候是组合索引键合并后再一起计算 Hash 值，而不是单独计算 Hash 值，所以通过组合索引的前面一个或几个索引键进行查询的时候，Hash 索引也无法被利用。
（4）MySQL Hash索引在任何时候都不能避免表扫描。
前面已经知道，Hash 索引是将索引键通过 Hash 运算之后，将 Hash运算结果的 Hash 值和所对应的行指针信息存放于一个 Hash 表中，由于不同索引键存在相同 Hash 值，所以即使取满足某个 Hash 键值的数据的记录条数，也无法从 Hash 索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果。
（5）MySQL Hash索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高。
对于选择性比较低的索引键，如果创建 Hash 索引，那么将会存在大量记录指针信息存于同一个 Hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下。 