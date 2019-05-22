---
layout: default
title: 现代PHP框架的选择
---

个人对现框架的看法与讨论，

PHP 框架很多，本次先拿非编译的框架对比:

- zend-mvc: v3
- zend-expressive: v3
- slim v3
- symfony v4 / silex
- laravel v5
- Yii v2
- CakePHP v3
- Lumen v5

主要探讨的点:

- 选择性加载
- PSR规范
- 最小化依赖
- 异步模式支持友好性
- 无状态服务设计

### 最小化依赖/选择性加载

指框架并未强制集成 Cache/db 等,非框架MVC/ADR 以外的组件,这样做好处:

- 减少项目目录大小. 比如你的业务不需要用到cache,就没必要下载 cache库.
- 非强制或非强引导性让你选择框架带的应用库
- 选择性加载. 可以根据项目需要，选择适合你的应用库。
  比如我的项目很小想直接用php的pdo更省事学习曲线低性能更好, 我的另一项目需要doctrine-orm 来更好的维护我的数据模型结构。

在这几点 zend v3, symfony v4, slim, 做的比较好。
其它几个框架则都集成了自己库类操作,无法分离。

### 异步执行支持

目前PHP通常运行的几种方式: 

- nginx+php-fpm
- Apache+php
- 基于异步:
  - swoole
  - workman
  - ReactPHP
  - 等等

### 对比

|                  | zend-mvc | zend-expressive | symfony | laravel | slim | yii | CakePHP | lumen |
|------------------|----------|-----------------|---------|---------|------|-----|---------|-------|
| 选择性加载        | ✓        | ✓              | ✓      |         |  ✓   |     |         |       | 
| 最小化依赖        | ✓       | ✓               | ✓      |         | ✓    |     |         |       |
| 异步架构支持       |         | ✓              |         |         |      |     |         |       |
| PSR-1/PSR-2/PSR-4 | ✓      | ✓               | ✓      | ✓       | ✓    | ✓  |  ✓      |  ✓    |
| PSR-7             |        | ✓               |        |         | ✓     |     | ✓      |       |
| PSR-15            | ✓      | ✓               |         |         | ⚪    |     | ⚪      |       |
| PSR-11            | ✓      | ✓               | ✓      | ✓       | ✓    |     |         | ✓     | 

### 并发性能

#### 硬件信息

- **Cpu:** Intel(R) Xeon(R) CPU E3-1270 V2 @ 3.50GHz 
- **Memory:** 16GB 

#### 软件

- nginx/1.10.3 (Ubuntu)
- PHP 7.2.5-1+ubuntu16.04.1+deb.sury.org+1 (cli) (built: May  5 2018 04:59:13) ( NTS )

#### 执行测试

> Composer 安装参数 `composer update -o --no-dev`
> 
> 以下皆为生产环境配置, php 开启 opcache 扩展

| Frameworks         | 单个请求内存使用<br>_(在Action中输出内存使用)_<br>`memory_get_usage()` | `webbench -c 100 -t 10` |
|--------------------|---------------------------------------------------------------------|--------------------------|
| Nginx 静态HTML     |   -       | Speed=2158812 pages/min, 8887110 bytes/sec. <br /> Requests: 359802 susceed, 0 failed. |
| PHP(无框架)        | 386,944    |  Speed=1903872 pages/min, 5076992 bytes/sec. <br /> Requests: 317312 susceed, 0 failed. |
| zend-mvc:v3        | 1,197,128 | Speed=153606 pages/min, 412176 bytes/sec. <br /> Requests: 25601 susceed, 0 failed. |
| zend-expressive:v3 | 709,016   | Speed=447930 pages/min, 1194480 bytes/sec. <br /> Requests: 74655 susceed, 0 failed. |
| symfony:v4         | 2,260,808 | Speed=62874 pages/min, 244160 bytes/sec. <br /> Requests: 10479 susceed, 0 failed. |
| laravel:v5         | 1,686,408 | Speed=64020 pages/min, 1043661 bytes/sec. <br /> Requests: 10670 susceed, 0 failed. |
| slim:v3            | 644,032   | Speed=468744 pages/min, 2562467 bytes/sec. <br /> Requests: 78124 susceed, 0 failed. |
| yii:v2             | 1,018,520 | Speed=409410 pages/min, 1098583 bytes/sec. <br /> Requests: 68235 susceed, 0 failed. |
| cakephp:v3         | 1,579,392 | Speed=121176 pages/min, 325139 bytes/sec. <br /> Requests: 20196 susceed, 0 failed. |
| lumen:v5           | 710,208   | Speed=459258 pages/min, 1484934 bytes/sec. <br /> Requests: 76543 susceed, 0 failed. |
