---
layout: default
title: PDO fetch 结果int类型不强转string
---

`PDO::fetch` 结果字段类型都被强转string.

需要设置PDO选项

```
$options = [
  PDO::ATTR_STRINGIFY_FETCHES => false, //设置不强转
  PDO::ATTR_EMULATE_PREPARES => false, //禁用预处理
];
$dsn = 'mydb';
$user = '';
$password = '';

try {
    $dbh = new PDO($dsn, $user, $password, $options);
} catch (PDOException $e) {
    echo 'Connection failed: ' . $e->getMessage();
}
```
