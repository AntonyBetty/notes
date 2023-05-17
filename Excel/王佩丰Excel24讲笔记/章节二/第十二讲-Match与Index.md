--------------------------------------

# 1、VLOOKUP函数的一个短板：只能从左往右找（只是查找最左侧一列），不能从右往左找



# 2、Match函数用于查找、Index函数用于引用，Match+Index实现VLOOKUP的效果



# 3、告诉Match区域，Match告诉我们“老张在隔壁、第四个座位上”；告诉Index“老张在第四个座位上”



# 4、match+index实现vlookup的效果（Match与Index更灵活）

![index和match](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十二讲图片\index和match.png)



# 5、混合引用

![混合引用](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十二讲图片\混合引用.png)

上图，只要涉及同时往右和往下拖拽填充的，都会涉及混合引用。



# 6、COLUMN函数返回列的序号

①=COLUMN(A1)，返回A1单元格对应的列数，即返回1（A是第一列）。

②=COLUMN（），返回当前写入单元格对应的列的列数。



# 7、要查找的数据和数据源顺序对应时，通过VLOOKUP查找多列（要查找的数据和数据源的顺序一一对应的可能性不大）

![顺序对应时VLOOKUP查找多列](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十二讲图片\顺序对应时VLOOKUP查找多列.png)

①注意1：因“公司名称”列在数据源中是第二列，而COLUMN（）将返回5（E列为该sheet中的第五列），因此使用COLUMN()-3进行拼凑出序号。

②注意2：将“D4”中的D进行锁住，这样往右边拖拽的时候，不会由D4变成E4了，可实现整个表格的拖拽。



# 8、要查找的数据和数据源顺序不能一一对应时，通过VLOOKUP+MATCH查找多列

![VLOOKUP与Match集合](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十二讲图片\VLOOKUP与Match集合.png)

①注意1：MATCH（）函数返回了表头在表头数据源（注意使用绝对引用）的序数（即第几列）。

②注意2：将“A3”中的A锁死，即把A列锁死，防止右拖的时候，A列发生变化（即客户ID的位置发生了变化）。

③注意3：将“B2”中的2锁死，即把2行锁死，防止下拖的时候，行数发生变化（此时找不到“公司名称”这种表头了）。



# 9、MATCH+INDEX函数结合同样可以引用图片

