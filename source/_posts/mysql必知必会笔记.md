title: mysql必知必会读书笔记
author: 禾田
tags:
  - mysql
categories:
  - mysql
date: 2018-04-25 10:26:00
description: mysql必知必会读书笔记
---
### 基本知识
1. 关系数据库设计把数据存储在多个表中，使数据更容易操纵、维护和重用。不用深究如何以及为什么进行关系数据库设计，在某种程度上说，设计良好的数据库模式都是关联的。

2. help show | select | update ...; 查看相关命令的帮助

3. MySql语句都以（；）结尾。（多条SQL语句必须以分号（；）分隔。 MySQL如同多数DBMS一样，不需要在单条SQL语句后加分号。但特定的DBMS可能必须在单条SQL语句后加上分号。当然，如果愿意可以总是加上分号。事实上，即使不一定需要，但加上分号肯定没有坏处。如果你使用的是mysql命令行，必须加上分号来结束SQL语句。）
4.  对所有SQL关键字使用大写，而对所有列和表名使用小写。（SQL语句不区分大小写，因此SELECT与select是相同的。同样，写成Select也没有关系。许多SQL开发人员喜欢对所有SQL关键字使用大写，而对所有列和表名使用小写，这样做使代码更易于阅读和调试。）

5. 将SQL语句分成多行更容易阅读和调试。（使用空格 在处理SQL语句时，其中所有空格都被忽略。 SQL语句可以在一行上给出，也可以分成许多行。多数SQL开发人员认为将SQL语句分成多行更容易阅读和调试。）

---

### 连接MySQL数据库 
```SQL
mysql –u用户名 [–h主机名或者IP地址] –p密码
```

说明：用户名是你登录的用户，主机名或者IP地址为可选项，如果是本地连接则不需要，远程连接需要填写，端口默认是3306，否则需要填写端口号，密码是对应用户的密码。
 
注意：
1. 该命令是在Windows命令行窗口下执行，而不是MySQL的命令行；  
2. 输入-p后可以直接跟上密码，也可以按回车，会提示你输入密码，二者都是相同的效果；
3. –p密码选项不一定是要在最后；
4. –u、-h、-p后无空格。


---

### 数据库基本操作 
#### 选择数据库  
```SQL
USE 数据库名; 
```  

注意： 必须先使用USE打开数据库，才能读取其中的数据。

#### 2. 显示数据库和表
```SQL
SHOW DATABASES;  
```

说明：返回可用数据库的一个列表。包含在这个列
表中的可能有MySQL内部使用的数据库。

```SQL
SHOW TABLES; 
```

说明：返回当前选择的数据库内可用表的列表。

```SQL
SHOW COLUMNS FROM 表名;  
DESCRIBE 表名;（更快捷的方式）
```

说明：要求给出一个表名，它对每个字段返回一行，行中包含字段名、数据类型、是否允许NULL、键信息、默认值以及其他信息。

---

### 数据查询
```SQL
SELECT 列名 FROM 表名;
```

说明：上述语句利用SELECT语句从表中检索一列。所需的列名在SELECT关键字之后给出， FROM关键字指出从其中检索数据的表名。

注意：如果没有明确排序查询结果，则返回的数据的顺序没有特殊意义。返回数据的顺序可能是数据被添加到表中的顺序，也可能不是。只要返回相同数目的行，就是正常的。

```SQL
SELECT 列名1, 列名2, 列名3 FROM 表名;
```

说明：这条语句使用SELECT语句从表中选择数据。在这个例子中，指定了3个列名，列名之间用逗号分隔。

```SQL
SELECT * FROM 表名;  
```

说明：如果给定一个通配符（*），则返回表中所有列。列的顺序一般是列在表定义中出现的顺序。但有时候并不是这样的，表的模式的变化（如添加或删除列）可能会导致顺序的变化。

注意：
1. 一般，除非你确实需要表中的每个列，否则最好别使用*通配符。虽然使用通配符可能会使你自己省事，不用明确列出所需列，但检索不需要的列通常会降低检索和应用程序的性能。
2. 使用通配符有一个大优点。由于不明确指定列名（因为星号检索每个列），所以能检索出名字未知的列。

```SQL
SELECT DISTINCT 列名 FROM 表名; 
```

说明：SELECT DISTINCT列名告诉MySQL只返回不同（唯一）的对应列名的行，去掉重复行，如果使用DISTINCT关键字，它必须直接放在列名的前面。

注意：不能部分使用DISTINCT，DISTINCT关键字应用于所有列而不仅是前置它的列。如果给出SELECT DISTINCT 列1,列2，除非指定的两个列都不同，否则所有行都将被检索出来。 

```SQL
SELECT 列名 FROM 表名 LIMIT 5, 5;   
SELECT 列名 FROM 表名 LIMIT 5;
```

说明：
1. LIMIT 5, 5指示MySQL返回从行5开始的5行。第一个数为开始位置，第二个数为要检索的行数。
2. LIMIT 5 指示MySQL返回不多于5行。

注意：
1. 行0检索出来的第一行为行0而不是行1。因此，LIMIT 1, 1将检索出第二行而不是第一行。
2. 在行数不够时，LIMIT中指定要检索的行数为检索的最大行数。如果没有足够的行（例如，给出LIMIT 10, 5，但只有13行），MySQL将只返回它能返回的那么多行。

```SQL
SELECT 表名.列名 FROM 数据库名.表名;
```

说明：指定了一个完全限定的列名和表名。

### 排序查询数据
```SQL
SELECT 列名 FROM 表名 ORDER BY 列名;  
```

说明：将输出的结果根据列进行排序。 

注意：通过非选择列进行排序。通常，ORDER BY子句中使用的列将是为显示所选择的列。但是，实际上并不一定要这样，用非检索的列排序数据是完全合法的。

```SQL
SELECT 列名1, 列名2, 列名3 FROM 表名 ORDER BY 列名1, 列名2;  
```

说明：上面的代码将检索3个列，并按其中两个列对结果进行排序，首先按
列1，然后再按列2排序。

```SQL
SELECT 列名1, 列名2, 列名3 FROM 表名 ORDER BY 列名1 DESC, 列名2 ;  
```

说明：DESC关键字只应用到直接位于其前面的列名。在上例中，只对列名1指定DESC，对列名2不指定。因此，列名1以降序排序，而列名2（在每个价格内）仍然按标准的升序排序。

注意：在多个列上降序排序，如果想在多个列上进行降序排序，必须对每个列指定DESC关键字。

应用：使用ORDER BY和LIMIT的组合，能够找出一个列中最高或最低的值。 

```SQL
SELECT prod_price FROM products ORDER BY prod_price DESC LIMIT 1; 
```

分析：prod_price DESC保证行是按照由最昂贵到最便宜检索的，而LIMIT 1告诉MySQL仅返回一行。

### 过滤查询数据
```SQL
SELECT 列名1, 列名2 FROM 表名 WHERE 列名 [ = | < | <= | > | >= | <> | != ] 值;  
SELECT 列名1, 列名2 FROM 表名 WHERE 列名 BETWEEN 值1 AND 值2; 
SELECT 列名1, 列名2 FROM 表名 WHERE 列名 IS NULL; 
```

说明：单引号用来限定字符串。如果将值与串类型的列进行比较，则需要限定引号。用来与数值列进行比较的值不用引号。

注意：不匹配不会返回空值（不匹配的前提表示有值，而NULL值根本就没有值，所以不会在有值的范围内，因此需要对无值进行单独处理。）

参考：[过滤数据](http://blog.csdn.net/mediocre117/article/details/53453866)

### 复合的过滤查询数据
```SQL
SELECT 列名1, 列名2 FROM 表名 WHERE 条件1 AND 条件2; 
SELECT 列名1, 列名2 FROM 表名 WHERE 条件1 OR 条件2; 
SELECT 列名1, 列名2 FROM 表名 WHERE 条件1 OR 条件2 AND 条件3; 
SELECT 列名1, 列名2 FROM 表名 WHERE (条件1 OR 条件2) AND 条件3;
SELECT 列名1, 列名2 FROM 表名 WHERE 列名 IN (值1,值2) ORDER BY 列名;
SELECT 列名1, 列名2 FROM 表名 WHERE 列名 NOT IN (值1,值2) ORDER BY 列名;
```

说明：
1. SQL（像多数语言一样）在处理OR操作符前，优先处理AND操作符。
2. IN操作符和OR操作符完成相同功能

IN 操作符的优点：
1.  在使用长的合法选项清单时，IN操作符的语法更清楚且更直观。
2. 在使用IN时，计算的次序更容易管理（因为使用的操作符更少）。
3.  IN操作符一般比OR操作符清单执行更快。
4. IN的最大优点是可以包含其他SELECT语句，使得能够更动态地建立WHERE子句。

NOT操作符： 
对于简单的WHERE子句，使用NOT确实没有什么优势。但在更复杂的子句中，NOT是非常有用的。例如，在与IN操作符联合使用时，NOT使找出与条件列表不匹配的行非常简单。

### 通配符过滤查询数据
```SQL
SELECT 列名1, 列名2 FROM 表名 WHERE 列名 LIKE 'jet%'; 
```

说明：在执行这条子句时，将检索任意以jet起头的词。

```SQL
SELECT 列名1, 列名2 FROM 表名 WHERE 列名 LIKE '_ ton anvil';
```

说明：在执行这条子句时，将检索任意以一个字符开头， ton anvil结尾的词。

通配符：  
%： 代表搜索模式中给定位置的0个、1个或多个字符。  
\_ : 与%能匹配0个字符不一样，_总是匹配一个字符，不能多也不能少。

注意：
1. MySQL的通配符很有用。但是通配符搜索的处理一般要比前面讨论的其他搜索所花时间更长。
2. 根据MySQL的配置方式，搜索可以是区分大小写的。如果区分大小写，'jet%'与JetPack 1000将不匹配。

技巧：  
1. 不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符。
2. 在确实需要使用通配符时，除非绝对有必要，否则不要把它们用在搜索模式的开始处。把通配符置于搜索模式的开始处，搜索起来是最慢的。
3. 仔细注意通配符的位置。如果放错地方，可能不会返回想要的数

---

### 正则表达式
说明：  
1. 当出现复杂的过滤条件时，可以使用正则表达式。  
2. 所有种类的程序设计语言、文本编辑器、操作系统等都支持正则表达式。
3. MySQL仅支持多数正则表达式实现的一个很小的子集。

```SQL
SELECT prod_name FROM products WHERE prod_name REGEXP '.000' ORDER BY prod_name;
```

结果：   
JetPack 1000  
JetPack 2000

分析：
这里使用了正则表达式.000。 .是正则表达式语言中一个特殊的字符。它表示匹配任意一个字符，因此， 1000和2000都匹配
且返回。

LIKE与REGEXP区别：
1. LIKE匹配整个列。如果被匹配的文本在列值中出现，LIKE将不会找到它，相应的行也不被返回（除非使用通配符）。  
2. REGEXP在列值内进行匹配，如果被匹配的文本在列值中出现，REGEXP将会找到它，相应的行将被返回。这是一个非常重要的差别。

注意：
MySQL中的正则表达式匹配（自版本3.23.4后）不区分大小写（即，大写和小写都匹配）。为区分大小写，可使用BINARY关键字，如WHERE prod_name REGEXP BINARY 'JetPack .000'。

1. OR 匹配等价  
'1 | 2'     匹配含1或2的字符串  
'[123]'     匹配含1或2或3的字符串  
'[^123]'    非123其中的字符 

2. 范围匹配  
[1-3] [6-9] [a-z]

3. 匹配特殊字符
'\\\\.'用来匹配.  

说明：  
多数正则表达式实现使用单个反斜杠转义特殊字符，以便能使用这些字符本身。但MySQL要求两个反斜杠（ MySQL自己解释一个，正则表达式库解释另一个）。

匹配多个实例 

符号 | 说明
:---:|:---:
* | 0个或多个匹配 
+ | 1个或多个匹配（等于{1,}）
? | 0个或1个匹配（等于{0,1}）
{n} | 指定数目的匹配
{n,} | 不少于指定数目的匹配

 <br> 
匹配特定位置  

符号 | 说明
:---:|:---:
^ | 文本的开始
+ | 文本的结尾
[[:<:]] | 词的开始
[[:>:]] | 词的结尾  


---

### 创建计算字段（别名）
说明：计算字段并不实际存在于数据库表中。计算字段是运行时在SELECT语句内创建的。
<br>
1. 拼接字段（Concat()）

注意： 
多数DBMS使用+或||来实现拼接，MySQL则使用Concat()函数来实现。当把SQL语句转换成
MySQL语句时一定要把这个区别铭记在心。

删除多余空格  
LTrim()  删除左边空格      
RTrim()  删除右边空格     
Trim()   同时删除左右空格
<br>
2.  AS 别名

实例：
```SQL
SELECT prod_id, quantity, item_price, quantity * item_price 
AS expanded_price FROM orderitems WHERE order_num = 200005;

```

---

### 高级查询
<br>
1. 分组查询  
```SQL
SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*) >= 2;
```
说明：通过GROUP BY关键字来分组，同时将分组后的数据通过HAVING关键字来过滤查询数据。

SELECT子句顺序


子句 | 说明 | 是否必须使用
:---:|:---:|:---:|
SELECT | 要返回的列或表达式 |是
FROM | 从中检索的数据表 |仅在从表选择数据时使用
WHERE | 行级过滤 |否
GROUP BY | 分组说明 |仅在按组计算聚集时使用
HAVING | 组级过滤 |否
ORDER BY | 输出排序顺序 |否
LIMIT | 要检索的行数 |否

<br>
2. 子查询

```SQL
SELECT cust_name, cust_contact FROM customers WHERE cust_id IN 
                (SELECT cust_id FROM orders WHERE order_num IN 
                (SELECT order_num FROM orderitems WHERE prod_id = 'TNT2'));
```
说明：为了执行上述SELECT语句， MySQL实际上必须执行3条SELECT语句。最里边的子查询返回订单号列表，此列表用于其外面的子查询的WHERE子句。外面的子查询返回客户ID列表，此客户ID列表用于最外层查询的WHERE子句。最外层查询确实返回所需的数据。

<br>
3. 联结查询
1) 内部联结
```SQL
SELECT vend_name, prod_name, prod_price 
FROM vendors INNER JOIN products 
ON vendors.vend_id = products.vend_id;
```
说明：这里，两个表之间的关系是FROM子句的组成部分，以INNER JOIN指定。在使用这种语法时，联结条件用特定的ON子句而不是WHERE
子句给出。传递给ON的实际条件与传递给WHERE的相同。（无法检索出含有空的信息）。

2) 外部联结
定义：联结包含了那些在相关表中没有关联行的行。这种类型的联结称为外部联结。
```SQL
SELECT customers.cust_id, orders.order_num FROM customers 
LEFT OUTER JOIN orders ON customers.cust_id = orders.cust_id;
```

说明：在使用OUTER JOIN语法时，必须使用RIGHT或LEFT关键字
指定包括其所有行的表（RIGHT指出的是OUTER JOIN右边的表，而LEFT
指出的是OUTER JOIN左边的表）。

3) 自联结

```SQL
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
AND p2.prod_id = 'DTNTR';
```
说明： 此查询中需要的两个表实际上是相同的表，因此products表在FROM子句中出现了两次。虽然这是完全合法的，但对products的引用具有二义性，因为MySQL不知道你引用的是products表中的哪个实例。（也可以用子查询实现）

4) 自然联结  
定义：自然联结排除多次出现，使每个列只返回一次。
```SQL
SELECT c.*, o.order_num, o.order_date,
oi.prod_id, oi.quantity, OI.item_price
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
AND oi.order_num = o.order_num
AND prod_id = 'FB';
```
说明：在这个例子中，通配符只对第一个表使用。所有其他列明确列出，所以没有重复的列被检索出来。

[高级联结](http://www.2cto.com/database/201604/497286.html)（博客帮助理解）

5) 组合查询
```SQL
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION 
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001, 1002);
```
说明：UNION指示MySQL执行两条SELECT语句，并把输出组合成单个查询结果集。对于更复杂的过滤条件，或者从多个表（而不是单个表）中检索数据的情形，使用UNION可能会使处理更简单。  
注意：返回重复行，可使用UNION ALL而不是UNION。

6) 全文本搜索
1. 性能相比一般通配符，正则表达式效率要高。
2. 可以将搜索出来的结果按照更好的匹配来排列它们，还可以匹配出不包含该词，但是包含其它词的记录。

---
### 插入数据
```SQL
INSERT INTO customers(
    cust_name,
    cust_email,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country)
VALUES(
    'Pep E. LaPew',
    NULL,
    NULL,
    '100 Main Street',
    'Los Angeles',
    'CA',
    '90046',
    'USA');
```
说明：总是使用列的列表一般不要使用没有明确给出列的列表的INSERT语句。使用列的列表能使SQL代码继续发挥作用，即使表结构发生了变化。  

注意：INSERT操作可能很耗时（特别是有很多索引需要更新时），而且它可能降低等待处理的SELECT语句的性能。如果数据检索是最重要的（通常是这样），则可以通过在
INSERT和INTO之间添加关键字LOW_PRIORITY，指示MySQL降低INSERT语句的优先级。这也适合于UPDATE 和 DELETE 语句。

### 更新数据
```SQL
UPDATE customers 
SET cust_email = 'elmer@fudd.com'
WHERE cust_id = 1005;
```
说明：UPDATE语句以WHERE子句结束，它告诉MySQL更新哪一行。没有WHERE子句，MySQL将会用这个电子邮件地址更新customers表中所有行。

注意：
1. 为了删除某个列的值，可设置它为NULL（假如表定义允许NULL值）
2. 在更新多个列时，只需要使用单个SET命令，每个“列=值”对之间用逗号分隔（最后一列之后不用逗号）。

### 删除数据
```SQL
DELETE FROM customers 
WHERE cust_id = 10006;
```
注意：
1. ELETE不需要列名或通配符。DELETE删除整行而不是删除列。为了删除指定的列，请使用UPDATE语句。
2. 如果想从表中删除所有行，不要使用DELETE。可使用TRUNCATETABLE语句，它完成相同的工作，但速度更快（ TRUNCATE实际是删除原来的表并重新创建一个表，而不
是逐行删除表中的数据）。

---
### 表操作
#### 创建表
```SQL
CREATE TABLE customers
(
    cust_id int NOT NULL AUTO_INCREMENT,
    cust_name char(50) NOT NULL,
    cust_address char(50) NULL,
    cust_city char(50) NULL,
    cust_state char(5) NULL,
    cust_zip char(10) NULL,
    cust_country char(50) NULL,
    cust_contact char(50) NULL,
    cust_email char(255) NULL,
    PRIMARY KEY (cust_id)
) ENGINE=InnoDB;
```
注意：在创建新表时，指定的表名必须不存在，否则将出错。如果要防止意外覆盖已有的表，SQL要求首先手工删除该表（请参阅后面的小节），然后再重建它，而不是简单地用创建表语句覆盖它。

#### 更新表
```SQL
ALTER TABLE vendors
ADD vend_phone CHAR(20);
```
说明：这条语句给vendors表增加一个名为vend_phone的列，必须明确其数据类型。

```SQL
ALTER TABLE vendors
DROP COLUMN vend_phone;
```
说明：删除某列。

ALTER TABLE的一种常见用途是定义外键
```SQL
ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_orders
FOREIGN KEY (order_num) REFERENCES orders (order_num);
```
说明：对单个表进行多个更改，可以使用单条ALTER TABLE语句，每个更改用逗号分隔。

#### 删除表
```SQL
DROP TABLE customers2;
```
说明：这条语句删除customers2表（假设它存在）。删除表没有确认，也不能撤销，执行这条语句将永久删除该表。

#### 重命名表
```SQL
RENAME TABLE customers2 TO customers;
```
说明：将表customers2 重命名为customers.

---
### 视图
定义：视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询。

1. 视图用CREATE VIEW语句来创建。
2. 使用SHOW CREATE VIEW viewname；来查看创建视图的语句。
3. 用DROP删除视图，其语法为DROP VIEW viewname;。
4. 更新视图时，可以先用DROP再用CREATE，也可以直接用CREATE OR REPLACE VIEW。如果要更新的视图不存在，则第2条更新语句会创建一个视图；如果要更新的视图存在，则第2条更新语句会替换原有视图。

视图的最常见的应用之一是隐藏复杂的SQL，这通常都会涉及联结。

```SQL
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
AND orderitems.order_num = orders.order_num;
```
说明：这条语句创建一个名为productcustomers的视图，它联结三个表，以返回已订购了任意产品的所有客户的列表。如果执行SELECT * FROM productcustomers，将列出订购了任意产品的客户。

### 存储过程（函数）
定义： 存储过程简单来说，就是为以后的使用而保存的一条或多条MySQL语句的集合。可将其视为批文件，虽然它们的作用不仅限于批处理。
<br>
1. 创建存储过程


1） 简化版本

```SQL
CREATE PROCEDURE productpricing()
BEGIN
    SELECT Avg(prod_price) AS priceaverage
    FROM products;
END;
```
说明: 在MySQL处理这段代码时，它创建一个新的存储过程productpricing。没有返回数据，因为这段代码并未调用存储过程，这里只是为以后使用而创建它。

注意：如果是命令行实用程序要解释存储过程自身内的 ; 字符，它们不会成为存储过程的一部分，会出现语法错误。

可以如下操作(临时更改分隔符）：
```SQL
DELIMITER //

CREATE PROCEDURE productpricing()
BEGIN
    SELECT Avg(prod_price) AS priceaverage
    FROM products;
END //

DELIMITER ;
```
注意： 除\符号外，任何字符都可以用作语句分隔符。

2) 带参数版本

```SQL
CREATE PROCEDURE productpricing(
    OUT p1 DECIMAL(8,2),
    OUT ph DECIMAL(8,2),
    OUT pa DECIMAL(8,2)
)
BEGIN
    SELECT Min(prod_price)
    INTO p1
    FROM products;
    SELECT Max(prod_price)
    INTO ph
    FROM products;
    SELECT Avg(prod_price)
    INTO pa
    FROM products;
END;
```
说明： 此存储过程接受3个参数： pl存储产品最低价格， ph存储产品最高价格， pa存储产品平均价格。每个参数必须具有指定的类型，这里使用十进制值。关键字OUT指出相应的参数用来从存储过程传出一个值（返回给调用者）。MySQL支持IN（传递给存储过程）、 OUT（从存储过程传出，如这里所用）和INOUT（对存储过程传入和传出）类型的参数。存储过程的代码位于BEGIN和END语句内，如前所见，它们是一系列
SELECT语句，用来检索值，然后保存到相应的变量（通过指定INTO关键字）

<br>
2. 删除存储过程
```SQL
DROP PROCEDURE productpricing;
```
注意：
- 这条语句删除刚创建的存储过程。请注意没有使用后面的()，只给出存储过程名。
- 当过程存在想删除它时（如果过程不存在也不产生错误）可使用DROP PROCEDURE IF EXISTS。

<br>
3. 执行存储过程
```SQL
CALL productpricing(@pricelow, @pricehigh, @priceaverage);
```
说明： 由于此存储过程要求3个参数，因此必须正好传递3个参数，不多也不少。所以，这条CALL语句给出3个参数。它们是存储过程将保存结果的3个变量的名字。

注意： 所有MySQL变量都必须以@开始。

### 游标
来源：使用简单的SELECT语句，例如，没有办法得到第一行、下一行或前10行，也不存在每次一行地处理所有行的简单方法（相对于成批地处理它们）。有时，需要在检索出来的行中前进或后退一行或多行。这就是使用游标的原因。

定义：就自己的理解是，游标相当于一个指针，用来灵活的处理从表中查询出来的数据，通常是与存储过程结合起来一起使用的。

创建游标
```SQL
CREATE PROCEDURE productpricing()
BEGIN 
    DECLARE ordernumbers CURSOR
    FOR 
    SELECT order_num FROM orders;
END;
```

打开和关闭游标
```SQL
-- 打开游标
OPEN ordernumbers;

-- 关闭游标
CLOSE rodernumbers;
```
游标使用可参考:  [游标的具体使用详解](http://www.cnblogs.com/mqxs/p/6018766.html)

### 触发器
来源：想要某条语句（或某些语句）在事件发生时自动执行。  
定义：触发器是MySQL响应以下任意语句而自动执行的一条MySQL语句（语句执行之前或者之后）（或位于BEGIN和END语句之间的一组语句）。  
仅支持：  

1. DELETE  
2. INSERT  
3. UPDATE  

其他MySQL语句不支持触发器。


注意： 
1. 在MySQL 5中，触发器名必须在每个表中唯一，但不是在每个数据库中唯一。
2. 现在最好是在数据库范围内使用唯一的触发器名。

```SQL
CREATE TRIGGER newproduct AFTER INSERT ON products
FOR EACH ROW SELECT 'Product added';
```
说明：对每个成功的插入，控制台都会显示Product added信息。

各语句具体使用查阅相关资料。

### 事务管理
定义： 事务处理是一种机制，用来管理必须成批执行的MySQL操作，以保证数据库不包含不完整的操作结果。  

说明：利用事务处理，可以保证一组操作不会中途停止，它们或者作为整体执行，或者完全不执行（除非明确指示）。如果没有错误发生，整组语句提交给（写到）数据库表。如果发生错误，则进行回退（撤销）以恢复数据库到某个已知且安全的状态。

MyISAM引擎不支持事务处理  

InnoDB引擎支持事务处理

基本术语：
1.  事务（transaction）指一组SQL语句；
2. 回退（rollback）指撤销指定SQL语句的过程；
3.  提交（commit）指将未存储的SQL语句结果写入数据库表；
4.  保留点（savepoint）指事务处理中设置的临时占位符（placeholder），你可以对它发布回退（与回退整个事务处理不同）。

```SQL
-- 事务开始
START TRANSACTION
```

```SQL
-- 回退事务
SELECT * FROM ordertotals;
START TRANSACTION;
DELETE FROM ordertotals;
SELECT * FROM ordertotals;
ROLLBACK;
SELECT * FROM ordertotals;
```
说明： 首先执行一条SELECT以显示该表不为空。然后开始一个事务处理，用一条DELETE语句删除ordertotals中的所有行。另一条SELECT语句验证ordertotals确实为空。这时用一条ROLLBACK语句回退START TRANSACTION之后的所有语句，最后一条SELECT语句显示该表不为空。  

注意： 
1. ROLLBACK只能在一个事务处理内使用（在执行一条START TRANSACTION命令之后）。
2. 事务处理用来管理INSERT、 UPDATE和DELETE语句。你不能回退SELECT语句。（这样做也没有什么意义。）你不能回退CREATE或DROP操作。事务处理块中可以使用这两条语句，但如果你执行回退，它们不会被撤销。

```SQL
-- 提交事务
START TRANSACTION;
DELETE FROM orderitems WHERE order_num = 2010;
DELETE FROM orders WHERE order_num = 20010;
COMMIT;
```
说明：在这个例子中，从系统中完全删除订单20010。因为涉及更新
两个数据库表orders和orderItems，所以使用事务处理块来
保证订单不被部分删除。最后的COMMIT语句仅在不出错时写出更改。如
果第一条DELETE起作用，但第二条失败，则DELETE不会提交（实际上，
它是被自动撤销的）。

注意：
1. 一般MySQL语句的使用都是隐含提交的，但是在事务处理中，提交不会隐含进行，需要使用COMMIT语句。
2.  当COMMIT或ROLLBACK语句执行后，事务会自动关闭（将来的更改会隐含提交）。

```SQL
-- 保留点
-- 设置保留点
SAVEPOINT delete1;
-- 事务回退到保留点
ROLLBACK TO delete1;
```
说明：为了支持回退部分事务处理，必须能在事务处理块中合适的位置放置占位符。这样，如果需要回退，可以回退到某个占位符。

注意：保留点在事务处理完成（执行一条ROLLBACK或COMMIT）后自动释放。

```SQL
-- 设置数据库不默认提交
SET autocommit=0;
```
说明：autocommit标志决定是否自动提交更改，不管有没有COMMIT语句。设置autocommit为0（假）指示MySQL不自动提交更改（直到autocommit被设置为真为止）

注意： autocommit标志是针对每个连接而不是服务器的。

---
### 用户管理
```SQL
-- 创建用户账号
CREATE USER ben IDENTIFIED BY '123456';
```
说明：创建用户名为 ‘ben’, 密码为 ‘123456’的用户。

用户的账号信息存储在Mysql.user表中。

```SQL
-- 重命名用户
RENAME USER ben TO bforta;
-- 删除用户账号
DROP USER bforta;
```

```SQL
-- 显示用户的权限信息
SHOW GRANTS FOR bforta;
-- 此GRANT允许用户在crashcourse.*（crashcourse数据库的所有表）上使用SELECT。在所有表中只有只读访问权限。
GRANT SELECT ON crashcourse.* TO bforta;
-- 撤销特定访问权限
REVOKE SELECT ON crashcourse.* FROM bforta;
```

```SQL
-- 修改bforta账户密码
SET PASSWORD FOR bforta = Password('root');
-- 修改登录账户密码
SET PASSWORD = Password('root');
```