---
layout: page
title: Mysql 读取 latin1 字符转为 gbk 的几种方式
---


起因
----

主业务在写入账号数据表时, 连接编码 mysql 使用的是 latin1, 这边另一业务系统读取数据时, 默认情况显示乱码.

数据表格式

```sql
CREATE TABLE `users` (
  `id` bigint(21) NOT NULL DEFAULT 0,
  `name` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

处理
----

### PHP 处理方式1

之前只了解PHP处理方式

PDO 连接数据库时, 连接的编码方式需要是 laint1, 查询结果读取出来就是 gbk 了.

```php
$dsn = 'mysql:dbname=test;host=127.0.0.1;charset=latin1;';
$user = 'root';
$password = '';
try {
    $dbh = new PDO($dsn, $user, $password,  [
       PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]);
} catch (PDOException $e) {
    echo 'Connection failed: ' . $e->getMessage();
}
$sth = $dbh->prepare('select id,name from users where id=2921 limit 1');
$sth->execute();
$res = $sth->fetch();
print_r($res);

// 结果为 GBK 编码中文:
// [name] => 佛爺
```

### PHP 处理方式2

接触 .net 后, 了解的另一方式.


PDO 连接数据库时, 连接的编码方式是 utf8, 查询结果读取结果进行转码 `Windows-1252`.

```php
$dsn = 'mysql:dbname=test;host=127.0.0.1;charset=utf8;';
$user = 'root';
$password = '';
try {
    $dbh = new PDO($dsn, $user, $password,  [
       PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]);
} catch (PDOException $e) {
    echo 'Connection failed: ' . $e->getMessage();
}
$sth = $dbh->prepare('select id,name from users where id=2921 limit 1');
$sth->execute();
$res = $sth->fetch();
$res['name2'] = iconv('utf-8', 'Windows-1252', $res['name'])
print_r($res);

// 结果为 GBK 编码中文:
// [name2] => 佛爺
```

### .net 处理

之前了解的是 **PHP处理方式1** , 原以为只要修改 mysql 连接编码为 latin1 就能解决.

```c#
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
string connection = "Server=localhost;Database=test;uid=root;password=;CharSet=latin1;SslMode=none;";

using (var conn = new MySqlConnection (connection)) {
    conn.Open();
    var sql = "select id,name from users where id=2921 limit 1";

    using (var cmd = new MySqlCommand (sql, conn)) {
        var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine(String.Format(
                "Result: {0}, {1}, {2}", 
                reader[0],
                reader[1],
                Encoding.GetEncoding("gbk").GetString((string)reader[1])
            ));
        }
    }
}
```

然并卵, 读取的结果还是 `Encoding.UTF8` latin 结果乱码.

后面经分析, 执行 `conn.Open()` 的时候, sql 执行日志 

```
		   116 Query	SHOW VARIABLES
		   116 Query	SELECT TIMEDIFF(NOW(), UTC_TIMESTAMP())
		   116 Query	SHOW COLLATION
		   116 Query	SET NAMES latin1
		   116 Query	SET character_set_results=NULL
```

主要是 `character_set_results`, 不知为何 `MySqlConnection` 要执行 `SET character_set_results=NULL`, 
导致结果还是utf8乱码.

后续手动执行 `SET character_set_results=latin1` 也不行.

通过断点调试发现 `MySql.Data.MySqlClient.NativeDriver.GetColumnData()` 读取的
`Encoding` 是 `SBCSCodePageEncoding` `CodePage=1252`.

也就是 `windows-1252` 编码

所以正确处理方式是:

```c#
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
string connection = "Server=localhost;Database=test;uid=root;password=;CharSet=utf8;SslMode=none;";

using (var conn = new MySqlConnection (connection)) {
    conn.Open();
    var sql = "select id,name from users where id=2921 limit 1";

    using (var cmd = new MySqlCommand (sql, conn)) {
        var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine(String.Format(
                "Result: {0}, {1}, {2}", 
                reader[0],
                reader[1],
                Encoding.GetEncoding("gbk").GetString(Encoding.GetEncoding(1252).GetBytes((string)reader[1]))
            ));
        }
    }
}
```

这后续也就想出 **php 处理方式2**

期间遇到个坑, 之前是使用的 `iso-8859-1` 编码, 字符串 `佛爺` 会被转换成 `佛?`. 这是由于 `iso-8859-1` 编码范围导致.
正确方式应该用 `windows-1252`.

参考文章
-------

- [不可使用 iso-8859-1, 而使用 windows-1252 原因.](https://www.i18nqa.com/debug/table-iso8859-1-vs-windows-1252.html)
