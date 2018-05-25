---
layout: default
title: 影响mysql insert 性能的参数
---


- `log_bin`  根据业务情况, 没必要就关掉.
- `innodb_flush_log_at_trx_commit` 根据业务情况, 没必要就设置 0

### 数据 insert 效率只有每秒300

**原因:** mysql innodb引擎开启了日志事务功能，参数配置：`my.cnf` 默认 `innodb_flush_log_at_trx_commit=1` 调整为0;

修改后 insert 大概每秒 5000,

### Delete 后的数据表,影响索引?

发现一数据总数据1700万, 由于程序脚本编写BUG导致, 向数据表误导入500+万数据.
经 Delete 语法删除尾部多余的 500+万数据后.
查询数据尾部数据 `SELECT * FROM t ORDER BY id DESC LIMIT 10` 非常缓慢.
原因未知,选择了清空表重新导入数据.

相关问题: https://stackoverflow.com/questions/2924671/mysql-delete-and-index-hint
