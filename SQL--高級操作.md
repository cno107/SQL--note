# Create

### 多数据插入

基本语法：insert into 表名   [(字段列表)]   values(值列表), (值列表)…;

```sql
insert into stu values('1','phb'),('2','sds'),('3','???');
```

### Primary Key冲突

在有的表中，使用的是业务主键（字段有业务含义），但是往往在进行数据插入的时候，又不确定数据表中是否已经存在对应的主键。

+ 解决方案

  - 主键冲突更新：

    类似插入数据语法，如果插入的过程中主键冲突，那么采用更新方法。

    ```sql
    Insert into 表名 [(字段列表)] values(值列表) on duplicate key update 字段 = 新值;
    ```

  - 主键冲突替换：

    当主键冲突之后，干掉原来的数据，重新插入进去。

    ```sql
    Replace into [(字段列表)] values(值列表);
    ```

### 蠕虫复制

蠕虫复制：一分为二，成倍的增加。从已有的数据中获取数据，并且将获取到的数据插入到数据表中。

+ 基本语法： Insert into 表名 [(字段列表)] select */字段列表 from 表;

  ```sql
  insert into stuCopy(id,name) select id,name from stuCopy;
  ```

+ 注意：

  - 蠕虫复制的确通常是重复数据，没有太大业务意义：可以在短期内快速增加表的数据量，从而可以测试表的压力，还可以通过大量数据来测试表的效率（索引）

  - 蠕虫复制虽好，但是要注意主键冲突。



# Update

+ 在更新数据的时候，特别要注意：通常一定是跟随条件更新

  **Update 表名 set 字段名 = 新值 where 判断条件;**

+ 如果没有条件，是全表更新数据。但是可以使用limit 来限制更新的数量；

  **Update 表名 set 字段名 = 新值 [where 判断条件] limit 数量;**



# Delete

+ **delete from 表名 where [条件];**

+  删除数据的时候尽量不要全部删除，应该使用where进行 判定；

+ 删除数据的时候可以使用limit来限制要删除的具体数量

+ Delete删除数据的时候无法重置auto_increment

+ Mysql有一个能够重置表选项中的自增长的语法;

  **Truncate 表名;** 







# Select

完整的查询指令：

 **<font color='red'>Select select选项 字段列表 from 数据源 where条件 group by分组 having条件 order by排序 limit限制;</font>**

### Select选项

Select选项：系统该如何对待查询得到的结果

+ All：默认的，表示保存所有的记录

+ Distinct：去重，去除重复的记录，只保留一条（所有的字段都相同）

+ 字段列表：有的时候需要从多张表获取数据，在获取数据的时候，可能存在不同表中有同名的字段，需要将同名的字段命名成不同名的：别名 alias

   **基本语法：字段名 [as] 别名 **

  ```sql
  select distinct name as N1,name N2   from stuCopy;
  --得到的新表 表头用 别名来表示 即N1 N2
  ```



### From数据源

**From是为前面的查询提供数据：数据源只要是一个符合二维表结构的数据即可。**

- 单表数据

  - From 表名;

- 多表数据

  - 从多张表获取数据，基本语法：from 表1,表2…

  - 结果：两张表的记录数相乘，字段数拼接    (3列表，7列表 = 21列表)

    本质：从第一张表取出一条记录，去拼凑第二张表的所有记录，保留所有结果。得到的结果在数学上有一个专业的说法：笛卡尔积，这个结果出了给数据库造成压力，没有其他意义：应该尽量避免出现笛卡尔积。

- 动态数据

  - **基本语法：from  (select 字段列表 from 表)  as  别名 ;**

  - From后面跟的数据不是一个实体表，而是一个从表中查询出来得到的二维结果表（**子查询**）。



### Where 子句

+ Where字句：用来从数据表获取数据的时候，然后进行条件筛选。
+ 数据获取原理：针对表去对应的磁盘处获取所有的记录（一条条），where的作用就是在拿到一条结果就开始进行判断，判断是否符合条件：如果符合就保存下来，如果不符合直接舍弃（不放到内存中）
+ **Where是通过运算符进行结果比较来判断数据**。
+ **Where不能使用聚合函数：聚合函数是用在group by分组的时候，where已经运行完毕**



### Group By

Group by表示分组的含义：根据指定的字段，将数据进行分组：**分组的目标是为了统计**

+ 分组统计

  - **基本语法： group by 字段名;**

+ **Group by是为了分组后进行数据统计的，如果只是想看数据显示，那么group by没什么含义：group by将数据按照指定的字段分组之后，只会保留每组的第一条记录。**

   利用一些统计函数：（聚合函数）

  count()：统计每组中的数量，如果统计目标是字段，那么不统计为空NULL字段，如果为*那么代表统计记录

  avg()：求平均值

  sum()：求和

  max()：求最大值

  min()：求最小值

  ```sql
  select class_id,count(*),max(age),min(height),avg(height) from sss group by  class_id;
  --按照class_id分组 
  --呈现形式  列名： class_id,歌班人数,max(age),min(height),avg(height)
  --几行？  ： 看class_id有几个
  ```

+ **Group_concat()**：为了将分组中指定的字段进行合并（字符串拼接）

```sql
select class_id,group_concat(name),count(*),max(age),min(height),avg(height)
from sss group by  class_id;

-- 把每班的同学名字 以Str形式列出来
```

#### 多分组<hr/>

**将数据按照某个字段进行分组之后，对已经分组的数据进行再次分组**

+  基本语法：group by 字段1,字段2; 

  //**先按照字段1进行排序，之后将结果再按照字段2进行排序**，以此类推。

#### 分组排序

Mysql中，分组默认有排序的功能：按照分组字段进行排序，默认是升序

+ **基本语法：group by 字段 [asc|desc]，字段 [asc|desc]**

#### 回溯统计

当分组进行多分组之后，往上统计的过程中，需要进行层层上报，将这种层层上报统计的过程称之为回溯统计：每一次分组向上统计的过程都会产生一次新的统计数据，而且当前数据对应的分组字段为NULL。

+ **基本语法：group by 字段 [asc|desc] with rollup;**
+ 多分组举个例子 ：  NULL就代表着回溯统计

| class_id | Gender | count(*) |
| :------: | :----: | :------: |
|    1     |   男   |    1     |
|    1     |   女   |    2     |
|    1     |  NULL  |    3     |
|    2     |   男   |    2     |
|    2     |   女   |    2     |
|    2     |  NULL  |    4     |
|   NULL   |  NULL  |    7     |



### Having子句

Having的本质和where一样，是用来进行数据条件筛选。

 

+  Having是在group by子句之后：可以针对分组数据进行统计筛选，但是where不行查询班级人数大于等于4个以上的班级
+ **Where不能使用聚合函数：聚合函数是用在group by分组的时候，where已经运行完毕**
+ Having在group by分组之后，可以使用聚合函数或者字段别名（where是从表中取出数据，别名是在数据进入到内存之后才有的）
+ **强调：having是在group by之后，group by是在where之后：where的时候表示将数据从磁盘拿到内存，where之后的所有操作都是内存操作。**



### Order by子句

Order by排序：根据校对规则对数据进行排序

+ **基本语法：order by 字段 [asc|desc]; **          //asc升序，默认的

+ Order by也可以像group by一样进行多字段排序：先按照第一个字段进行排序，然后再按照第二个字段进行排序。

  **Order by 字段1 规则,字段2 规则;**

### Limit子句

Limit限制子句：主要是用来限制记录数量获取

+ **基本语法： limt 数量;**

+ Limit通常在查询的时候如果限定为一条记录的时候，使用的比较多：有时候获取多条记录并不能解决业务问题，但是会增加服务器的压力。

+ **分页**

  - 利用limit来限制获取指定区间的数据。

  - **基本语法：limit offset,length; **   从offset开始获取length个数据

  -  Mysql中记录的数量从0开始

    Limit 0,2; 表示获取前两条记录

  - 注意：limit后面的length表示最多获取对应数量，但是如果数量不够，系统不会强求
  
  
# Select中的运算符号  

###运算：

- +、-、*、/、%
- **基本算术运算：通常不在条件中使用，而是用于结果运算（select 字段中）**

###比较：

- \>、>=、<、<=、=、<>  。between
   **通常是用来在条件中进行限定结果**
  <>这个是不等于
- = 和 <=> 
  - 前者无法比较NULL 即任何值和NULL比都是NULL
  - 后者可以比较NULL 比如 NULL <=> NULL 的结果为1(sql中的true)
- **Between 条件1 and 条件2;**

```sql
select * from stuCopy where id between 1 and 2;     --闭区间
```

- 特殊应用：就是在字段结果中进行比较运算

```sql
select '1' <=> 1 , 1 <=> 2 , 1 <> 2;

--          1         0         1
--select可以单独用来比较 后面不用from
-- sql中没有bool 。 这里 1 为 true ; 0 为 false
```



｀逻辑：

###逻辑：

- **and、or、not**

- **In**

  - In：在什么里面，是用来替代=，当结果不是一个值，而是一个结果集的时候
  - 基本语法： in (结果1,结果2,结果3…)，只要当前条件在结果集中出现过，那么就成立

  ```sql
  select * from stu where stu_id in ('s001','s002','s003');
  ```

  - **查找一个stu_id用 =   查找多个用 in**

- **Is**

  - Is是专门用来判断字段是否为NULL的运算符
  - **基本语法：is null / is not null**

  ```sql
  select * from stu where age is NULL;
  ```

  

- **Like**

  - Like运算符：是用来进行模糊匹配（匹配字符串）

  - 基本语法：like ‘匹配模式’;

  - 匹配模式中，有两种占位符：

    _：匹配对应的单个字符

    %：匹配多个字符

  ```sql
  select * from stu where name like '小_';       -- 必定俩字
  select * from stu where name like '%博';       -- 最后以博结尾就行
  ```



# Union 查询

联合查询是可合并多个相似的选择查询的结果集。等同于将一个表追加到另一个表，从而实现将两个表的查询组合到一起，使用谓词为UNION或UNION ALL。

+ **联合查询：将多个查询的结果合并到一起（纵向合并）：字段数不变，多个查询的记录数合并。**

+ 应用场景：

  - 将同一张表中不同的结果（需要对应多条查询语句来实现），合并到一起展示数据

    男生身高升序排序，女生身高降序排序

  - 最常见：在数据量大的情况下，会对表进行分表操作，需要对每张表进行部分数据统计，使用联合查询来讲数据存放到一起显示。

    QQ1表获取在线数据

    QQ2表获取在线数据 ---》将所有在线的数据显示出来

+ 基本语法：

​      Select 语句

​       Union [union 选项]

​       Select 语句;

-   Union选项：与select选项基本一样

​           Distinct：去重，去掉完全重复的数据（默认的）

​            All：保存所有的结果  

+ **注意细节：union理论上只要保证字段数一样，不需要每次拿到的数据对应的字段类型一致。永远只保留第一个select语句对应的字段名字。**

```sql
select id,name from stu
union all
select name,id from stu;
```

|  Id  | Name |
| :--: | :--: |
|  1   | aaa  |
|  2   | bbb  |
| aaa  |  1   |
| bbb  |  2   |

### Order By的使用

+ 在联合查询中，如果要使用order by，那么对应的select语句**必须使用括号括起来**
+ orderby在联合查询中若要生效，**必须配合使用limit**：而limit后面必须跟对应的限制数量（通常可以使用一个较大的值：大于对应表的记录数）

```sql
(select id,name from stu order by id asc limit 3)
union all
(select name,id from stu order by id desc limit 10);
```

|  Id  | name |      |      |
| :--: | :--: | ---- | ---- |
|  2   | Aaa  |      |      |
|  3   | Bbb  |      |      |
|  4   | Ccc  |      |      |
| Ccc  |  4   |      |      |
| Bbb  |  3   |      |      |
| Aaa  |  2   |      |      |



# 连接查询

连接查询：将多张表连到一起进行查询（会导致记录数行和字段数列发生改变）

+ **连接查询的意义**：

  - 在关系型数据库设计过程中，实体（表）与实体之间是存在很多联系的。在关系型数据库表的设计过程中，遵循着关系来设计：一对一，一对多和多对多，通常在实际操作的过程中，需要利用这层关系来保证数据的完整性。

+ 分类：

  - 交叉连接

  - 内连接

  - 外连接：左外连接（左连接）和右外连接（右连接）

  - 自然连接

### Cross join

交叉连接：将两张表的数据与另外一张表彼此交叉

+ **基本语法：表1 cross join 表2;**

+ 原理

  - 从第一张表依次取出每一条记录

  - 取出每一条记录之后，与另外一张表的全部记录挨个匹配

  - 没有任何匹配条件，所有的结果都会进行保留

  - 记录数 = 第一张表记录数 * 第二张表记录数；字段数 = 第一张表字段数  + 第二张表字段数（笛卡尔积）

+ 应用：

  - 交叉连接产生的结果是笛卡尔积，没有实际应用。

  - **本质：from 表1,表2;**

### Inner Join

内连接：inner join，从一张表中取出所有的记录去另外一张表中匹配：利用匹配条件进行匹配，成功了则保留，失败了放弃。

+ **应用：内连接通常是在对数据有精确要求的地方使用：必须保证两种表中都能进行数据匹配。**
+ 语法： **表1  [inner]  join  表2   on    连接条件**

```sql
create table stu(                             -- 表1 student
  stu_id varchar(10) primary key ,
  stu_name varchar(10) not null ,
  stu_gender enum('男','女'),
  stu_age tinyint,
  class_id tinyint not null

);

insert into stu values                      --表1 插入学生数据
('s02','pb','1',20,1),
('s03','tom','2',22,2),
('s04','pet','2',22,1),
('s05','poi','2',23,1),
('s06','abc','2',20,1),
('s07','tui','1',21,2),
('s08','rew','1',21,2);



create table myclass(                      --表2 班级名
  id int primary key auto_increment ,
  class_name varchar(10) not null
 );

insert into myclass values (null,'大班'),(null,'小班'),(null,'留级班');
```

```sql
select * from stu s inner join myclass m on s.class_id = m.id;
--将student表中的class_id和myclass表中的id 相连接
```

stu_id.   stu_name     stu_gender    age        class_id         id     class_name

s01	phb	男	21	1	1	大班
s02	pb	男	20	1	1	大班
s03	tom	女	22	2	2	小班
s04	pet	女	22	1	1	大班
s05	poi	女	23	1	1	大班
s06	abc	女	20	1	1	大班
s07	tui	男	21	2	2	小班
s08	rew	男	21	2	2	小班

### Outer join

外链接：outer join，按照某一张表作为主表（表中所有记录在最后都会保留），根据条件去连接另外一张表，从而得到目标数据。

**外连接分为两种：左外连接（left join），右外连接（right join）**

​                                 左连接：左表是主表   ;    右连接：右表是主表

+ 原理：

  - 确定连接主表：左连接就是left join左边的表为主表；right join就是右边为主表
  - 拿主表的每一条记录，去匹配另外一张表（从表）的每一条记录
  -  如果满足匹配条件：保留；不满足即不保留
  -  如果主表记录在从表中一条都没有匹配成功，那么也要保留该记录：从表对应的字段值都未NULL

+ **基本语法**：

  - **左连接：主表 left join 从表 on 连接条件;**
  - **右连接：从表 right join 主表 on连接条件;**

+ 特点：

  - 外连接中主表数据记录一定会保存：连接之后不会出现记录数少于主表（内连接可能）

  - 左连接和右连接其实可以互相转换，但是数据对应的位置（表顺序）会改变

+ **应用：非常常用的一种获取的数据方式：作为数据获取对应主表以及其他数据（关联）**

```sql
--用的inner join中的数据
-- outer join
-- stu为主表
select * from stu s left join myclass m on s.class_id = m.id;
select * from myclass m right join stu s on s.class_id = m.id;

--myclass为主表
select * from stu s right join myclass m on s.class_id = m.id;
select * from myclass m left join stu s on s.class_id = m.id;

-- 前表的列名先显示 然后拼接后面表的列名
```

select * from myclass m left join stu s on s.class_id = m.id;

的时候myclass 为主表 且 列名在前 **myclass中的‘留级班’里没有学生数据，但是mycalss为主表 所以连接后的studen表中都为NUll，反之(student为主表)不会有NULL**

1	大班	s01	phb	男	21	1
1	大班	s02	pb	男	20	1
2	小班	s03	tom	女	22	2
1	大班	s04	pet	女	22	1
1	大班	s05	poi	女	23	1
1	大班	s06	abc	女	20	1
2	小班	s07	tui	男	21	2
2	小班	s08	rew	男	21	2
3	留级班	null	null	null	null	null	





###Using关键字

是在连接查询中用来代替对应的on关键字的，进行条件匹配。

+  基本语法：
  **表1  [inner,left,right]  join  表2 using(同名字段列表); **//连接字段

+ 比如说student表中有class_id， myclass表中也有class_id时

```sql
select * from stu join myclass using class_id; 
```



#Sub Query

+ 子查询概念
  - 子查询：sub query
  -  子查询是一种常用计算机语言SELECT-SQL语言中**嵌套查询下层的程序模块**。当一个查询是另一个查询的条件时，称之为子查询。
  - 子查询：指在一条select语句中，嵌入了另外一条select语句，那么被嵌入的select语句称之为子查询语句。
+ 主查询概念：
  - 主查询：主要的查询对象，第一条select语句，确定的用户所有获取的数据目标（数据源），以及要具体得到的字段信息。
+ 关系：
  - 子查询是嵌入到主查询中的；
  - 子查询的辅助主查询的：要么作为条件，要么作为数据源
  - 子查询其实可以独立存在：是一条完整的select语句
+ 分类：
  - 标量子查询：子查询返回的结果是一个数据（一行一列）。  **Where**
  -  列子查询：返回的结果是一列（一列多行）                    **where**
  - 行子查询：返回的结果是一行（一行多列）                     where
  - 表子查询：返回的结果是多行多列（多行多列）                from
  - Exists子查询：返回的结果1或者0（类似布尔操作）
+ 按放的位置分：
  - Where子查询：子查询出现的位置在where条件中
  - From子查询：子查询出现的位置在from数据源中（做数据源）



## 标量子查询

标量子查询：子查询得到结果是一个数据（一行一列）

+ 语法：
  - **基本语法：select * from 数据源 where 条件判断 =/<>(select 字段名 from 数据源 where 条件判断);**
  -  //子查询得到的结果只有一个值

```sql
--已知phb的名字 来查找他所在班级的名字
select  * from myclass m where m.id = (select class_id from stu where stu_name = 'phb');
--主查询是班级名。子查询是phb所在班级id
--子查询：在stu表中找出stu_name是phb的那行数据中的 class_id
-- 主查询： 在class表中找出 和子查询结构一样的 id 
```



## 列子查询

列子查询：子查询得到的结果是一列数据（一列多行）

+ **基本语法：主查询 where 条件 in (列子查询);**

```sql
--   获取已有学生所在班级的所有班级名字
select * from myclass m where m.id in (select class_id from stu);
--子查询：获取所有学生所在的班级id
--主查询 ：看班级表中的id 有哪个属于 子查询集合的

--eg 子查询结果1 2 4 ； 班级表id有1 2 3 ；所以结果就是1 2 
```



## 行子查询

行子查询：子查询返回的结果是一行多列

+ 行元素：字段元素是指一个字段对应的值，行元素对应的就是多个字段：多个字段合起来作为一个元素参与运算，把这种情况称之为行元素。
+ **基本语法：主查询 where 条件[（构造一个行元素）] = (行子查询);**

```sql
-- 查找年龄最大的学生的所有数据
select * from stu where (stu_age) = (select max(stu_age) from stu);
-- 可以构建多个行元素
select * from stu where (stu_age,stu_height) = (select max(stu_age),max(stu_height) from stu);

--注意 下面是错的
select * from stu where stu_age = max(stu_age); -- where后面不可以用聚合函数
```



## 婊子查询

表子查询：子查询返回的结果是多行多列，表子查询与行子查询非常相似，只是行子查询需要产生行元素，而表子查询没有。

 行子查询是用于where条件判断：where子查询

 表子查询是用于from数据源：from子查询

+ 语法 ：**Select 字段表 from (表子查询) as 别名 [where] [group by] [having] [order by] [limit];**
  - **说白了就是在from后面再加一个完整的select语句 且把这个动态表  命名**
  - 因为 from后面只能加   **表名**

```sql
-- 找出性别为男 且是一班的学生
select * from (select * from stu where stu_gender=1) as man where class_id = 1;
```



## Exists子查询

Exists子查询：查询返回的结果只有0或者1，1代表成立，0代表不成立

+ 基本语法：where exists(查询语句); 
  - //exists就是根据查询得到的结果进行判断：如果结果存在，那么返回1，否则返回0
  - **where 1：永远为真**

```sql
select * from myclass m where exists(select * from stu s where s.class_id = m.id)

-- 子查询 ： 看学生表里的 每个学生的class_id和班级表中的id进行匹配 有则返回1 ，没有返回0
```





## Sub Query关键字

+ IN

  - 主查询 where 条件 in (列子查询);

+ Any

  - **= any(列子查询)**：条件在查询结果中有任意一个匹配即可，等价于in

  - **<>any(列子查询)**：条件在查询结果中不等于任意一个

  - 1  =any(1,2,3)  ===== true

    1  <>any(1,2,3)  ===== true

+ Some

  - 与any完全一样

+ All

  - **= all(列子查询)**：等于里面所有
  - **<>all(列子查询)**：不等于其中所有
  - 如果对应的匹配字段有NULL，那么不参与匹配

















 



