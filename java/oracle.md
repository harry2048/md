## jdbcType 对应数据库

http://blog.csdn.net/loongshawn/article/details/50496460

## oracle删除数据

> 删除当前用户下所有表里面的数据

```sql
declare
cursor curl is select table_name from cat where table_type='TABLE';
begin
  for cur2 in curl loop
    execute immediate 'delete from '|| cur2.table_name;
    commit;
  end loop;
end;
/
--删除当前用户下所有表
declare
cursor curl is select table_name from cat where table_type ='TABLE';
begin
  for cur2 in cur1 loop
  execute immediate 'drop table '|| cur2.table_name||'cascade constraints';
  end loop;
end;
/
--删除当前用户下所有的序列
declare
cursor cur1 is select table_name from cat where table_type='SEQUENCE';
begin
  for cur2 in cur1 loop
    execute immediate 'drop SEQUENCE' || cur2.table_name;
  end loop;
end;
```

