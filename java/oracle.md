# centos7安装oracle19c

安装oracle前，先配置好/etc/hosts ，需指定当前ip，例192.168.85.144

https://www.linuxidc.com/Linux/2019-06/158985.htm

## 重启监听

> lsnrctl
>
> stop
>
> start

## 重启oracle

> sqlplus / as sysdba
>
> startup  # 开启
>
> shutdown immediate  # 关闭

# jdbcType 对应数据库

http://blog.csdn.net/loongshawn/article/details/50496460

# oracle删除数据

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

# 导出数据泵

1 先在当前用户下，例  oracle用户下创建导出目录，必须创建。

> create or replace directory dumpdir as '/home/oracle';

2 导出脚本

> c##oracle/root@ORCLCDB  数据库用户名/密码@实例名
>
> sshpass -p  root(远程服务器的密码)
>
> backup.dmp 备份文件的名称

```sh
#!/bin/sh
. /etc/profile
. ~/.bash_profile
echo "$(date +'%Y-%m-%d %H:%M:%S') export start";
mv /home/oracle/backup.dmp /home/oracle/backup.dmp.bak;
expdp c##oracle/root@ORCLCDB dumpfile=backup.dmp directory=DUMPDIR;
sshpass -p root scp -r /home/oracle/backup.dmp oracle@192.168.85.145:/home/oracle/;
echo "$(date +'%Y-%m-%d %H:%M:%S') export success";
# end
```

3 定时任务，每俩分钟

在linux下，crontab -e

```sh
*/2 * * * * nohup /bin/sh /home/oracle/exp_imp/exportOracle.sh >> /home/oracle/exp_imp/log.txt &
```

若定时任务不生效

在~/.bash_profile中添加

```sh
export ORACLE_BASE=/opt/oracle/
export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1
export ORACLE_SID=ORCLCDB
export PATH=$PATH:$ORACLE_HOME/bin
```

# 导入数据泵

1 先在当前用户下，例  oracle用户下创建导出目录，必须创建。

> create or replace directory dumpdir as '/home/oracle';

2 导入脚本

> table_exists_action=replace  导入时，若表存在，则先删除表

```sh
#!/bin/sh
# update if table is exist
. /etc/profile
. ~/.bash_profile
echo "$(date +'%Y-%m-%d %H:%M:%S') import start";
impdp c##oracle/root@ORCLCDB dumpfile=backup.dmp directory=dumpdir table_exists_action=replace
echo "$(date +'%Y-%m-%d %H:%M:%S') import success";
```

3 定时任务，每俩分钟

在linux下，crontab -e

```sh
*/2 * * * * nohup /bin/sh /home/oracle/exp_imp/importOracle.sh >> /home/oracle/exp_imp/log.txt &
```

# 表空间

```sql
#创建用户
create user c##oracle IDENTIFIED BY root; 
# 授权
grant dba,connect,resource,unlimited tablespace to c##oracle container=all;
# 在指定用户登录下，创建表空间
create tablespace GDP datafile '/opt/oracle/product/19c/tables/gdp.dbf' size 200M  AUTOEXTEND ON;
# 创建临时表空间
create temporary tablespace GDP_temp  tempfile '\opt\oracle\product\19c\tables\gdp_temp.dbf' size 100m reuse autoextend on next 20m maxsize unlimited; 
# 给用户指定表空间
alter user c##oracle default tablespace GDP temporary tablespace GDP_temp;
alter user c##oracle default tablespace 'GDP';
```

# exp 和 imp

效率比数据泵低，但不用创建文件夹。需要在oracle用户下，指定导出和导入用户名

> https://blog.csdn.net/jqdelove/article/details/112508685
>
> https://blog.csdn.net/lan19900810/article/details/92783907?utm_term=oracle%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B9%8B%E9%97%B4%E5%AE%9A%E6%97%B6%E5%90%8C%E6%AD%A5%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-2-92783907&spm=3001.4430

```sh
# 导出 都是相对路径
oracle_username=name
oracle_password=pass
exp oracle_username/oracle_password file=exp_oracle.dmp log=exp.log fromuser=name

# 导入，导入前先清库
imp oracle_username/oracle_password file=exp_oracle.dmp log=imp.log fromuser=name touser=name ignore=y
```

# SpringBoot oracle 用户名和密码加密

> jasypt
>
> https://www.cnblogs.com/xuchen0117/p/14375211.html

# oracle死锁

> https://blog.csdn.net/baidu_30809315/article/details/107234267

查看被锁的表

```sql
select object_name, machine, s.sid, s.serial#
  from v$locked_object l, dba_objects o, v$session s
 where l.object_id = o.OBJECT_ID
       and l.session_id = s.sid;
```

kill

```sql
alter system kill session '277,1817';
```

