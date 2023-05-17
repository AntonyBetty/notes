

# 第二章 查询基础

## 2-1 SELECT语句基础

### 1. 从结构中删除重复行

- DISTINCT 可以**在多列之前**使用。此时，会将多个列的数据进行组合，将重复的数据合并为一条。

```SQL
# 在多列之前使用DISTINCT
SELECT DISTINCT product_type, regist_date
FROM product;
```

- **DISTINCT 关键字只能用在第一个列名之前。**

### 2. 注释的书写方法

注释有两种写法：

- **1行注释。**书写在 “  --  ” 之后，只能写在同一行。

- **多行注释。**书写在 “/*” 和 “\*/” 之间，可以跨多行。



## 2-2 算数运算符和比较运算符

### 1. 需要注意 NULL

- **所有包含NULL 的计算，结果肯定是NULL。**

### 2. 比较运算符

- 在使用大于等于（>=）或者小于等于（<=）作为查询条件时，一定要注意不等号（<、>）和等号（=）的位置不能颠倒。**一定要让不等号在左，等号在右。**如果写成（=<）或者（=>）就会出错。

### 3. 对字符串使用不等号时的注意事项

- **不要混淆了数字和字符串**，也就是说2 和'2' 并不一样。

- **字符串类型的数据原则上按照字典顺序进行排序，不能与数字的大小顺序混淆。**
- 列被定为字符串类型，并且在对字符串类型的数据进行大小比较时，使用的是和数字比较不同的规则。**典型的规则就是按照字典顺序进行比较，也就是像姓名那样，按照条目在字典中出现的顺序来进行排序。**

### 4. 不能对NULL使用比较运算符

- **希望选取NULL记录时，需要在条件表达式中使用IS NULL运算符。希望选取不是NULL的记录时，需要在条件表达式中使用IS NOT NULL运算符。**



## 2-3 逻辑运算符

### 1. 通过括号强化处理

- **AND 运算符优先于OR 运算符。**想要优先执行OR运算符时可以使用括号。

### 2. 逻辑运算符和真值

- 使用**AND 运算符进行的逻辑运算称为逻辑积，使用OR 运算符进行的逻辑运算称为逻辑和。**

### 3. 含有NULL时的真值

- **不确定（UNKNOWN）。**一般的逻辑运算并不存在这第三种值。SQL 之外的语言也基本上只使用真和假这两种真值。与通常的逻辑运算被称为二值逻辑相对，只有SQL 中的逻辑运算被称为三值逻辑。
- **尽量不适用NULL。**考虑NULL 时的条件判断会变得异常复杂，三值逻辑比二值逻辑情况更加复杂。因此，**数据库领域的有识之士们达成了“尽量不使用NULL”的共识。这就是为什么在创建Product 表时要给某些列设置NOT NULL 约束（禁止录入NULL）的缘故。**





# 第三章 聚合与排序

## 3-1 对表进行聚合查询

### 1. 聚合函数

- 所谓**聚合，就是将多行汇总为一行。所有的聚合函数都是这样，输入多行输出一行。**

### 2. 计算NULL之外的数据的行数

- COUNT函数的结果根据参数的不同而不同。**COUNT(*)会得到包含NULL的数据行数，而COUNT(<列名>)会得到NULL之外的数据行数。**

### 3. 计算合计值

- 对于SUM 函数来说，即使包含NULL，也可以计算出合计值。
- 所有的**聚合函数，如果以列名为参数，那么在计算之前就已经把NULL 排除在外了。**因此，无论有多少个NULL 都会被无视。
- **聚合函数会将NULL排除在外。但COUNT(*)例外，并不会排除NULL。**

### 4. 计算平均值

- 计算进货单价**平均值的情况与SUM 函数相同，会事先删除NULL再进行计算。**
- **MAX/MIN函数几乎适用于所有数据类型的列（包括日期、字符串等）。SUM/AVG函数只适用于数值类型的列。**

### 5. 使用聚合函数删除重复值（关键字DISTINCT）

- **想要计算值的种类时，可以在COUNT函数的参数中使用DISTINCT**。
- **DISTINCT 必须写在括号中。**这是因为必须要在**计算行数之前删除product_type 列中的重复数据**。
- 不仅限于COUNT 函数，**所有的聚合函数都可以使用DISTINCT。**
- 在聚合函数的参数中使用DISTINCT，可以删除重复数据。



## 3-2 对表进行分组

### 1. 聚合键中包含NULL的情况

- **聚合键中包含NULL时，在结果中会以“不确定”行（空行）的形式表现出来。**

### 2. 与聚合函数和GROUP BY子句有关的常见错误

#### （1）错误1——在SELECT子句中书写了多余的列

- 实际上，**使用聚合函数时，SELECT 子句中只能存在以下三种元素**：

  - **常数**
  - **聚合函数**
  - **GROUP BY子句中指定的列名（也就是聚合键）**

  【思考】：为什么分组后只能在SELECT子句中书写这三列？因为分组后，由元素的一阶逻辑上升到分组的二阶逻辑了，因此只能问常规的东西（常数）、问组的名称（GROUP BY子句中指定的列名）和组的信息（聚合函数），不能问组内成员的信息（聚合键以外的列名）。

- 这里**经常会出现的错误就是把聚合键之外的列名书写在SELECT 子句之中**。

- **只有MySQL认同这种语法，所以能够执行，不会发生错误（在多列候补中只要有一列满足要求就可以了）**。但是MySQL以外的DBMS都不支持这样的语法，因此请不要使用这样的写法。

- 【注意】：**使用GROUP BY子句时，SELECT子句中不能出现聚合键之外的列名。**

#### （2）错误2——在GROUP BY子句中写了列的别名

- SELECT 子句中的项目可以通过AS 关键字来指定别名。但是，**在GROUP BY 子句中是不能使用别名的。这是因为SELECT 子句在GROUP BY 子句之后执行。在执行GROUP BY 子句时，SELECT 子句中定义的别名，DBMS 还并不知道。**
- 这样的写法在个别DBMS中可用，在其他DBMS 中并不是通用的，因此请大家不要使用。
- 【注意】：**在GROUP BY子句中不能使用SELECT子句中定义的别名。**

#### （3）错误3——GROUP BY子句的结果能排序吗

- **GROUP BY 子句的结果是随机排列的。**当你再次执行同样的SELECT 语句时，得到的结果可能会按照完全不同的顺序进行排列。
- 【注意】：**GROUP BY子句结果的显示是无序的。**

#### （4）错误4——在WHERE子句中使用聚合函数

- 【注意】：**只有SELECT子句和HAVING子句（以及ORDER BY子句）中能够使用聚合函数。**

【思考】：WHERE子句和HAVING子句都有筛选的功能，但是筛选的对象却不同：WHERE子句是陪一阶元素玩的，HAVING子句是陪二阶组玩的。



## 3-3 为聚合结果指定条件

### 1. HAVING子句

- **HERE子句只能指定记录（行）的条件**，而不能用来指定组的条件（例如，“数据行数为2 行”或者“平均值为500”等）。**对集合指定条件就需要使用HAVING 子句。**

### 2. HAVING子句的构成要素

- HAVING 子句中能够使用的3 种要素：
  - **常数**
  -  **聚合函数**
  - **GROUP BY子句中指定的列名（即聚合键）**

【思考】：主要原因是HAVING是陪二阶组玩的，因此HAVING后只能跟组相关的东西，即上述三种。

### 3. 相对于HAVING子句，更适合写在WHERE子句中的条件

- **聚合键所对应的条件**既可以写在HAVING 子句当中，又可以写在WHERE 子句当中。

```SQL
# 情况1：将条件书写在HAVING子句中的情况
SELECT product_type, COUNT(*)
FROM Product
GROUP BY product_type
HAVING product_type = '衣服';

# 情况2：将条件书写在WHERE子句中的情况
SELECT product_type, COUNT(*)
FROM Product
WHERE product_type = '衣服'
GROUP BY product_type;
```

- **聚合键所对应的条件还是应该书写在WHERE 子句之中。**

  - **一是代码容易理解**。**HAVING 子句是用来指定“组”的条件的。因此，“行”所对应的条件还是应该写在WHERE 子句当中。**这样一来，书写出的SELECT 语句不但可以分清两者各自的功能，理解起来也更加容易。
  - **二是执行速度更快**。**先使用WHERE子句筛选元素、再进行分组，使得代码执行速度更快。**且可以对WHERE子句指定条件多对应的列创建**索引**，大幅提高处理速度。

  

  ## 3-4 对查询结果进行排序

### 1. ORDER BY子句

 - 通常，从表中抽取数据时，如果没有特别指定顺序，最终排列顺序便无从得知。**即使是同一条SELECT 语句，每次执行时排列顺序很可能发生改变。**

  ### 2. NULL的顺序

- **排序键中包含NULL时，会在开头或末尾进行汇总**（取决于使用的DBMS）。

### 3. 在排序键中使用显示用的别名

- **在GROUP BY 子句中不能使用SELECT 子句中定义的别名，但是在ORDER BY 子句中却是允许使用别名的**。这是**因为SELECT 子句的执行顺序在GROUP BY 子句之后，ORDER BY 子句之前。**
- SQL语句的执行顺序如下：

```SQL
FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY
```

### 4. ORDER BY子句中可以使用的列

- 在**ORDER BY子句中可以使用SELECT子句中未使用的列和聚合函数。**

### 5. 不要使用列编号

- 列编号是指SELECT 子句中的列按照从左到右的顺序进行排列时所对应的编号（1, 2, 3, …）。

```SQL
# 按照SELECT 子句中第3列的降序和第1列的升序进行排列
SELECT product_id, product_name, sale_price, purchase_price
FROM Product
ORDER BY 3 DESC, 1;
```

- 不使用的列编号的原因有2条：
  - **第一，代码阅读起来比较难。**
  - **第二，该排序功能将来会被删除。**
- 【注意】：**在ORDER BY子句中不要使用列编号。**





# 第四章 数据更新

- 第1-3节分别为 INSERT/DELETE/UPDATE 语句的用法，暂时跳过。

## 4-4 事务

- **事务是需要在同一个处理单元中执行的一系列更新处理的集合。**通过使用事务，可以对数据库中的数据更新处理的提交和取消进行管理。
- **事务处理的终止指令包括COMMIT（ 提交处理）和ROLLBACK（取消处理）两种。**
- 事务具有**ACID特性**。即，DBMS的事务具有**原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持久性（Durability）**四种特性。

### 1. 什么是事务

- 事务是需要**在同一个处理单元中执行的一系列更新处理的集合。**

### 2. 创建事务

- 事务的语法如下：

```SQL
事务开始语句;
# DML语句（INSERT/UPDATE/DELETE）
	DML语句1;
	DML语句2;
	DML语句3;
	......
事务结束语句(COMMIT 或者 ROLLBACK);
```

- **实际上，在标准SQL 中并没有定义事务的开始语句，而是由各个DBMS 自己来定义的。事务结束语句只有COMMIT和ROLLBACK两种，在所有的RDBMS 中都是通用的。**
- MYSQL 的事务开始语句如下：

```SQL
START TRANSACTION;
	......;
COMMIT;
```

#### （1）COMMIT——提交事务

【注意】：COMMIT的流程=直线进行

- **COMMIT 是提交事务包含的全部更新处理的结束指令，相当于文件处理中的覆盖保存。一旦提交，就无法恢复到事务开始前的状态了。**
- 万一由于误操作提交了包含错误更新的事务，就只能回到重新建表、重新插入数据这样繁琐的老路上了。由于可能会造成数据无法恢复的后果，请大家一定要注意（特别是在执行DELETE 语句的COMMIT 时尤其要小心）。

#### （2）ROLLBACK——取消处理

【注意】：ROLLBACK的流程=掉头回到起点

- **ROLLBACK 是取消事务包含的全部更新处理的结束指令，相当于文件处理中的放弃保存。一旦回滚，数据库就会恢复到事务开始之前的状态（回滚到上一个COMMIT状态下）。**
- 通常回滚并不会像提交那样造成大规模的数据损失。

### 3. ACID特性

#### （1）原子性

- 原子性是指在事务结束时，其中所包含的更新处理**要么全部执行，要么完全不执行**，也就是要么占有一切要么一无所有。

#### （2）一致性

- **一致性指的是事务中包含的处理要满足数据库提前设置的约束，如主键约束或者NOT NULL 约束等。**
- 例如，设置了NOT NULL 约束的列是不能更新为NULL 的，试图插入违反主键约束的记录就会出错，无法执行。对事务来说，这些不合法的SQL 会被回滚。也就是说，这些SQL 处理会被取消，不会执行。

#### （3）隔离性

- **隔离性指的是保证不同事务之间互不干扰的特性。该特性保证了事务之间不会互相嵌套。**
- 此外，在某个事务中进行的更改，在该事务结束之前，对其他事务而言是不可见的。因此，即使某个事务向表中添加了记录，在没有提交之前，其他事务也是看不到新添加的记录的。

#### （4）持久性

- **持久性也可以称为耐久性，指的是在事务（不论是提交还是回滚）结束后，DBMS 能够保证该时间点的数据状态会被保存的特性。**即使由于系统故障导致数据丢失，数据库也一定能通过某种手段进行恢复。
- 保证持久性的方法根据实现的不同而不同，其中最常见的就是将事务的执行记录保存到硬盘等存储介质中（该执行记录称为**日志**）。当发生故障时，可以通过日志恢复到故障发生前的状态。





# 第五章 复杂查询

- 第1节为视图，暂时跳过。

## 5-2 子查询

- 一言以蔽之，**子查询就是一次性视图（SELECT语句）。与视图不同，子查询在SELECT语句执行完毕之后就会消失。**
- 由于**子查询需要命名，因此需要根据处理内容来指定恰当的名称。**
- **标量子查询就是只能返回一行一列的子查询。**

### 1. 子查询和视图

- **视图**并不是用来保存数据的，而**是通过保存读取数据的SELECT 语句的方法来为用户提供便利。**
- **子查询就是将用来定义视图的SELECT语句直接用于FROM子句当中。**
- 【注意】：**子查询作为内层查询会首先执行。**

#### （1）增加子查询的层数

- 随着子查询嵌套层数的增加，SQL 语句会变得越来越难读懂，性能也会越来越差。因此，请大家**尽量避免使用多层嵌套的子查询。**

### 2. 子查询的名称

- 原则上**子查询必须设定名称**，因此请大家尽量从处理内容的角度出发为子查询设定恰当的名称。
- 为子查询设定名称时需要使用AS 关键字，该关键字有时也可以省略。

### 3. 标量子查询

#### （1）什么是标量

- **标量子查询则有一个特殊的限制，那就是必须而且只能返回1 行1列的结果，也就是返回表中某一行的某一列的值。**

- **标量子查询就是返回单一值的子查询。**
- 由于返回的是单一的值，因此**标量子查询的返回值可以用在= 或者<> 这样需要单一值的比较运算符之中**。这也正是标量子查询的优势所在。

#### （2）在WHERE子句中使用标量子查询

- 例如：查询出销售单价高于平均销售单价的商品

```mysql
# 错误写法：在WHERE子句中不能使用聚合函数
SELECT product_id, product＿name, sale_price
FROM Product
WHERE sale_price > AVG(sale_price);


# 正确写法：使用标量子查询
SELECT product_id, product_name, sale_price
FROM Product
WHERE sale_price > (SELECT AVG(sale_price)
					FROM Product); # 计算平均销售单价的标量子查询
```

### 4. 标量子查询的书写位置

- **标量子查询的书写位置并不仅仅局限于WHERE 子句中**，通常任何可以使用单一值的位置都可以使用。也就是说，**能够使用常数或者列名的地方，无论是SELECT 子句、GROUP BY 子句、HAVING 子句，还是ORDER BY 子句，几乎所有的地方都可以使用。**
- 【注意】：在SELECT子句中使用标量子查询，如下，**将会增加一列，显示全部商品的平均单价！**

```mysql
SELECT product_id,
        product_name,
        sale_price,
        (SELECT AVG(sale_price)
        FROM Product) AS avg_price
FROM Product;
```

### 5. 使用标量子查询时的注意事项

- **标量子查询绝对不能返回多行结果。**
- 如果子查询返回了多行结果，那么它就不再是标量子查询，而仅仅是一个普通的子查询了，因此**不能被用在= 或者<> 等需要单一输入值的运算符当中，也不能用在SELECT 等子句当中。**



## 5-3 关联子查询

### 1. 普通的子查询和关联子查询的区别

需求：选取出各商品种类中高于该商品种类的平均销售单价的商品。

#### （1）使用关联子查询的解决方案

```mysql
# 通过关联子查询按照商品种类对平均销售单价进行比较
SELECT product_type, product_name, sale_price
FROM Product AS P1 
WHERE sale_price > (SELECT AVG(sale_price)
                    FROM Product AS P2 
                    WHERE P1.product_type = P2.product_type
                    GROUP BY product_type);
```

- 这里起到关键作用的就是**在子查询中添加的WHERE 子句的条件**。该条件的意思就是，在**同一商品种类中**对各商品的销售单价和平均单价进行比较。
- **使用关联子查询时，通常会使用“限定（绑定）”或者“限制”这样的语言**，例如本次示例就是限定“商品种类”对平均单价进行比较。
- 【注意】：在细分的组内进行比较时，需要使用关联子查询。

#### （2）关联子查询也是用来对集合进行划分的

- 我们首先需要计算各个商品种类中商品的平均销售单价，由于该单价会用来和商品表中的各条记录进行比较，因此关联子查询实际只能返回1行结果。这也是关联子查询不出错的关键。

### 2. 结合条件一定更要写在子查询中

- 以下是错误的关联子查询写法：该写法**违反了关联名称的作用域规则**！**关联名称具有一定的有效范围。SQL是按照先内层子查询后外层查询的顺序来执行的。**这样，子查询执行结束时**只会留下执行结果，作为抽出源的P2 表其实已经不存在了。**因此，在执行外层查询时，由于P2 表已经不存在了，因此就会返回“不存在使用该名称的表”这样的错误。

```mysql
# 错误的关联子查询书写方法
SELECT product_type, product_name, sale_price
FROM Product AS P1
WHERE P1.product_type = P2.product_type # 错误写法!!
AND sale_price > (SELECT AVG(sale_price)
                FROM Product AS P2
                GROUP BY product_type);
```





# 第六章 函数、谓词、CASE表达式

## 6-1 各种各样的函数

### 1. 字符串函数

#### （1）CONCAT函数 - 拼接字符串

```MYSQL
SELECT CONCAT('A', 'B', 'C')
FROM DUAL;
```

#### （2）LENGTH函数 - 计算字符串长度（字节数）

- **对1个字符使用LENGTH函数有可能得到2字节以上的结果**（取决于编码格式，比如MYSQL中UTF-8的字符串格式的，一个汉字是3个字符）。

#### （3）CHAR_LENGTH函数 - 计算字符串长度（字符数）

- CHAR_LENGTH函数计算表达式中的字符数，而不是字节数。

#### （4）REPLACE函数 - 替换字符串

```mysql
REPLACE(对象字符串，替换前的字符串，替换后的字符串)
```

```mysql
# 替换字符串的一部分
SELECT str1, str2, str3,
	   REPLACE(str1, str2, str3) AS rep_str
FROM SampleStr;
```

结果如下：

![1](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\1.png)

#### （5）SUBSTRING函数 - 截取字符串

```mysql
SUBSTRING（对象字符串 FROM 截取的起始位置 FOR 截取的字符数）
```

- 使用SUBSTRING 函数可以截取出字符串中的一部分字符串：

```mysql
# 截取字符串的第3位和第4位
SELECT str1,
SUBSTRING(str1 FROM 3 FOR 2) AS sub_str
FROM SampleStr;

# 也可以写成下列方式
SELECT str1,
SUBSTRING(str1, 3, 2) AS sub_str
FROM SampleStr;
```

### 2. 日期函数

#### （1）EXTRACT - 截取日期元素

```MYSQL
EXTRACT(日期元素 FROM 日期)
```

- **使用EXTRACT 函数可以截取出日期数据中的一部分**，例如“年”“月”，或者“小时”“秒”等。

```mysql
SELECT CURRENT_TIMESTAMP,
        EXTRACT(YEAR FROM CURRENT_TIMESTAMP) AS year,
        EXTRACT(MONTH FROM CURRENT_TIMESTAMP) AS month,
        EXTRACT(DAY FROM CURRENT_TIMESTAMP) AS day,
        EXTRACT(HOUR FROM CURRENT_TIMESTAMP) AS hour,
        EXTRACT(MINUTE FROM CURRENT_TIMESTAMP) AS minute,
        EXTRACT(SECOND FROM CURRENT_TIMESTAMP) AS second;
```

![2](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\2.png)

### 3. 转换函数

- 在SQL 中主要有两层意思：

  - 一是**数据类型的转换，简称为类型转换，在英语中称为cast ；**

  - 另一层意思是**值的转换。**

#### （1）CAST - 类型转换

- 进行类型转换需要使用CAST 函数。

```mysql
# 将字符串类型转换为数值类型
SELECT CAST('0001' AS SIGNED INTEGER) AS int_col;

# 将字符串类型转换为日期类型
SELECT CAST('2009-12-14' AS DATE) AS date_col;
```

#### （2）COALESCE - 将NULL转换为其他值

```MYSQL
COALESCE(数据1，数据2，数据3……)
```

- COALESCE 是SQL 特有的函数。该函数会**返回可变参数中左侧开始第1 个不是NULL 的值。**参数个数是可变的，因此可以根据需要无限增加。

```MYSQL
SELECT coalesce(real_name, “xiaoai”)
FROM table_name;
```

- 上述代码中，当real_name 为null值的时候，将返回xiaoai，否则将返回real_name的具体值。

```MYSQL
SELECT coalesce(real_name ,nick_name,“xiaoai”) 
FROM table_name;
```

- 上述代码中，当real_name不为null，那么无论nick_name是否为null，都将返回real_name的具体值；当real_name为null，而nick_name,不为null的时候，返回nick_name具体值。只有当real_name和nick_name都为null的时候，才返回xiaoai。



## 6-2 谓词

- 谓词就是**返回值为真值的函数**。

### 1. 什么是谓词

- 常见的谓词包括以下：
  - LIKE
  - BETWEEN
  - IS NULL, IS NOT NULL
  - IN
  - EXISTS

### 2. LIKE谓词——字符串的部分一致查询

- 我们还可以使用 _（下划线）来代替%，与% 不同的是，它代表了“任意1 个字符”。

### 3. BETWEEN谓词——范围查询

- **BETWEEN包含了临界值。**
- 如果不想让结果中包含临界值，那就必须使用< 和>。

### 4. IS NULL、IS NOT NULL——判断是否为NULL

- 为了选取出某些值为NULL 的列的数据，不能使用=，而只能使用特定的谓词IS NULL。
- 选取NULL 以外的数据时，需要使用IS NOT NULL。

### 5. IN谓词——OR的简便用法

- 使用 IN 谓词“IN（值，......）”来替换OR子句。
- 【注意】：**在使用IN 和NOT IN 时是无法选取出NULL 数据的。**

【思考】：本质是因为IN是OR的替代，而OR与NULL进行计算，得到的还是NULL，因此无法筛选数据。

### 6. EXIST谓词

#### （1）EXIST谓词的使用方法

- **谓词的作用就是“判断是否存在满足某种条件的记录”。如果存在这样的记录就返回真（TRUE），如果不存在就返回假（FALSE）。EXIST（存在）谓词的主语是“记录”。**

#### （2）EXIST的参数

- EXIST 是只有1 个参数的谓词。**EXIST 只需要在右侧书写1 个参数，该参数通常都会是一个子查询。**
- 【注意】：**通常指定关联子查询作为EXIST的参数。**

#### （3）子查询中的SELECT * 

- **EXIST 只关心记录是否存在，因此返回哪些列都没有关系。**
- **把在EXIST 的子查询中书写SELECT * 当作SQL 的一种习惯。**

#### （4）使用NOT EXIST替换NOT IN

- 就像**EXIST 可以用来替换IN 一样，NOT IN 也可以用NOT EXIST来替换。**
- NOT EXIST 与EXIST 相反，当“不存在”满足子查询中指定条件的记录时返回真（TRUE）。



## 6-3 CASE表达式

### 1. CASE表达式的使用方法

- **ELSE 子句也可以省略不写，这时会被默认为ELSE NULL。**虽然CASE表达式中的ELSE子句可以省略，但还是希望大家不要省略。





# 第七章 集合运算

- 本章中我们学习了以下4 个集合运算符。
  - UNION（并集）
  - EXCEPT（差集）
  - INTERSECT（交集）
  - CROSS JOIN（笛卡儿积）

## 7-1 表的加减法

### 1. 表的加法——UNION

- UNION即是集合的并集运算。

- **UNION 等集合运算符通常都会除去重复的记录。**

### 2. 集合运算的注意事项

#### （1）注意事项1——作为运算对象的记录的列数必须相同

- 列数不一致时会发生错误，无法进行加法运算。

#### （2）注意事项2——作为运算对象的记录中列的类型必须一致

- 从左侧开始，相同位置上的列必须是同一数据类型。
- 一定要使用不同数据类型的列时，可以使用类型转换函数CAST。

#### （3）注意事项3——可以使用任何SELECT语句，但ORDER BY子句只能在最后使用一次

- **通过UNION 进行并集运算时可以使用任何形式的SELECT 语句，之前学过的WHERE、GROUP BY、HAVING 等子句都可以使用。但是ORDER BY 只能在最后使用一次。**

```mysql
SELECT product_id, product_name
FROM Product
WHERE product_type = '厨房用具'
UNION
SELECT product_id, product_name
FROM Product2
WHERE product_type = '厨房用具'
ORDER BY product_id; # ORDER BY 只能在最后使用一次
```

### 3. 包含重复行的集合运算——ALL选项

- **在集合运算符中使用ALL选项，可以保留重复行。**

### 4. 选取表中公共部分——INTERSECT

- **INTERSECT 应用于两张表，选取出它们当中的公共记录，即交集。**

### 5. 记录的减法——EXCEPT

- EXCEPT进行减法运算。



## 7-2 联结（以列为单位对表进行联结）

- 联结（JOIN）就是将其他表中的列添加过来，进行“添加列”的集合运算。**UNION是以行（纵向）为单位进行操作，而联结则是以列（横向）为单位进行的。**
- **联结大体上分为内联结和外联结两种。**

### 1. 什么是联结

- **联结（JOIN）运算，简单来说，就是将其他表中的列添加过来，进行“添加列”的运算**。而UNION 和INTERSECT 等集合运算，这些集合运算的特征就是以行方向为单位进行操作。

### 2. 内联结——INNER JOIN

#### （1）内联结要点1——FROM子句

- 进行联结时需要在FROM子句中使用多张表。

#### （2）内联结要点2——ON子句

- **ON 是专门用来指定联结条件的，它能起到与WHERE 相同的作用。需要指定多个键时，同样可以使用AND、OR。**
- 【注意】：**进行内联结时必须使用ON子句，并且要书写在FROM和WHERE之间。**ON 就像是连接河流两岸城镇的桥梁一样。
- 联结条件也可以使用“=”来记述。在语法上，还可以使用<= 和BETWEEN 等谓词。

#### （3）内联结要点3——SELECT子句

- **使用联结时SELECT子句中的列需要按照“<表的别名>.<列名>”的格式进行书写。**虽说只有同时存在于两张表中的列才需要按“< 表的别名>.< 列名>”的格式来书写，否则报错，但都按这种写法去写让代码更加易懂。

#### （4）内联结和WHERE子句结合使用

- **使用联结运算将满足相同规则的表联结起来时，WHERE、GROUP BY、HAVING、ORDER BY 等工具都可以正常使用。我们可以将联结之后的结果想象为新创建出来的一张表，对这张表使用WHERE 子句等工具，这样理解起来就容易多了吧。**

### 3. 外联结——OUTER JOIN

#### （1）外联结要点1——选取出单张表中全部的信息

- **内联结只能选取出同时存在于两张表中的数据；对于外联结来说，只要数据存在于某一张表当中，就能够读取出来。**
- 使用**外联结能够得到固定行数的结果。**

#### （2）外联结要点2——每张表都是主表吗？

- **外联结还有一点非常重要，那就是要把哪张表作为主表。最终的结果中会包含主表内所有的数据。指定主表的关键字是LEFT 和RIGHT。**顾名思义，使用LEFT 时FROM 子句中写在左侧的表是主表，使用RIGHT时右侧的表是主表。
- **外联结中使用LEFT、RIGHT来指定主表。使用二者所得到的结果完全相同（使用LEFT更加容易理解，符合思考逻辑）。**

### 4. 3张以上的表的联结

- 多个INNER JOIN ... ON ... 的重复使用可以实现3张以上的表的联结。即使想要把联结的表增加到4 张、5 张……使用INNER JOIN 进行添加的方式也是完全相同的。

### 5. 交叉联结——CROSS JOIN

- CROSS JOIN 是第三种联结方式（第一种、第二种分别是内联结、外联结）。这种联结在实际业务中并不会使用。
- **对满足相同规则的表进行交叉联结的集合运算符是CROSS JOIN（笛卡儿积）。**
- 交叉联结没有应用到实际业务之中的原因有两个。一是其结果没有实用价值，二是由于其结果行数太多，需要花费大量的运算时间和高性能设备的支持。

### 6. 联结的特定语法和过时语法

- 这种写法是过时的联结写法，尽量避免：

```mysql
# 这是内联结的过时语法
SELECT SP.shop_id, SP.shop_name, SP.product_id, P.product_name, 
P.sale_price
FROM ShopProduct SP, Product P
WHERE SP.product_id = P.product_id # 联结条件
AND SP.shop_id = '000A';
```

- 由于这样的语法不仅过时，而且还存在很多其他的问题，因此不推荐大家使用，理由主要有以下三点。
  - 第一，使用这样的语法**无法马上判断出到底是内联结还是外联结**（又或者是其他种类的联结）。
  - 第二，由于联结条件都写在WHERE 子句之中，因此**无法在短时间内分辨出哪部分是联结条件**，哪部分是用来选取记录的限制条件。
  - 第三，我们不知道这样的语法到底还能使用多久。**每个DBMS 的开发者都会考虑放弃过时的语法**，转而支持新的语法。虽然并不是马上就不能使用了，但那一天总会到来的。





# 第八章 SQL高级处理

## 8-1 窗口函数

- 【总结】：**窗口函数兼具分组（PARTITION BY，不聚合、不减少原表记录行数）和排序（ORDER BY）两种功能。**

- 窗口函数可以进行**排序、生成序列号等一般的聚合函数无法实现的高级操作**。
- 理解PARTITION BY和ORDER BY这两个关键字的含义十分重要。

### 1. 什么是窗口函数

- 窗口函数也称为OLAP 函数（OnLine Analytical Processing），意思是对数据库数据进行**实时分析处理**。

### 2. 窗口函数的语法

```MYSQL
<窗口函数> OVER ([PARTITION BY <列清单>]
ORDER BY <排序用列清单>)
```

- 重要的关键字是**PARTITION BY 和ORDER BY**。

#### （1）能够作为窗口函数使用的函数

- 窗口函数大体可以分为以下两种：
  - **能够作为窗口函数的聚合函数（SUM、AVG、COUNT、MAX、MIN）**。
  - **RANK、DENSE_RANK、ROW_NUMBER 等专用窗口函数**。

### 3. 语法的基本使用方法——使用RANK函数

- RANK 是用来计算记录排序的函数。

```MYSQL
# 根据不同的商品种类，按照销售单价从低到高的顺序创建排序表
SELECT product_name, product_type, sale_price,
	   RANK () OVER (PARTITION BY product_type ORDER BY sale_price) AS ranking
FROM Product;
```

- **PARTITION BY 能够设定排序的对象范围（分组在组内排序）！ORDER BY 能够指定按照哪一列、何种顺序进行排序。**
- 窗口函数中的ORDER BY 与SELECT 语句末尾的ORDER BY 一样，可以通过关键字ASC/DESC 来指定升序和降序。省略该关键字时会默认按照ASC，也就是升序进行排序。
- **PARTITION BY 在横向上对表进行分组，而ORDER BY决定了纵向排序的规则。**
- **窗口函数兼具之前我们学过的GROUP BY 子句的分组功能以及ORDER BY 子句的排序功能。但是，PARTITION BY 子句并不具备
  GROUP BY 子句的汇总功能。因此，使用RANK 函数并不会减少原表中记录的行数。**
- 【注意】：**窗口函数兼具分组（PARTITION BY，不聚合、不减少原表记录行数）和排序（ORDER BY）两种功能。**
- 【注意】：**通过PARTITION BY分组后的记录集合称为“窗口”。**从词语意思的角度考虑，**可能“组”比“窗口”更合适一些**，但是在
  SQL中，“组”更多的是用来特指使用GROUP BY分割后的记录集合，因此，为了**避免混淆**，使用PARTITION BY时称为窗口。

### 4. 无需指定PARTITION BY

- **PARTITION BY 并不是必需的**，即使不指定也可以正常使用窗口函数。这**和使用没有GROUP BY 的聚合函数时的效果一样，也就是将整个表作为一个大的窗口来使用。**

### 5. 专用窗口函数的种类

#### （1）RANK函数

- 计算排序时，如果**存在相同位次的记录，则会跳过之后的位次。**例：有3 条记录排在第1 位时：1 位、1 位、1 位、4 位……

#### （2）DENSE_RANK函数

- 同样是计算排序，**即使存在相同位次的记录，也不会跳过之后的位次**。例：有3 条记录排在第1 位时：1 位、1 位、1 位、2 位……

#### （3）ROW_NUMBER函数

- **赋予唯一的连续位次。**例：有3 条记录排在第1 位时：1 位、2 位、3 位、4 位……

- 由于专用窗口函数无需参数，因此通常括号中都是空的。

### 6. 窗口函数的适用范围

- 原则上**窗口函数只能在SELECT子句中使用。**这类函数不能在WHERE 子句或者GROUP BY 子句中使用。
- **在SELECT 子句之外“使用窗口函数是没有意义的”！**

### 7. 作为窗口函数使用的聚合函数

- 所有的聚合函数都能用作窗口函数，其语法和专用窗口函数完全相同。
- 使用SUM 函数时，并不像RANK 或者ROW_NUMBER 那样括号中的内容为空，需要在括号内指定作为汇总对象的列。
- 在按照时间序列的顺序，计算各个时间的销售额总额等的时候，通常都会使用这种称为累计的统计方法。

### 8. 计算移动平均

- 窗口函数就是将表以窗口为单位进行分割，并在其中进行排序的函数。其实**其中还包含在窗口中指定更加详细的汇总范围的备选功能，该备选功能中的汇总范围称为框架。**

```mysql
# 指定“最靠近的3行”作为汇总对象
SELECT product_id, product_name, sale_price,
        AVG (sale_price) OVER (ORDER BY product_id ROWS 2 PRECEDING) AS moving_avg
FROM Product;
```

#### （1）指定框架（汇总范围）

- 上述代码中我们使用了**ROWS（“行”）**和**PRECEDING（“之前”）**两个关键字，**将框架指定为“截止到之前~ 行”，因此“ROWS 2 PRECEDING”就是将框架指定为“截止到之前2 行”，也就是将作为汇总对象的记录限定为如下的“最靠近的3 行”。**

```MYSQL
ROWS 2 PRECEDING
—— 自身(当前记录) √
—— 之前1行的记录  √
—— 之前2行的记录  √
```

- 也就是说，由于**框架是根据当前记录来确定的，因此和固定的窗口不同**，其范围会随着当前记录的变化而变化。

![3](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\3.png)

- 如果将条件中的数字变为“ROWS 5 PRECEDING”，就是“截止到之前5 行”（最靠近的6 行）的意思。这样的统计方法称为**移动平均（moving average）**。由于这种方法在希望实时把握**“最近状态”**时非常方便，因此常常会应用在对**股市趋势的实时跟踪**当中。
- **使用关键字FOLLOWING（“之后”）替换PRECEDING，就可以指定“截止到之后~ 行”作为框架了！**

![4](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\4.png)

#### （2）将当前记录的前后行作为汇总对象

- **同时使用PRECEDING（“之前”）和FOLLOWING（“之后”）关键字来实现将当前记录的前后行作为汇总对象！**

```MYSQL
# ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
SELECT product_id, product_name, sale_price,
        AVG (sale_price) OVER (ORDER BY product_id
                                ROWS BETWEEN 1 PRECEDING AND 
                                1 FOLLOWING) AS moving_avg
FROM Product;
```

![5](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\5.png)

- 我们通过指定框架，将“1 PRECEDING”（之前1 行）和“1 FOLLOWING”（之后1 行）的区间作为汇总对象。具体来说，就是将如下3 行作为汇总对象来进行计算：

```mysql
——之前1行的记录
——自身（当前记录）
——之后1行的记录
```

![6](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\6.png)

### 9. 两个ORDER BY

- **OVER 子句中的ORDER BY 只是用来决定窗口函数按照什么样的顺序进行计算的，对结果的排列顺序并没有影响。**

```mysql
# 无法保证如下SELECT语句的结果的排列顺序
SELECT product_name, product_type, sale_price,
		RANK () OVER (ORDER BY sale_price) AS ranking
FROM Product;
```

有可能会得到下面这样的结果：

![7](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\7.png)

- 在SELECT 语句的最后，使用ORDER BY子句进行指定，才能让记录切实按照ranking 列的升序进行排列。

```mysql
# 　在语句末尾使用ORDER BY子句对结果进行排序
SELECT product_name, product_type, sale_price,
		RANK () OVER (ORDER BY sale_price) AS ranking
FROM Product
ORDER BY ranking;
```

- SQL语句中最后的ORDER BY 和SELECT 子句中用到的窗口函数的ORDER BY是两个不同的概念，实现了不同的功能。
- 【注意】：**将聚合函数作为窗口函数使用时，会以当前记录为基准来决定汇总对象的记录。**



## 8-2 GROUPING运算符

- **只使用GROUP BY子句和聚合函数是无法同时得出小计和合计的。如果想要同时得到，可以使用GROUPING运算符。**
- 理解GROUPING运算符中CUBE的关键在于形成“积木搭建出的立方体”的印象。

### 1. ROLLUP——同时得出合计和小计

- GROUPING 运算符包括以下三种：
  - ROLLUP
  - CUBE
  - GROUPING SETS

#### （1）ROLLUP的使用方法

```mysql
# 使用ROLLUP同时得出合计和小计
SELECT product_type, SUM(sale_price) AS sum_price
FROM Product
GROUP BY product_type WITH ROLLUP;
```

- WITH ROLLUP 运算符的作用，一言以蔽之，就是“**一次计算出不同聚合键组合的结果**”。例如，上例中就是一次计算出了如下两种组合的汇总结果。
  -  GROUP BY ()
  -  GROUP BY (product_type)

**第一个GROUP BY () 表示没有聚合键，也就相当于没有GROUP BY子句（这时会得到全部数据的合计行的记录）！**

#### （2）将“登记日期”添加到聚合键当中

```mysql
# 　在GROUP BY中添加“登记日期”（不使用ROLLUP）
SELECT product_type, regist_date, SUM(sale_price) AS sum_price
FROM Product
GROUP BY product_type, regist_date;
```

![8](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\8.png)

```MYSQL
# 在GROUP BY中添加“登记日期”（使用ROLLUP）
SELECT product_type, regist_date, SUM(sale_price) AS sum_price
FROM Product
GROUP BY product_type, regist_date WITH ROLLUP;
```

![9](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\9.png)

- 该SELECT 语句的结果相当于使用UNION 对如下3 种模式的聚合级的不同结果进行连接（从小到大！！）。
  -  GROUP BY ()
  - GROUP BY (product_type)
  - GROUP BY (product_type, regist_date)

![10](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\10.png)

- **ROLLUP 是“卷起”的意思，比如卷起百叶窗、窗帘卷，等等。其名称也形象地说明了该操作能够得到像从小计到合计这样，从最小的聚合级开始，聚合单位逐渐扩大的结果。**

### 2. GROUPING函数——让NULL更加容易分辨

![11](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\11.png)

- **GROUPING函数。**该函数在其参数列的值为**超级分组记录所产生的NULL 时返回1，其他情况返回0。**

```MYSQL
# 使用GROUPING函数来判断NULL
SELECT GROUPING(product_type) AS product_type,
	   GROUPING(regist_date) AS regist_date, 
	   SUM(sale_price) AS sum_price
FROM Product
GROUP BY ROLLUP(product_type, regist_date);
```

![12](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\12.png)

- 使用GROUPING 函数还能在超级分组记录的键值中插入字符串。也就是说，当GROUPING 函数的返回值为1 时，指定“合计”或者“小计”等字符串，其他情况返回通常的列的值。

```MYSQL
SELECT CASE WHEN GROUPING(product_type) = 1
		    THEN '商品种类 合计'
			ELSE product_type END AS product_type,
	   CASE WHEN GROUPING(regist_date) = 1
			THEN '登记日期 合计'
			ELSE CAST(regist_date AS VARCHAR(16)) END AS regist_date,
		SUM(sale_price) AS sum_price
FROM Product
GROUP BY ROLLUP(product_type, regist_date);
```

![13](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\13.png)

- 为什么还要将SELECT 子句中的regist_date 列转换为CAST（regist_date AS VARCHAR（16））形式的字符串呢？这是**为了满足CASE 表达式所有分支的返回值必须一致的条件**。如果不这样的话，那么各个分支会分别返回日期类型和字符串类型的值，执行时就会发生语法错误。

```mysql
CAST(regist_date AS VARCHAR(16))
```

- 【注意】：**使用GROUPING函数能够简单地分辨出原始数据中的NULL和超级分组记录中的NULL。**

### 3. CUBE——用数据来搭积木

- MYSQL不支持CUBE语句，暂时跳过。

### 4. GROUPING SETS——取得期望的积木

- MYSQL不支持GROUPING SETS语句，暂时跳过。





# 第九章 通过应用程序连接数据库

## 9-1 数据库世界和应用程序世界的连接

- 在实际的系统中是通过**应用程序**向数据库发送SQL语句的。
- 此时，需要通过**“驱动”**这座桥梁来连接应用程序世界和数据库世界。如果没有驱动，应用程序就无法连接数据库。

### 1. 数据库和应用程序之间的关系

- **系统其实就是由应用和数据库组合而成的**。

![14](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\14.png)

### 2. 驱动——两个世界之间的桥梁

- 驱动就是一个用来连接应用和数据库的非常小的特殊程序（大概只有几百KB）。
- 驱动就是应用和数据库这两个世界之间的桥梁。

![15](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL基础教程\SQL笔记-2022年12月8日一刷\15.png)



## 9-2 Java基础知识

- 暂时跳过。

## 9-3 通过Java连接PostgreSQL

- 暂时跳过。
