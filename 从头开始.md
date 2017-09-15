从头开始
====

>SQL是对关键字和标识符大小写不敏感的语言，只有在标识符用双引号包围时才能保留它们的大小写。 

SQL语言
----

* 类型
```sql
--以下描述仅为猜想，orz
int  --整型int
smallint  --短整型short
real  --单精度浮点型float
double precision  --双精度浮点型double
char(N)
varchar(N)
date  --可自行解释类型
time
timestamp
interval
point
```

* 创建一个新表
```sql
--创建表
CREATE TABLE tablename (
    name       int, --name是列名，int是列的类型，"--"是注释的开头
    ...
); 

--删除表
DROP TABLE tablename; 
```
* 在表中增加行
```sql
--插入一行
INSERT INTO tablename (name1, name2, name3, name5)  --此行括号内容可缺省，加入括号可选择插入哪些列
    VALUES (value1, value2, value3, value5);

--从文本文件中装载大量数据
--源文件的文件名必须在运行后端进程的机器上是可用的， 而不是在客户端上，因为后端进程将直接读取该文件
COPY tablename FROM '/home/user/tablename.txt';
```

- 查询一个表

    一个查询可以使用WHERE子句"修饰"，它指定需要哪些行。WHERE子句包含一个布尔（真值）表达式，只有那些使布尔表达式为真的行才会被返回。在条件中可以使用常用的布尔操作符（AND、OR和NOT）。
```sql
SELECT * FROM tablename;  --这里*是"所有列"的缩写。等于：
SELECT name, ..., ... FROM tablename;

SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;  
--AS子句是如何给输出列重新命名的（AS子句是可选的）

SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;

SELECT * FROM weather
    ORDER BY city;  --ORDER BY为排序
    
SELECT DISTINCT city
    FROM weather;  --DISTINCT可以查询的结果中消除重复的行
```

- 在表之间连接
    
    类似于拿 weather表每行的city列和cities表所有行的name列进行比较， 并选取那些在该值上相匹配的行对。但这里只是一个概念上的模型。连接通常以比实际比较每个可能的行对更高效的方式执行。
```sql
SELECT W1.city, W1.temp_lo AS low, W1.temp_hi AS high,  
    W2.city, W2.temp_lo AS low, W2.temp_hi AS high
--如果在两个表里有重名的列，你需要限定列名来说明你究竟想要哪一个
    FROM weather W1, weather W2
--可以把一个表和自己连接起来。这叫做自连接
--在这里我们把weather表重新标记为W1和W2以区分连接的左部和右部。你还可以用这样的别名在其它查询里节约一些敲键
    WHERE W1.temp_lo < W2.temp_lo
    AND W1.temp_hi > W2.temp_hi;  

--到目前为止，这种类型的连接查询也可以用下面这样的形式写出来
SELECT *
    FROM weather INNER JOIN cities ON (weather.city = cities.name);

--如果我们没有找到匹配的行，那么我们需要一些"空值"代替cities表的列。 
--这种类型的查询叫外连接 （我们在此之前看到的连接都是内连接）
SELECT *
    FROM weather LEFT OUTER JOIN cities ON (weather.city = cities.name);  --此为左外连接
```

* 聚集函数

    理解聚集和SQL的WHERE以及HAVING子句之间的关系对我们非常重要。WHERE和HAVING的基本区别如下：WHERE在分组和聚集计算之前选取输入行（因此，它控制哪些行进入聚集计算）， 而HAVING在分组和聚集之后选取分组行。因此，WHERE子句不能包含聚集函数； 因为试图用聚集函数判断哪些行应输入给聚集运算是没有意义的。相反，HAVING子句总是包含聚集函数（严格说来，你可以写不使用聚集的HAVING子句， 但这样做很少有用。同样的条件用在WHERE阶段会更有效）。
```sql
--例如，行集合上计算count（计数）、sum（和）、avg（均值）、max（最大值）和min（最小值）的函数
SELECT max(temp_lo) FROM weather;
SELECT city FROM weather WHERE temp_lo = max(temp_lo);     --错误
--聚集max不能被用于WHERE子句中
--（存在这个限制是因为WHERE子句决定哪些行可以被聚集计算包括；因此显然它必需在聚集函数之前被计算）

SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);  
--可以使用子查询，因为子查询是一次独立的计算，它独立于外层的查询计算出自己的聚集
    
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city;  --GROUP BY为分组

SELECT city, max(temp_lo)
    FROM weather
    WHERE city LIKE 'S%'(1)  --LIKE操作符进行模式匹配
    GROUP BY city
    HAVING max(temp_lo) < 40;
```

* 更新和删除
```sql
--更新
UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';

--删除
DELETE FROM weather WHERE city = 'Hayward';
DELETE FROM tablename;  --如果没有一个限制，DELETE将从指定表中删除所有行，把它清空。做这些之前系统不会请求你确认
```

高级特性
----

- 视图

    假设天气记录和城市为止的组合列表对我们的应用有用，但我们又不想每次需要使用它时都敲入整个查询。我们可以在该查询上创建一个视图，这会给该查询一个名字，我们可以像使用一个普通表一样来使用它。   
    对视图的使用是成就一个好的SQL数据库设计的关键方面。视图允许用户通过始终如一的接口封装表的结构细节，这样可以避免表结构随着应用的进化而改变。
```sql
CREATE VIEW myview AS  --创建视图
    SELECT city, temp_lo, temp_hi, prcp, date, location
        FROM weather, cities
        WHERE city = name;

SELECT * FROM myview;
```

- 外键

    我们希望确保在cities表中有相应项之前任何人都不能在weather表中插入行。这叫做维持数据的引用完整性。在过分简化的数据库系统中，可以通过先检查cities表中是否有匹配的记录存在，然后决定应该接受还是拒绝即将插入weather表的行。这种方法有一些问题且并不方便，于是PostgreSQL可以为我们来解决。
```sql
CREATE TABLE cities (
        city     varchar(80) primary key,  --主键
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(city),  --外键
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);

--现在尝试插入一个非法的记录：
INSERT INTO weather VALUES ('Berkeley', 45, 53, 0.0, '1994-11-28');
--返回：
ERROR:  insert or update on table "weather" violates foreign key constraint "weather_city_fkey"
DETAIL:  Key (city)=(Berkeley) is not present in table "cities".
```

- 事务
    
    事务是所有数据库系统的基础概念。事务最重要的一点是它将多个步骤捆绑成了一个单一的、要么全完成要么全不完成的操作。步骤之间的中间状态对于其他并发事务是不可见的，并且如果有某些错误发生导致事务不能完成，则其中任何一个步骤都不会对数据库造成影响。    
    我们需要一种保障，当操作中途某些错误发生时已经执行的步骤不会产生效果。将这些更新组织成一个事务就可以给我们这种保障。一个事务被称为是原子的：从其他事务的角度来看，它要么整个发生要么完全不发生。    
    我们同样希望能保证一旦一个事务被数据库系统完成并认可，它就被永久地记录下来且即便其后发生崩溃也不会被丢失。    
    事务型数据库的另一个重要性质与原子更新的概念紧密相关：当多个事务并发运行时，每一个都不能看到其他事务未完成的修改。    
    所以事务的全做或全不做并不只体现在它们对数据库的持久影响，也体现在它们发生时的可见性。一个事务所做的更新在它完成之前对于其他事务是不可见的，而之后所有的更新将同时变得可见。    
    在PostgreSQL中，开启一个事务需要将SQL命令用BEGIN和COMMIT命令包围起来。    
    如果，在事务执行中我们并不想提交（或许是我们注意到Alice的余额不足），我们可以发出ROLLBACK命令而不是COMMIT命令，这样所有目前的更新将会被取消。  
    PostgreSQL实际上将每一个SQL语句都作为一个事务来执行。如果我们没有发出BEGIN命令，则每个独立的语句都会被加上一个隐式的BEGIN以及（如果成功）COMMIT来包围它。一组被BEGIN和COMMIT包围的语句也被称为一个事务块。    
    
    > 某些客户端库会自动发出BEGIN和COMMIT命令，因此我们可能会在不被告知的情况下得到事务块的效果。具体请查看所使用的接口文档。    
    
    也可以利用保存点来以更细的粒度来控制一个事务中的语句。保存点允许我们有选择性地放弃事务的一部分而提交剩下的部分。在使用SAVEPOINT定义一个保存点后，我们可以在必要时利用ROLLBACK TO回滚到该保存点。该事务中位于保存点和回滚点之间的数据库修改都会被放弃，但是早于该保存点的修改则会被保存。    
    在回滚到保存点之后，它的定义依然存在，因此我们可以多次回滚到它。反过来，如果确定不再需要回滚到特定的保存点，它可以被释放以便系统释放一些资源。记住不管是释放保存点还是回滚到保存点都会释放定义在该保存点之前的所有其他保存点。   
    所有这些都发生在一个事务块内，因此这些对于其他数据库会话都不可见。当提交整个事务块时，被提交的动作将作为一个单元变得对其他会话可见，而被回滚的动作则永远不会变得可见。    
    在一个事务块中使用保存点存在很多种控制可能性。此外，ROLLBACK TO是唯一的途径来重新控制一个由于错误被系统置为中断状态的事务块，而不是完全回滚它并重新启动。  
    记住那个银行数据库，假设我们从Alice的账户扣款100美元，然后存款到Bob的账户，结果直到最后才发现我们应该存到Wally的账户。我们可以通过使用保存点来做这件事：     
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Wally';
COMMIT;
```

- 窗口函数

    一个窗口函数在一系列与当前行有某种关联的表行上执行一种计算。这与一个聚集函数所完成的计算有可比之处。但是与通常的聚集函数不同的是，使用窗口函数并不会导致行被分组成为一个单独的输出行--行保留它们独立的标识。在这些现象背后，窗口函数可以访问的不仅仅是查询结果的当前行。    
    一个窗口函数调用总是包含一个直接跟在窗口函数名及其参数之后的OVER子句。这使得它从句法上和一个普通函数或聚集函数区分开来。OVER子句决定究竟查询中的哪些行被分离出来由窗口函数处理。OVER子句中的PARTITION BY列表指定了将具有相同PARTITION BY表达式值的行分到组或者分区。对于每一行，窗口函数都会在当前行同一分区的行上进行计算。    
    一个窗口函数所考虑的行属于那些通过查询的FROM子句产生并通过WHERE、GROUP BY、HAVING过滤的"虚拟表"。例如，一个由于不满足WHERE条件被删除的行是不会被任何窗口函数所见的。在一个查询中可以包含多个窗口函数，每个窗口函数都可以用不同的OVER子句来按不同方式划分数据，但是它们都作用在由虚拟表定义的同一个行集上。    
    我们已经看到如果行的顺序不重要时ORDER BY可以忽略。PARTITION BY同样也可以被忽略，在这种情况下只会产生一个包含所有行的分区。    
    这里有一个与窗口函数相关的重要概念：对于每一行，在它的分区中的行集被称为它的窗口帧。 很多（但不是全部）窗口函数只作用在窗口帧中的行上，而不是整个分区。默认情况下，如果使用ORDER BY，则帧包括从分区开始到当前行的所有行，以及后续任何与当前行在ORDER BY子句上相等的行。如果ORDER BY被忽略，则默认帧包含整个分区中所有的行。    
    窗口函数只允许出现在查询的SELECT列表和ORDER BY子句中。它们不允许出现在其他地方，例如GROUP BY、HAVING和WHERE子句中。这是因为窗口函数的执行逻辑是在处理完这些子句之后。另外，窗口函数在普通聚集函数之后执行。这意味着可以在窗口函数的参数中包括一个聚集函数，但反过来不行。    
    如果需要在窗口计算执行后进行过滤或者分组，我们可以使用子查询。    
    当一个查询涉及到多个窗口函数时，可以将每一个分别写在一个独立的OVER子句中。但如果多个函数要求同一个窗口行为时，这种做法是冗余的而且容易出错的。替代方案是，每一个窗口行为可以被放在一个命名的WINDOW子句中，然后在OVER中引用它。    
```sql
SELECT depname, empno, salary,
       rank() OVER (PARTITION BY depname ORDER BY salary DESC) FROM empsalary;
--OVER子句形成rank()窗口函数；PARTITION BY将行按depname分组，ORDER BY对每组进行排序，DESC表示降序

--没加ORDER BY
SELECT salary, sum(salary) OVER () FROM empsalary;
--结果：
 salary |  sum
--------+-------
   5200 | 47100
   5000 | 47100
   3500 | 47100
   4800 | 47100
   3900 | 47100
   4200 | 47100
   4500 | 47100
   4800 | 47100
   6000 | 47100
   5200 | 47100
--加入ORDER BY
SELECT salary, sum(salary) OVER (ORDER BY salary) FROM empsalary;
--结果：
 salary |  sum
--------+-------
   3500 |  3500
   3900 |  7400
   4200 | 11600
   4500 | 16100
   4800 | 25700
   4800 | 25700
   5000 | 30700
   5200 | 41100
   5200 | 41100
   6000 | 47100
   
--使用子查询：
SELECT depname, empno, salary, enroll_date
FROM
  (SELECT depname, empno, salary, enroll_date,
          rank() OVER (PARTITION BY depname ORDER BY salary DESC, empno) AS pos
     FROM empsalary
  ) AS ss
WHERE pos < 3;

--窗口命名w
SELECT sum(salary) OVER w, avg(salary) OVER w
  FROM empsalary
  WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);
```

- 继承

```sql
--继承的写法
CREATE TABLE cities (
  name       text,
  population real,
  altitude   int     -- (in ft)
);

CREATE TABLE capitals (
  state      char(2)
) INHERITS (cities);  --capitals继承至cities

--在这种情况下，一个capitals的行从它的父亲cities继承了所有列（name、population和altitude）。
--列name的类型是text，一种用于变长字符串的本地PostgreSQL类型。州首都有一个附加列state用于显示它们的州。
--在PostgreSQL中，一个表可以从0个或者多个表继承。

--查询可以寻找所有海拔500尺以上的城市名称，包括州首都：
SELECT name, altitude
  FROM cities
  WHERE altitude > 500;
--返回
   name    | altitude
-----------+----------
 Las Vegas |     2174
 Mariposa  |     1953
 Madison   |      845

--查询可以查找所有海拔高于500尺且不是州首府的城市：
SELECT name, altitude
    FROM ONLY cities
    WHERE altitude > 500;
--返回：
   name    | altitude
-----------+----------
 Las Vegas |     2174
 Mariposa  |     1953
```
    其中cities之前的ONLY用于指示查询只在cities表上进行而不会涉及到继承层次中位于cities之下的其他表。  
    很多我们已经讨论过的命令 — SELECT、UPDATE 和DELETE — 都支持这个ONLY记号。  
