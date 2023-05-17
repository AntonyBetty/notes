# 1、COUNT函数只会数数字，不会去数文本

 

# 2、COUNTIF(rang，criteria标准)，在某个范围内依据某个标准数数



# 3、COUNTIF函数中，标准中涉及比较的，用双引号括起来，如“>=60“



# 4、COUNTIF默认处理前15位，使用COUNTIF函数在处理超过15位的文本时候，配合通配符可实现处理效果

=COUNTIF(range,A1&"*")

![countif与通配符](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第九讲图片\countif与通配符.png)

  

# 5、IF函数与COUNTIF函数联合使用用来查找重复

![COUNTIF函数与IF函数联合使用](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第九讲图片\COUNTIF函数与IF函数联合使用.png)

上图，如果左列的数据在右列区域内，则COUNTIF函数返回1，再用IF函数进行逻辑判断后输出即可。



# 6、COUNTIF函数与条件格式的结合使用，可以使某类数据格式突出

![条件格式-1](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第九讲图片\条件格式-1.png)

上图，选择某列，点”条件格式“中的”新建规则”。

![条件格式-2](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第九讲图片\条件格式-2.png)

上图，将在某区域内有数的人（即已经体检）的数据，设置为绿色单元格。



# 7、使用COUNTIF函数完成自定义数据有效性——实现某列数据不能重复

![使用COUNTIF函数实现某列输入值不得重复](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第九讲图片\使用COUNTIF函数实现某列输入值不得重复.png)



# 8、某一区域限制数据有效性——注意使用绝对引用

![区域数据有效性](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第九讲图片\区域数据有效性.png)



# 9、COUNTIFS函数是COUNTIF函数的升级版，实现多个区域内、多个条件的筛选

![countifs](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第九讲图片\countifs.png)