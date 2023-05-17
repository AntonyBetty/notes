INDIRECT——间接的



# 1、直接引用对比间接引用

## ① 直接引用——INDEX

![直接引用对比间接引用-1](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十九讲图片\直接引用对比间接引用-1.png)

注意：先使用 【ROW()*5-25】找到了想要的取值行数的关系。再用INDEX函数（取值函数）拿到了对应的数据。这是直接引用



#  ② 间接引用——INDIRECT

![直接引用对比间接引用-2](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十九讲图片\直接引用对比间接引用-2.png)



# 2、使用INDIRECT函数实现跨表引用（各Sheet中顺序相同）

![直接引用对比间接引用-3](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十九讲图片\直接引用对比间接引用-3.png)

跨表引用需注意引用的格式 “SHEET名！A1”这样的格式。



# 3、使用INDIRECT函数实现跨表引用（各Sheet中顺序不同）

![VLOOKUP函数+INDIRECT函数](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十九讲图片\VLOOKUP函数+INDIRECT函数.png)



# 4、使用INDIRECT函数实现跨表引用（各Sheet中顺序不同）

![混合引用](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十九讲图片\混合引用.png)





# 5、二级下拉框的设置方法——使用INDIRECT方法

![二级下拉框](D:\StudyMaterials\IT技术学习\1、Excel\王佩丰Excel24讲笔记\章节二\第十九讲图片\二级下拉框.png)

注意要先对各地级市进行自定义名称，如选择吉林省的所有地级市，定义名称为“吉林省”。在数据有效性中，对INDIRECT方法的使用。



# 6、关于编程语言\工具等学习的方法论——“先把书读薄、再把书读厚”

如要学VBA，千万不要抠具体细节，不会的先放下。

先找本薄的书籍或24小时入门视频（薄书），花2天时间过一遍，了解架构、体系。再看对应的厚的书，包含各个技术细节。

