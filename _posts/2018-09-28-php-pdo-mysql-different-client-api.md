---
layout: page
title: PHP pdo_mysql 不同的Client API 区别
---

在设置 PHP PDO 查询结果 int 不强制转为 string:

```
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
$pdo->setAttribute(PDO::ATTR_STRINGIFY_FETCHES, false);
```

遇到情况, 同PHP版本A服务器设置有效, 查询结果为 int 类型, B服务器无效.

后续发现 与 PDO 的 `Client API verion` 有关.

### MariaDB

```
pdo_mysql

PDO Driver for MySQL => enabled
Client API version => 10.1.30-MariaDB
```

mariadb api `PDO::ATTR_EMULATE_PREPARES` 的设置无效.

### mysqlnd

```
Client API version => mysqlnd 5.0.12-dev - 20150407 - $Id: 38fea24f2847fa7519001be390c98ae0acafe387 $
```

需要不强制转换 string的, 需要设置此驱动api
