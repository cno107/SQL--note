# where &聚合函数

+ 大致解释如下，sql语句的执行过程是：from-->where-->group by -->having --- >order by --> select;

+ 聚合函数是针对结果集进行的，但是where条件并不是在查询出结果集之后运行，所以主函数放在where语句中，会出现错误，

+ 而having不一样，having是针对结果集做筛选的，所以我门一般吧组函数放在having中，用having来代替where，having一般跟在group by后



# PK & auto_increament

+ pk且自增长 时 无法直接drop pk ； 必须先drop auto 才能drop pk



# * 和 %的区别

```sql
-- 直接裸露在外的用*    在引号里的用%

mydb.*

'user'@'%'
```

# = 和 ：=

在mysql中因为没有比较符号==，所以是用=代替比较符号：有时候在赋值的时候，会报错：mysql为了避免系统分不清是赋值还是比较：特定增加一个变量的赋值符号：  :=