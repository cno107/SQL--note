# MySQL

**<font color=#FF0000 >Structured.  Query   Language</font>**

# 基礎知識

## SQL分類

+ **DQL**:Data Query Language
  - 专门用于查询数据：代表指令为select/show
+ **DML**：Data Manipulation Language
  - 专门用于写数据：代表指令为insert，update和delete
+ **TPL**  
  - 专门用于事务安全处理：BEGIN TRANSACTION，COMMIT和ROLLBACK
+ **DCL** : Data Control Language
  - 专门用于权限管理：代表指令为grant和revoke
+ **DDL** :  (Data Definition Language)
  - 专门用于结构管理：代表指令create和drop（alter）

## ターミナル操作

Mac      ```mysql -u root -p```

退出。   ```Exit```                                ``\q``

## Mysql服务端架构

 Mysql服务端架构有以下几层构成：

 

+ 数据库管理系统（最外层）：DBMS，专门管理服务器端的所有内容
+  数据库（第二层）：DB，专门用于存储数据的仓库（可以有很多个）
+ ​    二维数据表（第三层）：Table，专门用于存储具体实体的数据
+ ​     字段（第四层）：Field，具体存储某种类型的数据（实际存储单元）

数据库中常用的几个关键字

Row：行

Column：列（field）





# DB基礎操作

数据库是数据存储的最外层（最大单元） 

 ## Create DB

  - 基本语法：create database 数据库名字 [库选项];

  - 库选项：数据库的相关属性

    字符集：charset 字符集，代表着当前数据库下的所有表存储的数据默认指定的字符集（如果当前不指定，那么采用DBMS默认的）

    校对集：collate 校对集

  - ```sql
    create database mydatabase charset utf8;
    ```

 ##  Show DB

  + 基本语法：show  databases;
  
  + 默认数据库：
    - ：information_schema ：保存db中所有结构info
    - ：mysql： core db 权限关系
    - ：performance_schema 效率库
    - ；test  测试：空库
  + 显示部分： show databases like ‘匹配模式’; 
    - _：匹配当前位置单个字符
    - %：匹配指定位置多个字符 
    - ```
      获取以my开头的全部数据库： 'my%';
      获取m开头，后面第一个字母不确定，最后为database的数据库；'m_database';
      获取以database结尾的数据库：'%database';
        记住一定要加引号
      ```
  + 显示数据库创建语句

    - 基本语法：show create database 数据库名字;

## Choose DB
  + 基本语法：use 数据库名字
## Alter DB
  + 修改数据库字符集(库选项)：字符集和校对集
    基本语法：alter database 数据库名字 charset = 字符集;
## Drop DB
  + 基本语法：drop database 数据库名字;

# Table基礎操作
## Create Table
 + 普通创建 
    - 基本语法：create table 表名(字段名 字段类型 [字段属性], 字段名 字段类型 [字段属性],…)  
    - ```sql
      create table xxx(
      -- 字段名 字段类型(这俩分不开)
      name varchar(10) -- 不能超过10字符
      
      );
      ```
    - 如何把表创建到指定DB下
      - 在数据表名字前面加上数据库名字，用“.”连接即可：数据库.数据表
      - 在创建之前进入DB。 use 数据库名字
  
 + 表选项：与数据库选项类似
   -  Engine：存储引擎，mysql提供的具体存储数据的方式，默认有一个innodb（5.5以前默认是myisam）
   -  Charset：字符集，只对当前自己表有效（级别比数据库高）
   -  Collate：校对集
 + 复制已有表结构
   - 从已经存在的表复制一份（只复制结构：如果表中有数据不复制）
   - 基本语法：create table 新表名 like 表名;
## Show Table
 + 基本语法：show tables;
 + 基本语法：show tables like  ‘匹配模式’;

   - _ %
 + 显示表结构
   - 本质含义：显示表中所包含的字段信息（名字，类型，属性等）
   - ```sql
        这仨一样 
        Describe 表名
        Desc 表名
        show columns from 表名
     ```
 + show create table 表名;

## Set Table attribute    
 表属性指的就是表选项：engine，charset和collate
 + 基本语法：alter table 表名 表选项 [=] 值;
   - ```alter table student2 charset gbk;```
 + 修改表名：rename table 旧表名 to 新表名
 + 修改表选项：alter table 表名 表选项 [=] 新值
 + 新增字段：alter table 表名 add [column] 新字段名 列类型 [列属性] [位置first/after 字段名]
    - ```alter table my_student add id int first ;```
 + 修改字段名：alter table 表名 change 旧字段名 新字段名 字段类型 [列属性] [新位置]
    - ```alter table my_student change  age nj int;```   
 + <font color="red">修改字段类型（属性）</font>：alter table 表名 modify 字段名 新类型 [新属性] [新位置]
    - ```alter table my_student modify  name varchar(20);```
    - ```alter table mDate modify d4 timestamp not null ;```
 + 删除字段：alter table 表名 drop 字段名
   -  ```alter table my_student drop nj;``` 
## Drop Table
 基础语法：drop table 表名[,表名2…]，可以同时删除多个数据表
+ ```drop table mydatabase.class,mydatabase.student;```   

# Data基礎操作
##Insert
本质含义：将数据以SQL的形式存储到指定的数据表（字段）里面
+ 基本语法：Insert into 表名[(字段列表)] values(对应字段列表)
  - ```insert into my_student (id,name) values(1,'phb');```
  - (字段列表)必须与values(对应字段列表)对应，但不用和table结构对应,也不用全部填写
+ 省略[(字段列表)]
  - 必须与表结构完全一样 比如表字段有id,name,age,skill
  - ```insert into my_student values(3,'xxx',18,'cook');```  
  必须四个值都给 且必须按顺序对应 缺一不可
  
##查询Select
 + 查询全部:  select * from 表名;  
   - */ 表是所有
 + 查询部分字段: select 字段1,字段2 from 表名;
   - 用','连接
 + 条件查询: select 字段列表/* from 表名 where 字段名 = 值;
   - mysql里没有==
   - ```select * from my_student where age = 18;``` 
   - ```select name from my_student where name = 'xxx';```
##Delete
 + 基本语法：delete from 表名 [where 条件];
   - 如果没有where条件：意味着系统会自动删除该表所有数据（慎用）
   - ```delete from my_student where name = 'xxx';```
##Update
更新：将数据进行修改（通常是修改部分字段数据）
+ 基本语法：update 表名 set 字段名 = 新值 [where 条件];
  - 如果没有where条件，那么所有的表中对应的那个字段都会被修改成统一值。
  - ```update my_student set skill='web' where name='phb';```

     




   	  


​     




