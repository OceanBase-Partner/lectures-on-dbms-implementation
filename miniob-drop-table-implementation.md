# miniob - drop table 实现解析

**代码部分主要添加在：**

sql流转到default_storge阶段的时候，在处理sql的函数中，新增一个drop_table的case。

drop table就是删除表，在create table t时，会新建一个t.table文件，同时为了存储数据也会新建一个t.data文件存储下来。

那么删除表，就需要**删除t.table文件和t.data文件**。

同时由于buffer pool的存在，在新建表和插入数据的时候，会写入buffer pool缓存。所以drop table，不仅需要删除文件，也需要**清空buffer pool** ，防止在数据没落盘的时候，再建立同名表，仍然可以查询到数据。

如果建立了索引，比如t_id on t(id)，那么也会新建一个t_id.index文件，也需要删除这个文件。

这些东西全部清空，那么就完成了drop table。