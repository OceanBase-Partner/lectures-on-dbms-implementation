miniob date实现解析

date 数据本身可以有多种处理方式。

其中一种处理方式：date以int类型的YYYYMMDD格式保存，在select展示时转成字符串YYYY-MM-DD格式。比如 "2021-10-26"保存为整数： 20211026(2021*1000 + 10 * 100 + 26)

需要修改的代码：

1.语法上修改支持
2.增加新的数据类型后，代码里有多处类型判断的地方，类型传string名字需要同步修改，DefaultConditionFilter,BplusTreeScanner,ATTR_TYPE_NAME,insert_record_from_file
3.date的合法性判断，输入日期的格式可以在词法分析时正则表达式里过滤掉，润年，大小月日期的合法性再做进一步判断
4.只要输入的日期不合法，输出都是FAILURE\n