[Home](index)

# select-meta 实现解析

## 语法错误返回失败

sql parse失败返回时需要`set_response("FAILURE\n")`

在其它的题目中，也有一些用例可以通过语法解析失败，直接判断返回FAILURE，比如聚合函数中，非法参数，可以直接在语法解析时返回FAILURE，也可以在后续逻辑判断中返回FAILURE

代码在`parse_stage.cpp`的`ParseStage::handle_request`中：
```c++
StageEvent *ParseStage::handle_request(StageEvent *event) {
  SQLStageEvent *sql_event = static_cast<SQLStageEvent *>(event);
  const std::string &sql = sql_event->get_sql();

  Query *result = query_create();
  if (nullptr == result) {
    LOG_ERROR("Failed to create query.");
    return nullptr;
  }

  RC ret = parse(sql.c_str(), result);
  if (ret != RC::SUCCESS) {
    // set error information to event
    //const char *error = result->sstr.errors != nullptr ? result->sstr.errors : "Unknown error";
    //char response[256];
    //snprintf(response, sizeof(response), "Failed to parse sql: %s, error msg: %s\n", sql.c_str(), error);
    //sql_event->session_event()->set_response(response);
    sql_event->session_event()->set_response("FAILURE\n"); // 直接修改为返回FAILURE即可，换行符可以有也可以没有
    query_destroy(result);
    return nullptr;
  }

  return new ExecutionPlanEvent(sql_event, result);
}
```

parse失败处理在`ParseStage::handle_request`，这里原来的snprintf的error参数是有问题的，会导致coredump，因为不需要返回详细的出错信息，都可以不要了.也就是上面注释掉的原始代码.

## 查询时表、字段不存在返回失败

查询入口在`ExecuteStage::handle_request`中，case `SCF_SELECT`会调用`do_select`，如果失败 `set_response("FAILURE\n")`;
需要考虑的元数据包括查询的表、字段以及where条件中的字段(可能也有表)。
参考代码：
```c++
RC do_select(Session *session, const char* db, Query* sql, int id, std::map<int,TupleSet>& tuple_val, TupleSet& tuple_set, bool& multi_table) {
    Selects& selects = sql->sstr[id].selection;
    Trx *trx = session->current_trx();
    RC rc = preprocess_attr_in_relation(db, selects); // 检查查询的字段/属性、表是否都存在
    if(rc != RC::SUCCESS) {
        return rc;
    }
    rc = preprocess_condition_in_relation(db, selects); // 检查查询条件是否都是合法的
    if(rc != RC::SUCCESS) {
        return rc;
    }

		...
}
```



[Home](index)
