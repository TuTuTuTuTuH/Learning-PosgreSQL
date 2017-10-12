SQL 语言
====

语法结构
----
```sql
关键词：CREAT  
标识符：tablename  
```
> 关键词和不被引号修饰的标识符是大小写不敏感的。  
受限标识符或被引号修饰的标识符是由双引号（"）包围的一个任意字符序列。  
一个受限标识符总是一个标识符而不会是一个关键字。  

```sql
- U&"name" 转义的用代码点标识的Unicode字符
- 反斜线接上4位16进制代码点号码或者反斜线和加号接上6位16进制代码点号码
U&"d\0061t\+000061" == "data"
U&"\0441\043B\043E\043D" == "slon"
U&"d!0061t!+000061" UESCAPE '!' - UESCAPE使'!'代替'\'
```

> PostgreSQL将非受限名字转换为小写形式与SQL标准是不兼容 的，SQL标准中要求将非受限名称转换为大写形式。  

- 常量  
    隐式类型常量：字符串、位串和数字  
    
    两个只由空白及至少一个新行分隔的字符串常量会被连接在一起，并且将作为一个写在一起的字符串常量来对待
```sql
SELECT 'foo'
'bar';
- 等于
SELECT 'foobar';
- 但是
SELECT 'foo'      'bar'; - 不合法
```

表 4-1. 反斜线转义序列  
```sql
反斜线转义序列	                            解释  
\b	                                    退格  
\f                                          换页  
\n	                                    换行  
\r	                                    回车  
\t	                                    制表符  
\o, \oo, \ooo (o = 0 - 7)	            八进制字节值  
\xh, \xhh (h = 0 - 9, A - F)	            十六进制字节值  
\uxxxx, \Uxxxxxxxx (x = 0 - 9, A - F)	    16 或 32-位十六进制 Unicode 字符值  
```

美元引用：$      
    美元引用字符串中不需要对字符进行转义：字符串内容总是按其字面意思写出。  
```sql
- 指定字符串"Dianne's horse"
$$Dianne's horse$$
$SomeTag$Dianne's horse$SomeTag$
```

位串常量：  
```sql
B'1001' - b大小写均可，只能接01
X'1FF' - 16进制
- 可跨行连续
```

数字常量：  
    digits是一个或多个十进制数字（0 到 9）。如果使用了小数点，在小数点前面或后面必须至少有一个数字。如果存在一个指数标记（e），在其后必须跟着至少一个数字。
```sql
digits
digits.[digits][e[+-]digits]
[digits].digits[e[+-]digits]
digitse[+-]digits
- 例如：
42
3.5
4.
.001
5e2
1.925e-3
- 使用real强转数值类型
REAL '1.23'  -- string style
1.23::REAL   -- PostgreSQL (historical) style
```

类型转换：  
```sql
type 'string'
'string'::type
CAST ( 'string' AS type )

typename ( 'string' )
```

操作符：   
```sql
+ - * / < > = ~ ! @ # % ^ & | ` ?
```

特殊符号：
```sql
$ () [] , ; : * .  
```
    
注释：  
```sql
除'-'外，可以使用C风格注释/* */     
```
    
操作符优先级：  
```sql
表 4-2. 操作符优先级（最高到最低）

操作符/元素	                         结合性	        描述
.	                                   左	       表/列名分隔符
::	                                   左	       PostgreSQL-风格的类型转换
[ ]	                                   左	       数组元素选择
+ -	                                   右	       一元加、一元减
^	                                   左	       指数
* / %	                                   左	       乘、除、模
+ -	                                   左	       加、减
（任意其他操作符）	                   左          所有其他本地以及用户定义的操作符
BETWEEN IN LIKE ILIKE SIMILAR	 	               范围包含，设置成员，字符串匹配
< > = <= >= <>	 	                               比较操作符
IS ISNULL NOTNULL	 	                       IS TRUE、IS FALSE、IS NULL、IS DISTINCT FROM等
NOT	                                   右	       逻辑否定
AND	                                   左	       逻辑合取
OR	                                   左	       逻辑析取
```

值表达式
----  

- 列引用  
```sql
correlation.columnname
```  
    correlation是一个表（有可能以一个模式名限定）的名字，或者是在FROM子句中为一个表定义的别名。  
    
- 位置参数  
```sql
$number

CREATE FUNCTION dept(text) RETURNS dept
    AS $$ SELECT * FROM dept WHERE name = $1 $$
    LANGUAGE SQL;
- $1引用函数被调用时第一个函数参数的值。
```

- 下标  
```sql
expression[subscript] - 按秩查询

expression[lower_subscript:upper_subscript] - 查询多个相邻元素（数组切片）

mytable.arraycolumn[4]
mytable.two_d_column[17][34]
$1[10:42]
(arrayfunction(a,b))[42]
```  
通常，数组表达式必须被加上括号，但是当要被加下标的表达式只是一个列引用或位置参数时，括号可以被忽略。还有，当原始数组是多维时，多个下标可以被连接起来。  
- 域选择  
```sql
expression.fieldname

mytable.mycolumn
$1.somecolumn
(rowfunction(a,b)).col3

- 这里需要圆括号来显示compositecol是一个列名而不是一个表名。
- 在第二种情况中则是显示mytable是一个表名而不是一个模式名。
(compositecol).somefield
(mytable.compositecol).somefield

- 可以通过书写.*来请求一个组合值的所有域。
(compositecol).*
```

- 操作符调用  
```sql
expression operator expression -（二元中缀操作符）
operator expression -（一元前缀操作符）
expression operator -（一元后缀操作符）

OPERATOR(schema.operatorname) - 受限定操作符名
```

- 函数调用  
    一个函数调用的语法是一个函数的名称（可能受限于一个模式名）后面跟上封闭于圆括号中的参数列表：  
```sql
function_name ([expression [, expression ... ]] )

sqrt(2)
```

- 聚集表达式  
```sql
aggregate_name (expression [ , ... ] [ order_by_clause ] ) [ FILTER ( WHERE filter_clause ) ]
aggregate_name (ALL expression [ , ... ] [ order_by_clause ] ) [ FILTER ( WHERE filter_clause ) ]
aggregate_name (DISTINCT expression [ , ... ] [ order_by_clause ] ) [ FILTER ( WHERE filter_clause ) ]
aggregate_name ( * ) [ FILTER ( WHERE filter_clause ) ]
aggregate_name ( [ expression [ , ... ] ] ) WITHIN GROUP ( order_by_clause ) [ FILTER ( WHERE filter_clause ) ]

SELECT array_agg(a ORDER BY b DESC) FROM table;

SELECT string_agg(a, ',' ORDER BY a) FROM table;

SELECT string_agg(a ORDER BY a, ',') FROM table;  -- 不正确

SELECT percentile_disc(0.5) WITHIN GROUP (ORDER BY income) FROM households;
 percentile_disc
-----------------
           50489
           
SELECT
    count(*) AS unfiltered,
    count(*) FILTER (WHERE i < 5) AS filtered
FROM generate_series(1,10) AS s(i);
 unfiltered | filtered
------------+----------
         10 |        4
(1 row)
```

- 窗口函数调用  
```sql
function_name ([expression [, expression ... ]]) [ FILTER ( WHERE filter_clause ) ] OVER window_name
function_name ([expression [, expression ... ]]) [ FILTER ( WHERE filter_clause ) ] OVER ( window_definition )
function_name ( * ) [ FILTER ( WHERE filter_clause ) ] OVER window_name
function_name ( * ) [ FILTER ( WHERE filter_clause ) ] OVER ( window_definition )

-- 其中window_definition的语法是
[ existing_window_name ]
[ PARTITION BY expression [, ...] ]
[ ORDER BY expression [ ASC | DESC | USING operator ] [ NULLS { FIRST | LAST } ] [, ...] ]
[ frame_clause ]

-- 可选的frame_clause是下列之一
{ RANGE | ROWS } frame_start
{ RANGE | ROWS } BETWEEN frame_start AND frame_end

-- frame_start和frame_end可以是下面形式中的一种
UNBOUNDED PRECEDING
value PRECEDING
CURRENT ROW
value FOLLOWING
UNBOUNDED FOLLOWING
```
