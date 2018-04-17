---
layout: default
title: Zend Rbac 使用场合
---


对于 zend-permission 组件应用, 用于简单的业务确实用组件有点啰嗦,

比如:
表 persmissions  字段 id, role_id, permission
表 roles  字段 id, role, desc
在业务实现上直接:

```
select * from roles where role='member';
select * from permissions where role = '$role_id' and permission='article.view'
```

查询出结果就有权限, 没有就没权限

Rbac 是遇到复杂的关系情况. 比如父子关系的时候, 就派上用场了.

比如:

表 persmissions  字段 id, role_id, permission

表 roles  字段
```
id, role, parent_id, desc
1,member,0,
2,tester,1
3,executor,2
4,editor,1
...
```

假如用户多从角色('editor', 'executor'), 处理流程是:
1. 得所有角色, 包括父级角色都查出来
   ```
   select * from roles where role='editor'
   select * from roles where role='executor'
   select * from roles where role='tester'
   ```

2. 然后遍历角色把所有角色权限都扫个遍, 直到查出有权限为止
   ```
   select * from permissions where role_id = '$id1' and permission='article.view'
   ...
   select * from permissions where role_id = '$idN' and permission='article.view'
   ```

这种情况如果角色体系庞大, 1. PHP处理递归逻辑, 2. 角色和权限关系多的话,在查询上是很费劲.

Rbac 这种设计就派上用场了, 不用你写递归逻辑, 
但在数据查询上最好把数据都查出来缓存起来.

```
select * from roles;
select * from permissions;
```