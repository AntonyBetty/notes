# 第一章-神奇的SQL

## 1-1 CASE表达式（在SQL里表达条件分支）

### （1）CASE表达式概述

- CASE表达式有简单CASE表达式（CASE sex WHEN '1' THEN '男'）和搜索CASE表达式（CASE WHEN sex = '1' THEN '男'）两种写法。
- 搜索CASE表达式实现的功能更多，以后使用搜索CASE表达式。
- **WHEN子句具有排他性**。当发现真的WHEN子句时，CASE表达式的真假判断就会中止，剩余的WHEN子句会被忽略。
- **注意养成写ELSE子句的习惯**。ELSE子句是可选的，如果不写ELSE子句的话，CASE表达式的执行结果是NULL。但最好明确的协商ELSE子句，这样更易于排查BUG。

### （2）将已有编号方式转换为新的方式并统计

- 需求：以东北、关东、九州等地区为单位来分组，并统计人口数量。

![1](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\1.png)

```sql
SELECT (CASE WHEN pref_name IN ('德岛','香川','爱媛','高知') THEN '四国'
			 WHEN pref_name IN ('福冈','佐贺','长崎') THEN '九州'
             ELSE '其他' 
	    END) AS tag,
        SUM(population)
FROM poptbl
GROUP BY tag;
```

【注意1】：上述代码关键是，将SELECT子句中的CASE表达式复制到GROUP BY子句里。按照新的类别进行分组，GROUP BY 后上升到“组”的二阶状态，此时再用聚合函数SUM统计的是各组的聚合数据！

【注意2】：SQL语句的执行顺序如下：

```
FROM XX
WHERE XX
GROUP BY XX
HAVING XX
SELECT XX
ORDER BY XX
LIMIT XX
```

上述代码中，GROUP BY子句用了SELECT子句中的别名，是因为MYSQL进行了优化，别的SQL数据库并不保证可运行。

### (3)用一条SQL语句进行不同条件的统计

- ​	需求：求按性别、县名汇总的人数。

![2](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\2.png)

```sql
SELECT pref_name,
	   SUM(CASE WHEN sex = '1' THEN population ELSE 0 END) AS '男',
       SUM(CASE WHEN sex = '2' THEN population ELSE 0 END) AS '女'
FROM poptbl2
GROUP BY pref_name;
```

【注意1】：上述代码将“行结构”的数据转换成了“列结构”的数据。除了SUM，COUNT、AVG等聚合函数也都可以用于将行结构的数据转换成列结构的数据。

【注意2】：上述技巧的可贵之处在与，它能将SQL的查询结果**转换为二维表的格式**。

【注意3】：**新手用WHERE子句进行条件分支（用多个WHERE子句来写，再用UNION连接），高手用SELECT子句进行条件分支（使用CASE）**。

### (4)表之间的数据匹配

- 需求：假设这里有一张资格培训学校的课程一览表和一张管理每个月所设课程的表。

![3](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\3.png)

要使用这两张表来生成下面这样的交叉表，以便一目了然地知道每个月开设的课程。

![4](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\4.png)

```sql
#方法1：使用IN谓词（CASE WHEN ... IN ...）
SELECT course_name,
       CASE WHEN course_id IN (SELECT course_id FROM opencourses WHERE month = '200706') THEN 'O' ELSE '×' END AS '6月',
       CASE WHEN course_id IN (SELECT course_id FROM opencourses WHERE month = '200707') THEN 'O' ELSE '×' END AS '7月',
       CASE WHEN course_id IN (SELECT course_id FROM opencourses WHERE month = '200708') THEN 'O' ELSE '×' END AS '8月'
FROM coursemaster;
```

```sql
#方法2：使用EXISTS谓词（CASE WHEN EXISTS ...）
SELECT cm.course_name,
       CASE WHEN EXISTS (SELECT * FROM opencourses op WHERE op.month = '200706' AND op.course_id = cm.course_id) THEN 'O' ELSE '×' END AS '六月',
       CASE WHEN EXISTS (SELECT * FROM opencourses op WHERE op.month = '200707' AND op.course_id = cm.course_id) THEN 'O' ELSE '×' END AS '七月',
       CASE WHEN EXISTS (SELECT * FROM opencourses op WHERE op.month = '200708' AND op.course_id = cm.course_id) THEN 'O' ELSE '×' END AS '八月'
FROM coursemaster cm;
```

【注意】：无论使用IN 还是EXISTS，得到的结果是一样的，但从性能方面来说，EXISTS 更好。通过EXISTS 进行的子查询能够用到“month, course_id”这样的主键索引，因此尤其是当表OpenCourses 里数据比较多的时候更有优势。

### （5）在CASE表达式中使用聚合函数

- 需求：有的学生同时加入了多个社团（如学号为100、200 的学生），有的学生只加入了某一个社团（如学号为300、400、500 的学生）。对于加入了多个社团的学生，我们通过将其“主社团标志”列设置为Y 或者N 来表明哪一个社团是他的主社团；对于只加入了一个社团的学生，我们将其“主社团标志”列设置为N。接下来，我们按照下面的条件查询这张表里的数据。一是获取只加入了一个社团的学生的社团ID。二是获取加入了多个社团的学生的主社团ID。

![5](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\5.png)

```sql
SELECT std_id,
	   CASE WHEN COUNT(club_id) = 1 THEN MAX(club_id) #只加了一个社团的学生；之所以是MAX（club_id），是体现了分组后的二阶概念。
            ELSE MAX(CASE WHEN main_club_flg = 'Y' THEN club_id ELSE NULL END)
            END AS main_club #加了多个社团的学生；之所以将CASE...WHEN...THEN...函数外面用MAX聚合函数包起来，主要是为了消除该分组内部得到的NULL值，取得正常的club_id（防止NULL值代表了整个分组，其实也是分组后二阶状态的影响，仔细体会下）。
FROM studentclub
GROUP BY std_id;
```

【注意1】：新手用HAVING子句进行条件分支，高手用SELECT子句进行条件分支。

【注意2】：通过上述代码可以知道，CASE表达式用在SELECT子句里时，既可以写在聚合函数内部，也可以写在聚合函数外部。

## 1-1 要点总结

- 在GROUP BY子句里使用CASE表达式，可以灵活地选择作为聚合的单位的编号或等级。
- 在聚合函数里使用CASE表达式，可以轻松地将行结构的数据转换为列结构的数据。
- 相反，聚合函数也可以嵌套进CASE表达式里使用。

## 1-1 练习题

- 需求：现有一个表格，默认按‘A-B-C-D'的顺序进行排列。思考一个查询语句，按照'B-A-D-C'这样的指定顺序进行排列。

| key  | X    | Y    | Z    |
| ---- | ---- | ---- | ---- |
| A    | 1    | 2    | 3    |
| B    | 5    | 5    | 2    |
| C    | 4    | 7    | 1    |
| D    | 3    | 3    | 8    |

```sql
#使用“CASE...WHEN...THEN...”语句进行顺序排列
SELECT *
FROM greatests
ORDER BY CASE key1 WHEN 'B' THEN 1
				   WHEN 'A' THEN 2
                   WHEN 'D' THEN 3
                   WHEN 'C' THEN 4
		 END; 
```











---

## 1-2 自连接的用法（面向集合语言SQL）

### （1）写在前面

- 理解自连接，将增进我们对“面向集合”这一SQL语言重要特征的理解。
- 面向集合语言SQL以集合的方式来描述世界。自连接技术充分体现了SQL面向集合的特性。

### （2）可重排列、排列、组合

- 在SQL中，只要表名被赋予了不同的名称，即使是相同的表也应该当作不同的表（集合）来对待。即，相同的表的自连接和不同表间的普通连接并没有什么区别。

```sql
SELECT t1.name, t2.name, t3.name
FROM products t1
CROSS JOIN products t2
CROSS JOIN products t3
WHERE t1.name > t2.name
AND t2.name > t3.name;
```

【注意1】：上述代码将非等值连接（>或<或！=）和自连接结合使用了，称为“非等值自连接”。

【注意2】：“>”和“<”等比较运算符不仅可以用于比较数值大小，**也可用于比较字符串（此时比较的是顺序，见上）**或者日期等。

### （3）排序

- 窗口函数 RANK() 与 DENSE_RANK() 的区别：***rank（）是跳跃排序***，有两个第二名时接下来就是第四名（同样是在各个分组内）。***dense_rank（）***是连续排序，有两个第二名时仍然跟着第三名。

- 需求：按照价格由高到低的顺序，对下面表中的商品进行排序。

![6](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\6.png)

```sql
#使用窗口函数的写法
SELECT name,
	   price,
	   RANK() OVER(ORDER BY price DESC) AS rank_1,
       DENSE_RANK() OVER(ORDER BY price DESC) AS rank_2
FROM products;
```

```sql
#使用非等值自连接的常规写法（跳过写法，类似于RANK函数）
SELECT t1.name,
	   t1.price,
       ((SELECT COUNT(*) FROM products t2 WHERE t2.price > t1.price) + 1) AS rank_1 #比较t2表中有多少条数据价格大于t1表中的本条记录
FROM products t1
ORDER BY rank_1;
```

【注意】：上述使用非等值连接的常规写法很容易扩展，去掉“+1”则可以使排序从0开始排。

```sql
#使用非等值自连接的常规写法（不跳过写法，类似于DENSE_RANK函数）
#由COUNT(*)改为COUNT(DISTINCT t2.price)，本质还是看有多少个不重复的价格比当前价格高。
SELECT t1.name,
       t1.price,
       ((SELECT COUNT(DISTINCT t2.price) FROM products t2 WHERE t2.price > t1.price ) + 1) AS rank_2
FROM products t1
ORDER BY rank_2;
```

【注意1】：如果修改成“ COUNT(DISTINCT t2.price) “，那么存在相同位次的记录时，就可以不跳过之后的位次，而是连续输出（相当于DENSE_RANK函数）。

【注意2】：这道例题很好地体现了面向集合的思维方式。子查询所做的，是计算出价格比自己高的记录的条数并将其作为自己的位次。为了便于理解，我们先考虑从0 开始，对去重之后的4 个价格“ { 100, 80, 50, 30 } ”进行排序的情况。首先是价格最高的100，因为不存在比它高的价格，所以 COUNT 函数返回0。接下来是价格第二高的80，比它高的价格有一个100，所以 COUNT  函数返回1。同样地，价格为50 的时候返回2，为30 的时候返回3。这样，就生成了一个与每个价格对应的集合。

- 上述功能实现也可是使用自连接的写法，逻辑是将两表左连接（条件是 t2.price > t1.price )，然后按 name 分组，数大于本值的个数。

```sql
SELECT t1.name,
	   MAX(t1.price) AS price,
       COUNT(t2.name) + 1 AS rank_1
FROM products t1
LEFT JOIN products t2 #此处之所以使用LEFT JOIN，而不是INNER JOIN，是因为没有比橘子价格还高的商品，因此结果会剔除橘子。用LEFT JOIN即可保留之。
ON t1.price < t2.price 
GROUP BY t1.name
ORDER BY rank_1;
```

- 去掉表格中重复的行，只留下句子、西瓜、葡萄和柠檬这四行，观察出同心圆状的包含关系。从下图可以看出，集合每增大一个，元素也增多1个，，通过数集合里的元素的个数就可以算出位次。

![7](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\7.png)

## 1-2 要点总结

- 与多表之间进行的普通连接相比，自连接的性能开销更大（特别是与非等值连接结合使用的时候），因此**用于自连接的列推荐使用主键或者在相关列上建立索引**。
- 自连接经常和非等值连接（“ > ”、“ < ”、“ <> ”）结合起来使用。
- 自连接和 GROUP BY 结合使用可以生成递归集合。
- 将自连接看做不同的表之间的连接更容易理解。
- **应将表看做行的集合，用面向集合的方法来思考**。
- 自连接的性能开销更大，应尽量**给用于连接的列建立索引**。

## 1-2 练习题

- 需求：将价格由高到底的顺序进行排序，如果出现相同位次，就跳过之后的位次。重点是，给分开后的几个部分排序，而不是对一整张表进行排序。

![8](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\8.png)

```sql
#解法1：使用窗口函数
SELECT district,
	   name,
       price,
       RANK() OVER(PARTITION BY district ORDER BY price DESC) AS rank_1
FROM districtproducts;
```

```sql
#解法2：使用非等值自连接
SELECT t1.district,
	   t1.name,
       MAX(t1.price) AS price,
       COUNT(t2.name) + 1 AS rank_1
FROM districtproducts t1
LEFT JOIN districtproducts t2
ON t1.district = t2.district
AND t1.price  < t2.price
GROUP BY t1.district, t1.name
ORDER BY t1.district, rank_1;
```

```sql
#解法3：关联子查询
SELECT t1.district,
	   t1.name,
       t1.price,
       (SELECT COUNT(t2.price) 
        FROM districtproducts t2 
        WHERE t1.district = t2.district 
        AND t1.price < t2.price) + 1 AS rank_1
FROM districtproducts t1;
```









---

## 1-3 三值逻辑和NULL（SQL的温柔陷阱）

- SQL语言采用一种特别的逻辑体系——三值逻辑，即逻辑真值除了真和假，还有第三个值“不确定”（ unknown ）。
- 之所以SQL的逻辑体系是三值逻辑，原因就在于 NULL 值。**关系型数据库里引进了 NULL ，所以不得不同时引进第三个值**。

### （1）理论篇

#### ①为什么必须写成“ IS NULL "，而不是” = NULL "？

- 因为**对 NULL 使用比较谓词后得到的结果总是 unknown **（这是因为，对非值的 NULL 使用比较谓词本来就是没有意义的）。
- 查询结果只会包含 WHERE 子句里的判断结果为 true 的行，不会包含判断结果为 false 和 unknown 的行。
- NULL不是值，应该把 IS NULL 看做一个谓词。如果可以的话，写成 IS_NULL 这样也许更合适。

#### ②unknown、第三个真值

【注意1】：对于 AND 和 OR ，请注意这三个真值之间有下面这样的优先级顺序。

- AND 的情况： false > unknown > true
- OR 的情况： true > unknown > false

【注意2】：优先级高的真值会决定计算结果。例如，true AND unknown，因为unknown的优先级更高，所以结果是unknown。而 true OR unknown的话，因为true的优先级更高，所以结果是true。

【注意3】：需要特别记住的是，**当AND运算中包含unknown时，结果肯定不是true（反之，如果AND运算结果为true，则参与运算的双方必须都为true）**。

### （2）实践篇

#### ①CASE表达式和NULL

- CASE sex WHEN NULL THEN 1 ——> 这样写是错的，永远不会得到1的结果（因为任何值与NULL进行比较，得到的还是NULL）。
- CASE WHEN sex IS NULL THEN 1 ——> 这样写是正确的。

#### ②NOT IN 和 NOT EXISTS不是等价的

- 如果**NOT IN 子查询中用到的表里被选择的列中存在NULL，则SQL语句整体的查询永远是空**。
- **EXISTS谓词永远不会返回unknown。EXISTS只会返回true或者false**。
- 因此，**IN 和 EXISTS 可以互相替换使用，而 NOT IN 和 NOT EXISTS 却不可以相互替换**。

#### ③限定谓词和极值函数不是等价的

- 极值函数在统计时会把为NULL的数据排除掉。
- 注意ALL 与 极值函数的区别：**极值函数（包括COUNT以外的聚合函数也是如此）在输入为空表（空集）时会返回NULL**。

```sql
#当表b中不存在住在东京的学生，此时使用ALL谓词，返回了a表中所有学生的信息
SELECT *
FROM class_a 
WHERE age < ALL (SELECT age FROM class_b WHERE city = '东京');
```

```sql
#但使用极值函数时，子查询返回NULL值， age < NULL 仍是NULL值，因此返回的是空集
SELECT *
FROM class_a 
WHERE age <  (SELECT MIN(age) FROM class_b WHERE city = '东京');
```

- 比较对象原本就不存在时，根据业务需求有时需要返回所有行，有时需要返回空集。需要返回所有行时（感觉这类似于“不战而胜”），需要使用ALL 谓词。

## 1-3 要点总结

- NULL 不是值。
- 因为 NULL 不是值，所以不能对其使用谓词。
- 对 NULL 使用谓词后的结果是unknown。
- unknown 参与到逻辑运算时，SQL的运行会和预想的不一样。
- 要解决 NULL 带来的各种问题，最佳方法应该是往表里添加 NOT NULL 约束来尽力排除 NULL。这样就可以回到美妙的二值逻辑世界（虽然并不能完全回到）。











---

## 1-4 HAVING子句的力量（出彩的配角）

### （1）写在前面

- HAVING子句是理解**SQL面向集合**这一本质的关键。
- 面向集合语言的第二个特性——以集合为单位进行操作。

### （2）寻找缺失的编号

- 需求：查询下张表中，是否存在数据缺失。

![9](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\9.png)

```sql
SELECT '存在缺失的编号' AS gap
FROM seqtbl
HAVING COUNT(*) != MAX(seq);
```

【注意】：上述使用HAVING子句，比较一个自然数集合中元素的个数与本集合中元素的个数是否一样，以此可以得出是否存在缺失的编号。

### （3）用HAVING子句进行子查询：求众数

- 需求：使用SQL语句从下表中求众数（思路是，将收入相同的毕业生汇总到一个集合里，然后从汇总后的集合的各个集合里找出元素个数最多的集合）。

![10](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\10.png)

```sql
#方法1：使用谓词
SELECT income,
	   COUNT(*) AS cnt
FROM graduates
GROUP BY income
HAVING cnt >= ALL (SELECT COUNT(*) FROM graduates GROUP BY income); #注意是 “>=”。
```

```sql
#方法2：使用极值函数
SELECT income,
	   COUNT(*) AS num
FROM graduates
GROUP BY income
HAVING num >= 
		(SELECT MAX(cnt) AS max_cnt
		FROM
			(SELECT COUNT(*) AS cnt
			FROM graduates
			GROUP BY income) t);
```

### （4）用HAVING子句进行自连接：求中位数

- 需求：求下表中的中位数。思路是，将集合里的元素按照大小分为上半部分和下半部分两个子集，同时让这两个子集共同拥有集合正中间的元素。

![11](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\11.png)

```sql
#如果一个集合同时满足两个条件，即是共同部分元素，再用AVG函数包起来，即求出平均值
SELECT AVG(DISTINCT income)
FROM
	(SELECT t1.income
	FROM graduates t1, graduates t2
	GROUP BY t1.income
	HAVING SUM(CASE WHEN t2.income >= t1.income THEN 1 ELSE 0 END) >= COUNT(*)/2  #上半部分集合
	AND SUM(CASE WHEN t2.income <= t1.income THEN 1 ELSE 0 END) >= COUNT(*)/2) temp_tab; #下半部分集合
```

【注意】：这条SQL 语句的要点在于比较条件“ >= COUNT(\*)/2 ”里的等号，这个等号是有意地加上的。加上等号并不是为了清晰地分开子集S1 和S2，而是为了让这2 个子集拥有共同部分。如果去掉等号，将条件改成“ >COUNT(\*)/2 ”，那么当元素个数为偶数时，S1 和S2 就没有共同的元素了，也就无法求出中位数了。

### （5）查询不包含 NULL 的集合

- **COUNT(*) 可以用于NULL，查询的是所有行的数目**。而**COUNT（列名）与其他聚合函数一样，要先排除掉NULL的行再进行统计**。

## 1-4 要点总结

- SQL通过不断生成子集来求得目标集合。SQL不像面向过程语言那样通过画流程图来思考问题，而是通过画集合的关系图来思考。
- **GROUP BY 子句可以用来生成子集**。
- **WHERE 子句用来调查集合元素的性质，而 HAVING 子句用来调查集合本身的性质**。











---

## 1-5 外连接的用法（SQL的弱点及其趋势和对策）

- 本节部分内容MYSQL不适用，因此跳过。

### （1）用外连接进行行列转换（1）（行——>列）：制作交叉表

- 需求：将下列表格转换为交叉表。

![12](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\12.png)

![13](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\13.png)

```sql
#方法1：使用外连接。通过外连接，不断拓展列的方向
SELECT t1.name,
	   CASE WHEN t2.name IS NOT NULL THEN 'O' ELSE NULL END AS 'SQL入门',
       CASE WHEN t3.name IS NOT NULL THEN 'O' ELSE NULL END AS 'UNIX基础',
       CASE WHEN t4.name IS NOT NULL THEN 'O' ELSE NULL END AS 'Java中级'
FROM (SELECT DISTINCT name FROM courses) t1
LEFT JOIN (SELECT name FROM courses WHERE course = 'SQL入门') t2
ON t1.name = t2.name
LEFT JOIN (SELECT name FROM courses WHERE course = 'UNIX基础') t3
ON t1.name = t3.name
LEFT JOIN (SELECT name FROM courses WHERE course = 'Java中级') t4
ON t1.name = t4.name;
```

```sql
#方法2：使用标量子查询；该方法扩展性较好，需要增加或减少课程时，只修改SELECT子句即可，代码修改起来较为方便。缺点在于，在SELECT子句中使用标量子查询（或者关联子查询）的话，性能开销很大。、
SELECT t1.name,
	   (SELECT 'O' FROM courses t2 WHERE t1.name = t2.name AND t2.course = 'SQL入门') AS 'SQL入门',
       (SELECT 'O' FROM courses t3 WHERE t1.name = t3.name AND t3.course = 'UNIX基础') AS 'UNIX基础',
       (SELECT 'O' FROM courses t4 WHERE t1.name = t4.name AND t4.course = 'Java中级') AS 'Java中级'
FROM (SELECT DISTINCT name FROM courses) t1;
```

```sql
#方法3：嵌套使用CASE表达式；优点是写法简洁、
SELECT name,
	   CASE WHEN SUM(CASE WHEN course = 'SQL入门' THEN 1 ELSE 0 END) = 1 THEN 'O' ELSE NULL END AS 'SQL入门',
       CASE WHEN SUM(CASE WHEN course = 'UNIX基础' THEN 1 ELSE 0 END) = 1 THEN 'O' ELSE NULL END AS 'UNIX基础',
       CASE WHEN SUM(CASE WHEN course = 'Java中级' THEN 1 ELSE 0 END) = 1 THEN 'O' ELSE NULL END AS 'Java中级'
FROM courses
GROUP BY name;
```

## 1-5 要点总结

- 从行数来看，表连接可以看成乘法。因此，当表之间是一对多的关系时，连接后行数不会增加。











## 1-6 用关联子查询比较行与行（用SQL进行行与行之间的比较）

- 关联子查询：如果子查询的执行依赖于外部查询，通常情况下都是因为子查询的表用到了外部的表，并进行了条件关联，因此每执行一次外部查询，子查询都要重新计算一次，这样的子查询就称之为关联子查询。
- 说明：子查询中使用了主查询中的列。

### （1）移动累计值和移动平均值

- 需求：以3次处理为单位求累计值，即移动累计值。所谓移动，指的是将累计的数据行数固定，下例中为3行，一行一行地偏移。

![14](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\14.png)

```sql
#求移动累计值（1）：使用窗口函数
SELECT prc_date,
	   prc_amt,
       SUM(prc_amt) OVER(ORDER BY  prc_date ROWS 2 PRECEDING) AS onhand_amt
FROM accounts;
```

```SQL
#求移动累计值（2）：不满3行的时间区间也输出
#t3.prc_date在以t2.prc_date为起点，以t1.prc_date为终点的区间内移动，
SELECT prc_date,
	   t1.prc_amt,
       (SELECT SUM(prc_amt)
       FROM accounts t2
       WHERE t1.prc_date >= t2.prc_date
       AND (SELECT COUNT(*) FROM accounts t3 WHERE t3.prc_date BETWEEN t2.prc_date AND t1.prc_date) <= 3
       ) AS mvg_sum
FROM accounts t1;
```

```sql
# 移动累计值（3）：不满3行的区间按无效处理
SELECT prc_date,
	   t1.prc_amt,
       (SELECT SUM(prc_amt)
       FROM accounts t2
       WHERE t1.prc_date >= t2.prc_date
       AND (SELECT COUNT(*) FROM accounts t3 WHERE t3.prc_date BETWEEN t2.prc_date AND t1.prc_date) <= 3
       HAVING COUNT(*) = 3  #不满3行数据的不显示
       ) AS mvg_sum
FROM accounts t1;
```

### （2）查询重叠的时间区间

- 需求：查找有时间重叠的人。

![15](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\15.png)

```SQL
# 求重叠的住宿区间
SELECT reserver,
	   start_date,
       end_date
FROM reservations t1
WHERE EXISTS # 如果求“与任何住宿期间都不重叠的日期”，只需将EXISTS改为 NOT EXISTS即可。
	  (SELECT *
      FROM reservations t2
      WHERE t1.reserver != t2.reserver # 与自己之外的客人进行比较；（如果没这个条件，将把所有的人搜出来）
      AND (t1.start_date BETWEEN t2.start_date AND t2.end_date # 要么是自己的入住日期在他人的住宿期间内
      OR t1.end_date BETWEEN t2.start_date AND t2.end_date));# 要么是自己的离店日期在他人的住宿期间内
```

```sql
# 升级版：把完全包含别人的住宿期间的情况也输出（如将山本的入住日期改为11月4日）
SELECT t1.reserver,
	   t1.start_date,
       t1.end_date
FROM reservations t1
WHERE EXISTS
	        (SELECT *
            FROM reservations t2
            WHERE t1.reserver != t2.reserver
            AND ((t1.start_date BETWEEN t2.start_date AND t2.end_date) OR (t1.end_date BETWEEN t2.start_date AND t2.end_date)
				 OR (t2.start_date BETWEEN t1.start_date AND t1.end_date) AND (t2.end_date BETWEEN t1.start_date AND t1.end_date))); # OR后面的代码将“内田这种自己的住宿期间完全包含了他人的住宿期间的情况”也输出了。
```

## 1-6 要点

- 关联子查询的缺点在于，一是代码的可读性不好。二是性能不好。











---

## 1-7 用SQL进行集合运算（SQL和集合论）

- 本节某些内容用到的EXCEPT等在MYSQL中不适用，因此跳过了。

### （1）写在前面

- SQL是面向集合语言。只有从集合的角度来思考，才能明白SQL的强大威力。

### （2）比较表和表：检查集合相等性之基础篇

- 需求：检查两个集合是否相等

```sql
# 原理：UNION操作会去除重复行，如果查询结果与a表或b表的行数一致，说明两张表相等。
SELECT COUNT(*) AS row_cnt
FROM (SELECT * FROM tbl_a UNION SELECT * FROM tbl_b) temp;
```

### （3）寻找相等的子集

- 需求：查询经营的零件在种类数和种类上都完全相同的供应商组合。（这个问题的特点在于比较的对象是集合！）

![16](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\16.png)

```sql
SELECT t1.sup, t2.sup
FROM supparts t1, supparts t2
WHERE t1.sup < t2.sup # 生成供应商的全部组合
AND t1.part = t2.part # 条件1：经营同种类型的零件
GROUP BY t1.sup, t2.sup
HAVING COUNT(*) = (SELECT COUNT(*) FROM supparts t3 WHERE t3.sup = t1.sup) # 条件2：经营的零件种类数相同
AND COUNT(*) = (SELECT COUNT(*) FROM supparts t4 WHERE t4.sup = t2.sup);
```











---

## 1-8 EXISTS谓词的用法（SQL中的谓词逻辑）

- EXISTS是为了实现谓词逻辑中“量化”这一强大功能而被引入SQL的。
- 含义解析：EXISTS的意思是用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回True或Fasle。
- 通俗理解：将外查询表的每一行，代入内查询作为检验，如果内查询返回的结果取非空集，则EXISTS子句返回TRUE，name代入内查询作为检验的这一行数据，便会作为结果显示出来，反之则不显示。

- 需求：

![21](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\21.png)

- 过程分析：

首先，外查询会将a表中的第一行数据（id = 1，name = 李三）代入EXISTS得房内查询中，通过主键关联检测是否有数据关联，如果有则返回True，并**将a表中的第一行返回到前台**。例如，a表中的第一行是（1，张三），它会代入到b表中，通过id = 1 检测到能与b关联（因为b表中作为主键的b_id也有值1），这样就会返回结果True，语句便会将这一行返回给前台，并不会向下继续监测。然后再将a表的第二三、第三行代入b表......如果没有能找到b表b_id关联的，那就会返回False。SQL便不会将表中的这行数据显示出来。

- 上述分析逻辑，NOT EXISTS 与 EXISTS 正好相反。

### （1）理论篇

- 谓词是一种特殊的函数，返回值都是真值。
- EXISTS也是一种特殊的函数，其参数是括号中的内容。
- 在EXISTS的子查询中，“ SELECT + 通配符* ”、“ SELECT + 常量 ” 、“SELECT + 列名 ”，这三类写法得到的结果是一样的。

![17](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\17.png)

- EXISTS的特殊性在于输入值（输出值和其他谓词一样，都是真值）。= 或者 BETWEEN 得房输入值为一行的谓词叫做 ” 一阶谓词“ ，而像 EXISTS 这样输入值为行的集合的谓词叫做”二阶谓词”。
- 全称量词：所有的x都满足条件P。
- 存在量词：存在（至少一个）满足条件P的x。**SQL中的EXISTS谓词实现了谓词逻辑中的存在量词**。
- 在SQL中，为了表达全称量化，需要将 “所有的行都满足条件P” 这样的命题转换成 “不存在不满足条件P的行” 。

### （2）实践篇

#### ①查询表中“不”存在的数据

- 需求：查询没有参加某次会议的人，输出列为meeting和person。

![18](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\18.png)

```SQL

SELECT DISTINCT t1.meeting, t2.person
FROM meetings t1
CROSS JOIN meetings t2 # 通过交叉连接生成 “所有人都参加了全部会议的集合”。
WHERE NOT EXISTS (SELECT * 
				  FROM meetings t3
                  WHERE t1.meeting = t3.meeting
                  AND t2.person = t3.person
                 ); # 通过 NOT EXISTS 返回不满足下述逻辑的集合。
```

- 通过上述SQL语句，NOT EXSITS 直接具备了差集运算的功能。

#### ②全称量化（1）：习惯“肯定 <——> 双重否定”之间的转换（如何使用EXISTS谓词来表达全称量化）

- 习惯从全称量化 ”所有的行都XX“ 到其双重否定 “不XX的行一行都不存在”。

- 需求：从下列表中查询出“所有科目成绩都在50分以上的学生”。

![19](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\19.png)

```sql
# “所有科目分数都在50分以上” ——> 双重否定格式：“没有一个科目分数不满50分”
SELECT student_id
FROM testscores t1
WHERE NOT EXISTS
			(SELECT *
            FROM testscores t2
            WHERE t1.student_id = t2.student_id  #理解此处为什么用的是这个等值连接，而加上 “t1.subject = t2.subject”。这是因为，t1中每个student_id，在t2中只要有一个分数<50分，就不能要。
            AND t2.score < 50)
```

- 需求：请查询满足“①数学的分数在80分以上；②语文的分数在50分以上”两个条件的学生。

- 将上述条件转换为 “某个学生的所有行数据中，如果科目是数学，则分数在80分以上，如果科目是语文，则分数在50根以上”。

```SQL
SELECT DISTINCT student_id
FROM testscores t1
WHERE subject IN ('数学', '语文')
AND NOT EXISTS
		(SELECT *
        FROM testscores t2
        WHERE t2.student_id = t1.student_id # 外查询每传入一个student_id，内查询就会遍历一次。
        AND (CASE WHEN t2.subject = '数学' AND t2.score < 80 THEN 1
				  WHEN t2.subject = '语文' AND t2.score < 50 THEN 1
			 ELSE 0 END) = 1)
```

- 需求：学生必须两门科目都要有成绩。

```SQL
# SELECT子句中去掉 DISTINCT
SELECT  student_id
FROM testscores t1
WHERE subject IN ('数学', '语文')
AND NOT EXISTS
		(SELECT *
        FROM testscores t2
        WHERE t2.student_id = t1.student_id
        AND (CASE WHEN t2.subject = '数学' AND t2.score < 80 THEN 1
				  WHEN t2.subject = '语文' AND t2.score < 50 THEN 1
			 ELSE 0 END) = 1)
GROUP BY student_id # 因为前面WHERE子句中已经对subject进行筛选了，因此此处按ID分组，选数量为2个的组（即同时有语文、数学成绩的）。
HAVING COUNT(*) = 2;
```

## 1-8 要点总结

- SQL中的谓词指的是返回真值的函数。
- EXISTS和其他的谓词不一样，**接受的参数是集合**。
- 因此EXISTS可以看成是一种高阶函数。
- SQL中没有与全称量词相当的谓词，可以使用NOT EXISTS代替。

## 1-8 练习题

- 需求：从表中查询出val全是1的key。因为要按“行方向”进行全称量化，因此要使用EXISTS谓词。

```SQL
# 选出val全是1的key。等价于：没有一个val不是1 + 没有一个val是空值。
# 此题注意NULL值的处理
SELECT DISTINCT key1
FROM arraytbl2 t1
WHERE NOT EXISTS
		  (SELECT *
          FROM arraytbl2 t2
          WHERE t2.key1 = t1.key1
          AND (t2.val != 1 OR t2.val IS NULL)); # 内部子查询中，只要有一个非1或NULL值，就不满足！换言之，全是1的情况下才会选出外部传递的数据。
```

```sql
# 其他方法：使用HAVING子句
SELECT key1
FROM arraytbl2
GROUP BY key1
HAVING SUM(CASE WHEN val =1 THEN 1 ELSE 0 END) = 10
```

```SQL
# 其他方法：使用极值函数。
#（逻辑：如果一个组的最大值、最小值都是1，那么这个组的数全部是1。）
SELECT key1
FROM arraytbl2
GROUP BY key1
HAVING MIN(val) = 1
AND MAX(val) = 1;
```











---

## 1-9 用SQL处理数列（灵活使用谓词逻辑）

## （1）生成连续编号

- 需求：生成0到99的的集合（现有表格为digit，包含了0~9的数字）。

```sql
# 通过两个集合求笛卡尔积得出0~99的数字；
# 笛卡尔积：得到所有可能的组合。
# 下述的处理逻辑是：仅把数看成是数字的组合。
SELECT t1.digit + t2.digit * 10 AS number
FROM digits t1
CROSS JOIN digits t2
ORDER BY number;
```

```sql
# 升级：通过求笛卡尔积的数量、过滤条件求1~666的数字
SELECT (t1.digit + t2.digit * 10 + t3.digit * 100) AS number
FROM digits t1
CROSS JOIN digits t2
CROSS JOIN digits t3
WHERE (t1.digit + t2.digit * 10 + t3.digit * 100) BETWEEN 0 AND 666
ORDER BY number ;
```

### （2）求全部的缺失编号

- 需求：有下面的表，范围是1到12，查询出缺失值，即3、9、10。

![22](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\22.png)

```sql
# 前置步骤：创建一个视图，该视图可取值为0到999
CREATE VIEW Sequence(number)
AS (SELECT (t1.digit + t2.digit * 10 + t3.digit * 100)
FROM digits t1
CROSS JOIN digits t2
CROSS JOIN digits t3);
```

```SQL
#  方法1：使用NOT IN。逻辑是，生成1到12的集合。查询没有在该表seqtbl中出现的数。
SELECT number
FROM sequence
WHERE number BETWEEN (SELECT MIN(seq) FROM seqtbl)
			 AND (SELECT MAX(seq) FROM seqtbl)
AND number NOT IN (SELECT seq FROM seqtbl)
ORDER BY number;
```

### （3）三个人坐得下吗

- 需求：找出连续3个空位的全部组合。

![23](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\23.png)

```SQL
# 关键逻辑：本例中需要满足的条件是：“所有座位的状态都是 '未预定' ”。
# “所有行都满足条件P” ——> “不存在不满足条件P的行”
# NOT EXISTS  （... != ...）
SELECT t1.seat AS start_seat,
	   '~',
       t2.seat AS end_seat
FROM seats t1
INNER JOIN seats t2
ON t2.seat = t1.seat + 3 - 1 # 此处这个3，表示筛选出所有连续3个座位的组合
WHERE NOT EXISTS
				(SELECT *
                FROM seats t3
                WHERE t3.seat BETWEEN t1.seat AND t2.seat
                AND t3.status != '未预订'); #内查询，只要有一个满足True，就不往下查了，内查询就返回True，外查询不选数据。
```

```SQL
# 使用HAVING子句解决
SELECT t1.seat AS start_seat,
	   '~',
       t2.seat AS end_seat
FROM seats t1
INNER JOIN seats t2
ON t2.seat = t1.seat + 3 - 1 # 此处这个3，表示筛选出所有连续3个座位的组合
INNER JOIN seats t3 # 此处连接表t3，展示middle_seat，其个数再用SUM（CASE WHEN THEN）函数进行对比，使用HAVING筛选出合适的集合
ON t3.seat BETWEEN t1.seat AND t2.seat
GROUP BY t1.seat,  t2.seat
HAVING COUNT(*) = SUM(CASE WHEN t3.status = '未预订' THEN 1 ELSE 0 END);
```

### （4）单调递增和单调递减

![24](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\24.png)

```SQL
# 求单调递增的区间的SQL语句：子集也输出
# 这段代码的关键在于，通过分组，来最大限度的眼神起点和终点
SELECT MIN(start_date) AS start_date, # 最大限度往前延伸
	   end_date
FROM
	(SELECT t1.deal_date AS start_date,
			MAX(t2.deal_date) AS end_date # 最大限度往后延伸
	FROM mystock t1
	INNER JOIN mystock t2
	ON t1.deal_date < t2.deal_date # 第一步：生成起点和终点的组合
	AND NOT EXISTS
			(SELECT * # 第二步：描述区间内所有日期需要满足的条件
			FROM mystock t3, mystock t4
			WHERE t3.deal_date BETWEEN t1.deal_date AND t2.deal_date
			AND t4.deal_date BETWEEN t1.deal_date AND t2.deal_date
			AND t4.deal_date > t3.deal_date
			AND t3.price >= t4.price
			)
	GROUP BY t1.deal_date) temp
GROUP BY end_date;
```

## 1-9 要点总结

- 要在SQL中表达全称量化时，需要将全称量化命题转换成存在量化命题的否定形式，并使用 NOT EXISTS 谓词。这是因为SQL只实现了谓词逻辑中的存在量词。











---

## 1-10 HAVING子句又回来了（再也不要叫它配角了！）

### （1）写在前面

- 学习HAVING子句的用法是帮助我们顺利地忘掉面向过程语言的思考方式并理解SQL面向集合特性的最为有效的方法。
- HAVING子句的处理对象是集合而不是记录。

### （2）各位，全体点名

- 需求：查询出可以出勤的队伍（全部成员都是待命状态）

![25](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\25.png)

```SQL
# 方法1：使用HAIVNG子句
SELECT team_id
FROM teams
GROUP BY team_id
HAVING COUNT(*) = SUM(CASE WHEN status = '待命' THEN 1 ELSE 0 END);
```

```SQl
# 方法2：使用NOT EXISTS
# “所有队员都处于待命状态” ——> 没有一个不待命
SELECT team_id
FROM teams t1 
WHERE NOT EXISTS   # 没有一个
		  (SELECT *
          FROM teams t2
          WHERE t1.team_id = t2.team_id
          AND t2.status <> '待命'); #不待命
```

- 使用NOT EXISTS谓词的好处是，性能好，而且结果中能体现出队员信息。
- 因为使用GROUP BY 和 HAVING子句将元素上升到组的阶层了，不再体现集合的元素的特征，最低也是集合的特征，因此无法体现出队员信息。

```sql
# 方法3：使用HAVING子句和极值函数 （如果一个组的最大数、最小数都是1，那么这个组的数都是1）
SELECT team_id
FROM teams
GROUP BY team_id
HAVING MIN(status) = '待命'
AND MAX(status) = '待命';
```

## 1-10 要点总结

- 如果实体对应的是表中的一行数据，那么该实体应该被看作集合中的元素，因此指定的查询条件应该使用 WHERE 子句（一阶状态）。
- 如果实体对应的是表中的多行数据，那么该实体应该被看做是集合，因此指定查询条件时应该使用 HAVING 子句。
- 总结：用于调查集合性质的常用条件及其用途

![26](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\26.png)

- 在SQL中指定搜索条件时，最重要的是搞清楚搜索的实体是集合还是集合的元素。
- 如果一个实体对应着一行数据 ——> 那就是元素，使用 WHERE 子句。
- 如果一个实体对应着多行数据 ——> 那就是集合，使用 HAVING 子句。
- 如果通过 CASE 表达式生成特征函数，那么无论多么复杂的条件都可以描述。











---

## 1-11 让SQL飞起来（简单的性能优化）

### （1）使用高效的查询

#### ①参数是子查询时，使用 EXISTS 代替 IN

- 大多数时候， [NOT] IN 和 [NOT] EXISTS 返回的结果是相同的。当二者用于子查询时，EXISTS 的速度会更快一些。 原因是：**一是如果连接列（id）上建立了索引，那么查询class_B时不用查实际的表，只需查索引就可以了。二是如果使用EXISTS，那么只要查到一行数据满足条件就会终止查询，不用像使用 IN 时一样扫描全表。在这一点上 NOT EXISTS 也一样。**

  当 IN 的参数是子查询时，数据库首先会执行子查询，然后将结果存储在一张临时的工作表里（内联视图），然后扫描整个视图。很多情况下这种做法都非常耗费资源。使用EXISTS 的话，数据库不会生成临时的工作表。

#### ②参数是子查询时，使用连接代替IN

- **想改善 IN 的性能，除了使用 EXISTS，还可以使用连接（INNER JOIN）**。这种写法至少能用到一张表的“id”列上的索引。而且，因为没有了子查询，所以数据库也不会生成中间表。

### （2）避免排序

- 会进行排序的代表性的运算有下面这些。**尽量避免（或减少）无谓的排序是我们的目标**。

![27](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\27.png)

### （3）在极值函数中使用索引（MAX/MIN）

-  SQL 语言里有MAX 和MIN 两个极值函数。**使用这两个函数时都会进行排序。但是如果参数字段上建有索引，则只需要扫描索引，不需要扫描整张表。**

### （4）能写在WHERE子句里的条件不要写在HAVING子句

- 从性能上来看，能用 WHERE 解决的事就不要用HAVING解决。原因，**第一个是在使用 GROUP BY 子句聚合时会进行排序，如果事先通过WHERE 子句筛选出一部分行，就能够减轻排序的负担。第二个是在WHERE 子句的条件里可以使用索引。HAVING 子句是针对聚合后生成的视图进行筛选的，但是很多时候聚合后的视图都没有继承原表的索引结构。**

### （5）在 GROUP BY 子句和 ORDER BY 子句中使用索引

- 通过指定带索引的列作为 GROUP BY 和 ORDER BY 的列，可以实现告诉查询。

### （6）真的用到了索引了吗?

#### ①使用索引时，**条件表达式的左侧应该是原始字段**。

```sql
# 1、在索引字段上计算，以下不会用到索引。
SELECT *
FROM SomeTable
WHERE col_1 * 1.1 > 100;

# 把运算的表达式放到查询条件的右侧，就能用到索引了
WHERE col_1 > 100 / 1.1;
```

```sql
# 2、在查询条件的左侧使用函数时，也不能用到索引。
SELECT *
FROM SomeTable
WHERE SUBSTR(col_1, 1, 1) = 'a';
```

#### ②**指定 IS NULL 和 IS NOT NULL 的话会使得索引无法使用，进而导致查询性能低下。**通常，索引字段是不存在 NULL 的。

```SQL
# 使用 IS NULL 将无法使用索引
SELECT *
FROM SomeTable
WHERE col_1 IS NULL;
```

#### ③使用否定形式，下面几种否定形式不能用到索引。

![28](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\28.png)

```sql
# 下面的SQL语句会进行全表扫描
SELECT *
FROM SomeTable
WHERE col_1 <> 100;
```

#### ④使用 OR

- 在col_1 和col_2 上分别建立了不同的索引，或者建立了（col_1,col_2）这样的联合索引时，**如果使用OR 连接条件，那么要么用不到索引，要么用到了但是效率比AND 要差很多。**

#### ⑤使用联合索引时，列的顺序错误

- 假设存在这样顺序的一个联合索引“col_1, col_2, col_3”。这时，指定条件的顺序就很重要。

![29](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\29.png)

- **联合索引中的第一列（col_1）必须写在查询条件的开头，而且索引中列的顺序不能颠倒**。

#### ⑥使用 LIKE 谓词进行后方一直或中间一致的匹配

- 使用LIKE 谓词时，只有前方一致的匹配才能用到索引。

![30](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\30.png)

#### ⑦进行默认的类型转换（如，对 char 类型的‘col_1’列指定条件的示例）

- **默认的类型转换不仅会增加额外的性能开销，还会导致索引不可用，可以说是有百害而无一利。虽然这样写还不至于出错，但还是不要嫌麻烦，在需要类型转换时显式地进行类型转换吧**（别忘了转换要写在条件表达式的右边）。

![31](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\31.png)

### （7）减少中间表

#### ①灵活使用 HAVING 子句

- 对聚合结果指定筛选条件时，使用HAVING 子句是基本原则。不习惯使用HAVING 子句的数据库工程师可能会倾向于像下面这样先生成一张中间表，然后在WHERE 子句中指定筛选条件。
- 然而，对聚合结果指定筛选条件时不需要专门生成中间表，像下面这样使用HAVING 子句就可以。**HAVING 子句和聚合操作是同时执行的，所以比起生成中间表后再执行的WHERE 子句，效率会更高一些，而且代码看起来也更简洁。**

#### ②需要对多个字段使用IN 谓词时，将它们汇总到一处

```SQL
SELECT *
FROM Addresses1 A1
WHERE (id, state, city)
IN (SELECT id, state, city
FROM Addresses2 A2);
```

## 1-11 要点总结

- 所谓的SQL优化：**不管是减少排序还是使用索引，抑或是避免中间表的使用，都是为了减少对硬盘的访问**。这是提高查询速度的关键！
- **参数是子查询时，使用 EXISTS 或者连接代替 IN。**
- **使用索引时，条件表达式的左侧应该是原始字段（不要涉及计算，如 col * 1.1 等）。**
- **在 SQL 中排序无法显式地指定，但是请注意很多运算都会暗中进行排序。**
- **尽量减少没用的中间表。**











---

## 1-12 SQL编程方法（确立SQL的编程风格）

- ***代码要清晰，不要为了“效率”牺牲可读性。***
- SQL编写的不成文的规定：**关键字使用大写字母，列名和表名使用小写字母**。
- **不适用通配符 “  * ”，指定需要筛选的哪些列写在SELECT中。**
- **尽量使用 LEFT JOIN 。**左连接有一个优势：一般情况下表头都出现在左边（笔者没遇见过表头出现在右边的情况）。使用左边的表作为主表的话，SQL 就能和执行结果在格式上保持一致。
- **从FROM子句开始写。**SQL 中各部分的执行顺序是：**FROM → WHERE → GROUP BY → HAVING →SELECT( → ORDER BY)。严格地说，ORDER BY 并不是SQL 语句的一部分，因此可以排除在外。这样一来，SELECT 就是最后才被执行的部分了。因此，如果需要写很复杂的SQL 语句，可以考虑按照执行顺序从FROM 子句开始写，这样添加逻辑时更加自然。**
- 编写程序永远要考虑程序的可读性。











***

***

# 第二章-关系数据库的世界

## 2-5 GROUP BY和PARTITION BY（物以”类“聚）

- **SQL 的语句中具有分组功能的是GROUP BY 和PARTITION BY，它们都可以根据指定的列为表分组。区别仅仅在于，GROUP BY 在分组之后会把每个分组聚合成一行数据。**

- **GROUP BY 和 PARTITION BY 都是用来划分团队成员的函数。**

```SQL
# 窗口函数涉及 PARTION BY
SELECT member,
	   team,
       age,
       RANK() OVER(PARTITION BY team ORDER BY age DESC) AS rn,
       DENSE_RANK() OVER(PARTITION BY team ORDER BY age DESC) AS dense_rn,
       ROW_NUMBER() OVER(PARTITION BY team ORDER BY age DESC) AS row_num
FROM teams
```

- 窗口函数类似于聚合函数，但又不同于聚合函数。聚合函数是将组内多个数据聚合成一个值，而**窗口函数除了可以将组内数据聚合成一个值，还可以保留原始的每条数据。**











---

## 2-6 从面向过程思维向声明式思维、面向集合思维转变的7个关键点（画圆）
- 学习SQL 的思维方式时，最大的阻碍就是我们已经习惯了的面向过程语言的思维方式。具体地说，就是以赋值、条件分支、循环等作为基本处理单元，并将系统整体分割成很多这样的单元的思维方式。
- SQL 的思维方式，从某种意义上来说刚好相反。SQL 中没有赋值或者循环的处理，数据也不以记录为单位进行处理，而以集合为单位进行处理。
- 如果硬要以面向过程的方式写SQL 语句，写出的SQL 语句要么长且复杂，可读性不好，要么大量借助于存储过程和游标，这样就又会回到已经习惯的面向过程的世界。

### （1）用 CASE 表达式代替 IF 语句和 CASE 语句。 SQL 更像一种函数式语言

### （2）用 GROUP BY 和关联子查询代替循环

- SQL 中没有专门的循环语句。虽然可以使用游标实现循环，但是这样的话还是面向过程的做法，和纯粹的SQL 没有关系。
- **面向过程语言在循环时经常用到的处理是“控制、中断”。在SQL 中，这两个处理可以分别用GROUP BY 子句和关联子查询来表达。其中，对于SQL 语言的初学者来说，关联子查询可能不太好理解，但是这个技术非常适合用来分割处理单元。**面向过程语言中使用的循环处理完全可以用这两个技术代替。
- SQL 中没有循环，而且没有也并不会带来什么问题。

### （3）表中的行没有顺序

- **在关系数据库中，读出的数据不一定是按照INSERT 的顺序排列的，因为SQL 在处理数据时不需要它们这样。**SQL 在处理数据时可以完全不依赖顺序。
- **关系数据库中的表（关系）是一种数学上的“集合”。表有意地放弃了行的顺序这一形象的概念，从而使它具有了更高的抽象度。**

### （4）将表看成集合

- 虽然我们很容易把表看成与文件一样的东西，但是实际上，一张表并非对应一个文件，读取表时也并不是像读取文件一样一行一行地进行的。
- **理解表的抽象性的最好的方法是使用自连接。原因很显然，自连接本身就是基于集合这一高度抽象（也可以说成自由）的概念的技术。**
- **在SQL语句中，我们给同一张表赋予不同的名称后，就可以把这两张表当成不同的表来处理。也就是说，通过自连接，我们可以添加任意数量的集合来处理。**

### （5）理解 EXISTS 谓词和“量化”的概念

- 对于SQL 中不具备的全称量化符，我们只能通过在程序中使用NOT EXISTS 来表达。
- 使用NOT EXISTS 的查询语句，可读性都不太好。而且，因为同样的功能也可以用HAVING 子句或者ALL 谓词来实现，所以很多程序
  员都不太愿意使用它。但是，**NOT EXISTS 有一个很大的优点，即性能比HAVING 子句和ALL 谓词要好得多。**

### （6）学习 HAVING 子句的真正价值

- **HAVING 子句是集中体现了SQL之面向集合理念的功能。多年以来，笔者一直认为掌握SQL 的思维方式的最有效的捷径就是学习HAVING 子句的用法。**
- **与WHERE 子句不同，HAVING 子句正是设置针对集合的条件的地方，因此为了灵活运用它，我们必须学会从集合的角度来理解数据。**通过练习HAVING 子句的用法，我们会在不经意间加深对面向集合这个本质的理解，真是一举两得。

### （7）不要画长方形，去画圆

- **目前，能够准确描述静态数据模型的标准工具是维恩图，即“圆”。通过在维恩图中画嵌套子集，可以很大程度地加深对SQL 的理解。**这是因为，嵌套子集的用法是SQL 中非常重要的技巧之一。例如，GROUP BY或者PARTITION BY 将表分割成一些称为“类”的子集A，以及冯·诺依曼型递归集合、用来处理树结构的嵌套子集模型，都是子集的代表性应用。











---

## 2-9 消灭NULL委员会（全世界的数据库工程师团结起来！）

- **总的逻辑：NULL 是在无法设置默认值情况下的妥协。**

——**①首先分析能不能设置默认值。**

——**②仅在无论如何都无法设置默认值时允许使用NULL。**

### （1）为什么 NULL 如此惹人讨厌

- **在进行 SQL 编码时，必须考虑违反人类直觉的三值逻辑（True、Fasle、Unknown）。**
- **在指定 IS NULL 、 IS NOT NULL 的时候，不会用到索引**，因而 SQL 语句执行起来性能很低。
- 如果四则运算以及 SQL 函数的参数中包含 NULL， 会引起 “ NULL 的传播”。

![33](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\33.png)

- 在接收 SQL 查询结果的宿主语言中， NULL 的处理方法没有统一标准。
- 与一般列的值不同，NULL 是通过在数据行的某处加上多余的位（bit）来实现的。因此NULL 会使程序占据更多的存储空间，使得检索性能变差。

- 对 NULL 的认识：**我们可以把NULL 视为一种药品。适当使用可能有益，但滥用会导致毁灭。最好的策略是尽可能地避免适用NULL，并在不得不使用时适当使用。**

### （1）消除NULL的方法：编号——使用异常编号

- 例如ISO 的性别编号中，除了“1: 男性”“2: 女性”，还定义了“0: 未知”“9: 不适用”这两个用于异常情况的编号。编号9 可用于法人的情况。这真是一种很棒的解决方案，无意间刚好与由Codd 区分的两类NULL“未知”和“不适用”相吻合了A。
- 编号列应该使用字符串类型。

### （2）消除NULL的方法：名字——使用“无名氏”

- **在使用名字的时候，处理方法和编号是一样的。也就是说，赋予表示未知的值就可以了。不论是“未知”还是“UNKNOWN”，**只要是在开发团队内部达成一致的适当的名字就行。

### （3）消除NULL的方法：数值——用0代替

- 对于数值型的列，笔者认为最好的方法是一开始就将NULL 转换为0再存储到数据库中。
- 因此更加可行的方案是下面这样的方案：**①转换为0。②如果一定要区分0 和NULL，那么允许使用NULL。**如果能转换为0，希望大家还是尽量把NULL 转换为0。

### （4）消除NULL的方法：日期——用最大值或最小值代替

- **当需要表示开始日期和结束日期这样的“期限”的时候，我们可以使用0000-01-01 或者9999-12-31 这样可能存在的最大值或最小值来处理。**例如表示员工的入职日期或者信用卡的有效期的时候，就可以这样处理。这种方法一直都被广泛使用着。
- 相反，当默认值原本就不清楚的时候，例如历史事件发生的日期，或者某人的生日等，也就是当NULL 的含义是“未知”的时候，我们不能像前面那样设置一个有意义的默认值。这时可以允许使用NULL。











---

## 2-10 SQL中的层级（严格的等级社会）

- **在SQL中，使用 GROUP BY 聚合之后，我们就不能引用原表中除聚合键以外的列。**其实这只是SQL中的一种逻辑，是为了严格区分层级。

### （1）为什么聚合后不能再引用原表中的列

- 标准SQL规定，**在对表进行聚合查询的时候，只能在 SELECT 子句里写以下三种内容：①通过 GROUP BY 子句指定的聚合键。②聚合函数（SUM、AVG等）。③常量。**
- **使用 GROUP BY 聚合之后，SQL的操作对象便由0阶的“行”变成了1阶的“行的集合”。此时，行的属性便不能使用了。**SQL的世界其实是层级分明的等级社会。

- 需求：从下表中查询出“小组中年龄最大的成员”。这是一个特殊的形式，**查询的结果中同时有组（小组中）和组内元素（年龄最大的成员）**。需要用到相关子查询。

  ![32](D:\StudyMaterials\IT技术学习\2、SQL\SQL书籍\SQL进阶教程\SQL笔记-2022年11月21日二刷\32.png)

```sql
# 很有意思的现象：同时出现了组（二阶）和组内元素（一阶），通过相关子查询（连接条件1：同组；连接条件2：组内最大年龄）实现。
SELECT team, 
	   MAX(age),
       (SELECT MAX(member)
       FROM teams t2
       WHERE t1.team = t2.team
       AND t2.age = MAX(t1.age)) AS oldest
FROM teams t1
GROUP BY team;
```

### （2）单元素集合也是集合

- 单元素集合也是集合。这两个层级的区别分别对应着SQL 中的WHERE 子句和HAVING 子句的区别。**WHERE 子句用于处理“行”这种 0 阶的对象，而HAVING 子句用来处理“集合”这种 1 阶的对象。**
