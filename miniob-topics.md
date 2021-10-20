# 背景

本次大赛赛题, 是在一个miniob(mini数据库)库的基础上, 让参数选手实现数据库的非常基础的功能, 功能分为入门（预选赛）, 中级（决赛）, 高阶（黑客松） 3个阶段。 入门门槛较低, 适合所有参赛选手。 面向的对象主要是在校学生，数据库爱好者, 或者对基础技术有一定兴趣的爱好者, 并且考题对诸多模块做了简化，比如不考虑并发操作, 事务比较简单。
目标是让不熟悉数据库设计和实现的同学能够快速的了解与深入学习数据库内核，期望通过miniob相关训练之后，能够对各个数据库内核模块的功能与它们之间的关联有所了解，并能够在使用时，设计出高效的SQL,  并帮助降低学习OceanBase 内核的学习门槛。

比赛分为三个阶段：预选赛、决赛和黑客松48小时极限挑战赛。

# 预选赛

预选赛，题目分为两类，一类必做题，一类选做题。选做题按照实现的功能计分。

## 必做题

| 名称 | 描述 | 测试用例示例 |
| ---- | ---- | -------------|
| 优化buffer pool | 必做。实现LRU淘汰算法或其它淘汰算法 | |
| 查询元数据校验 | 必做。查询语句中存在不存在的列名、表名等，需要返回失败。需要检查代码，判断是否需要返回错误的地方都返回错误了。 |create table t(id int, age int);<br/>select * from t where name='a'; <br/>select address from t where id=1;<br/>select * from t_1000;|
| drop table | 必做。删除表。清除表相关的资源。 |create table t(id int, age int);<br/>create table t(id int, name char);<br/>drop table t;<br/>create table t(id int, name char);|
| 实现update功能 | 必做。update单个字段即可。 |update t set age =100 where id=2;<br/>update set age=20 where id>100;|
| 增加date字段 | 必做。date测试不会超过2038年2月。注意处理非法的date输入，需要返回FAILURE。 |create table t(id int, birthday date);<br/>insert into t values(1, '2020-09-10');<br/>insert into t values(2, '2021-1-2');<br/>select * from t;|
| 多表查询 | 必做。支持多张表的笛卡尔积关联查询。需要实现select * from t1,t2; select t1.*,t2.* from t1,t2;以及select t1.id,t2.id from t1,t2;查询可能会带条件。查询结果展示格式参考单表查询。每一列必须带有表信息，比如:<br/>t1.id \|  t2.id <br/>1 \| 1 |select * from t1,t2;<br/>select * from t1,t2 where t1.id=t2.id and t1.age > 10;<br/>select * from t1,t2,t3;|
| 聚合运算 | 需要实现max/min/count/avg.<br/>包含聚合字段时，只会出现聚合字段。聚合函数中的参数不会是表达式，比如age +1 |select max(age) from t1; select count(*) from t1; select count(1) from t1; select count(id) from t1;|



## 选做题

| 名称               | 分值 | 描述                                                         | 测试用例示例                                                 |
| ------------------ | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 多表join操作       | 20   | INNER JOIN。需要支持join多张表。需要考虑大表问题，不要直接使用笛卡尔积再过滤 | select * from t1 inner join t2 on t1.id=t2.id;<br/>select * from t1 inner join t2 on t1.id=t2.id inner join t3 on t1.id=t3.id;<br/>selec * from t1 inner join t2 on t1.id=t2.id and t2.age>10 where t1.name >='a'; |
| 一次插入多条数据   | 10   | 一次插入的数据要同时成功或失败。                             | insert into t1 values(1,1),(2,2),(3,3);                      |
| 唯一索引           | 10   | 唯一索引：create unique index                                | create unique index i_id on t1(id);<br/>insert into t1 values(1,1);<br/>insert into t1 values(1,2); -- failed |
| 支持NULL类型       | 10   | 包括但不限于建表、查询和插入。默认情况不允许为NULL，使用nullable关键字表示字段允许为NULL。<br/>Null不区分大小写 | create table t1 (id int not null, age int not null, address nullable); create table t1 (id int, age int, address char nullable); insert into t1 values(1,1, null); |
| 简单子查询         | 10   | 支持简单的IN(NOT IN)语句；<br/>支持与子查询结果做比较运算；<br/>支持子查询中带聚合函数。<br/>子查询中不会与主查询做关联。 | select * from t1 where name in(select name from t2);<br/>select * from t1 where t1.age >(select max(t2.age) from t2);<br/>select * from t1 where t1.age > (select avg(t2.age) from t2) and t1.age > 20.0; <br/>NOTE: 表达式中可能存在不同类型值比较 |
| 多列索引           | 20   | 单个索引关联了多个字段                                       | create index i_id on t1(id, age);                            |
| 超长字段           | 20   | 超长字段的长度可能超出一页，比如常见的text,blob等。这里仅要求实现text（text 长度固定4096字节），可以当做字符串实现。<br/>注意：当前的查询，只能支持一次返回少量数据，需要扩展 | create table t(id int, age int, info text);<br/>insert into t(1,1, 'a very very long string');<br/>select * from t where id=1; |
| 查询条件支持表达式 | 20   | 查询条件中支持运算表达式，这里的运算表达式包括 +-*/。<br/>仅支持基本数据的运算即可，不对date字段做考察。<br/>运算出现异常，按照NULL规则处理。<br/>只需要考虑select。 | select * from t1,t2 where t1.age +10 > t2.age *2 + 3-(t1.age +10)/3;<br/>select t1.col1+t2.col2 from t1,t2 where t1.age +10 > t2.age *2 + 3-(t1.age +10)/3; |
| 复杂子查询         | 20   | 子查询在WHERE条件中，子查询语句支持多张表与AND条件表达式，查询条件支持max/min等 | select * from t1 where age in (select id from t2 where t2.name in (select name from t3)) |
| 排序               | 10   | 支持oder by功能。不指定排序顺序默认为升序(asc)。<br/>不需要支持oder by字段为数字的情况，比如select * from t order by 1; | select * from t,t1 where t.id=t1.id order by t.id asc,t1.score desc; |
| 分组               | 20   | 支持group by功能。group by中的聚合函数也不要求支持表达式     | select t.id, t.name, avg(t.score),avg(t2.age) from t,t2 where t.id=t2.id group by t.id; |

# 测试常见问题

- 优化buffer pool



题目中要求实现一个LRU算法。但是LRU算法有很多种，所以大家可以按照自己的想法来实现。因为不具备统一性，所以不做统一测试。

可以考虑后期，评出排名后，再检查算法实现。



- basic 测试

基础测试是隐藏的测试case，是代码本身就有的功能，比如创建表、插入数据等。如果选手把原生仓库代码提交上去，就能够测试通过basic。

- select-meta 测试

这个测试对应了“元数据校验”。选手们应该先做这个case。

常见测试失败场景有一个是 where 条件校验时 server core了。

- drop-table case测试失败

目前遇到最多的失败情况是没有校验元数据，比如表删除后，再执行select，按照“元数据校验”规则，应该返回"FAILURE"。

- date 测试

date测试需要注意校验日期有效性。比如输入"2021-2-31"，一个非法的日期，应该返回"FAILURE"。
date不需要考虑和string(char)做对比。
date也不会用来计算平均值。

> 温馨提示：date 可以使用整数存储，简化处理

- 浮点数展示问题

按照输出要求，浮点数最多保留两位小数，并且去掉多余的0。目前没有直接的接口能够输出这种格式。比如 printf("%.2f", f); 会输出 1.00，printf("%g", f); 虽然会删除多余的0，但是数据比较大或者小数位比较多时展示结果也不符合要求。

- update 测试

很多同学遇到丢数据的问题。

