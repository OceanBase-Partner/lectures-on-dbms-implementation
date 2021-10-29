### select-meta 实现解析

##### 语法错误返回失败

sql parse失败返回时需要set_response("FAILURE\n")

parse失败处理在ParseStage::handle_request，这里原来的snprintf的error参数是有问题的，会导致coredump，因为不需要返回详细的出错信息，都可以不要了

##### 查询时表、字段不存在返回失败

查询入口在ExecuteStage::handle_request中，case SCF_SELECT会调用do_select，如果失败 set_response("FAILURE\n");

