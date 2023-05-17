# 1、SUMIF函数的使用：=sumif(range,criteria,[sum_range])



# 2、SUMIF函数灵活度很大，处理跨区域的数据也是可以的

![sumif函数](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十讲图片\sumif函数.png)

SUMIF函数的灵活度很大，如上图，即使写的是SUMIF()中第三个[sum_range]区间写的是B列，然而Excel仍将其处理成为了整个区域，这就是它的灵活性。



# 3、通过建立辅助列，实现两个条件的SUMIF功能



![建立辅助列](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十讲图片\建立辅助列.png)



![在辅助列中进行判断](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十讲图片\在辅助列中进行判断.png)



# 4、SUMIFS函数实现满足多个条件的求和（条件之间是“且”的关系 ）

![SUMIFS函数实现两个条件的SUMIF](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十讲图片\SUMIFS函数实现两个条件的SUMIF.png)



# 5、用SUMIF实现VLOOKUP的功能（只能用于数值的查询，字符串的不行，因为SUMIF无法加字符串

）

![SUMIF查找数据](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十讲图片\SUMIF查找数据.png)



# 6、SUMIF设置数据有效性

![SUMIF设置数据有效性](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十讲图片\SUMIF设置数据有效性.png)

上图，设置数量列的数据有效性，使得产品累计的数量不得超过库存表中对应的总数量。