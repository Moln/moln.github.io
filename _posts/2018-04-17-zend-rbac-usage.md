---
layout: default
title: Zend Rbac ʹ�ó���
---


���� zend-permission ���Ӧ��, ���ڼ򵥵�ҵ��ȷʵ������еㆪ��,

����:
�� persmissions  �ֶ� id, role_id, permission
�� roles  �ֶ� id, role, desc
��ҵ��ʵ����ֱ��:

```
select * from roles where role='member';
select * from permissions where role = '$role_id' and permission='article.view'
```

��ѯ���������Ȩ��, û�о�ûȨ��

Rbac ���������ӵĹ�ϵ���. ���縸�ӹ�ϵ��ʱ��, �������ó���.

����:

�� persmissions  �ֶ� id, role_id, permission

�� roles  �ֶ�
```
id, role, parent_id, desc
1,member,0,
2,tester,1
3,executor,2
4,editor,1
...
```

�����û���ӽ�ɫ('editor', 'executor'), ����������:
1. �����н�ɫ, ����������ɫ�������
   ```
   select * from roles where role='editor'
   select * from roles where role='executor'
   select * from roles where role='tester'
   ```

2. Ȼ�������ɫ�����н�ɫȨ�޶�ɨ����, ֱ�������Ȩ��Ϊֹ
   ```
   select * from permissions where role_id = '$id1' and permission='article.view'
   ...
   select * from permissions where role_id = '$idN' and permission='article.view'
   ```

������������ɫ��ϵ�Ӵ�, 1. PHP����ݹ��߼�, 2. ��ɫ��Ȩ�޹�ϵ��Ļ�,�ڲ�ѯ���ǺܷѾ�.

Rbac ������ƾ������ó���, ������д�ݹ��߼�, 
�������ݲ�ѯ����ð����ݶ��������������.

```
select * from roles;
select * from permissions;
```