 [Home](index)

# Date实现解析

- by caizj

一种实现方式：date以int类型的YYYYMMDD格式保存，比如2021-10-21，保存为整数就是2021\*1000 + 10\*100 + 21，在select展示时转成字符串YYYY-MM-DD格式，注意月份和天数要使用0填充。

## 修改点

### 语法上修改支持

需要可匹配date的token词和DATE_STR值（一定要先于SSS，因为date的输入DATE_STR是SSS的子集)

语法(yacc文件)上增加type，value里增加DATE_STR值

### Date的合法性判断

输入日期的格式可以在词法分析时正则表达式里过滤掉。润年，大小月日期的合法性在普通代码中再做进一步判断。

在parse阶段，对date做校验，并格式化成int值保存。

日期输入的基础格式过滤正则表达式是：

```
{QUOTE}[0-9]{4}\-(0?[1-9]|1[012])\-(0?[1-9]|[12][0-9]|3[01]){QUOTE}
```



### 增加新的类型date枚举

代码里有多处和类型耦合地方（增加一个类型，要动很多处离散的代码，基础代码在这方面的可维护性不好)

包括不限于，以下几处：

> DefaultConditionFilter
>
> BplusTreeScanner
>
> ATTR_TYPE_NAME
>
> insert_record_from_file

### Date select展示

TupleRecordConverter::add_record时做格式转换

### 异常失败处理

只要输入的日期不合法，输出都是FAILURE\n。包括查询的where条件、插入的日期值、更新的值等。

## 自测覆盖点

1. 日期输入，包括合法和非法日期格式。非法日期可以写单元测试做。
2. 日期值比较=、 >、 <、 >=、 <=
3. 日期字段当索引
4. 日期展示格式，注意月份和天数补充0

 [Home](index)