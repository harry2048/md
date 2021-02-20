## 批量插入

### oracle

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.oracle.mapper.AccountInfoMapper"><!-- 接口的全类名 -->
 <!--批量插入工作日-->
 <insert id="batchInsertAccountInfo" parameterType="java.util.List">
 INSERT INTO ACCOUNT_INFO(ID, USERNAME,PASSWORD,GENDER, EMAIL,CREATE_DATE)
 ( 
     <foreach collection="list" index="" item="accountInfo" separator="union all">
         select #{accountInfo.id},#{accountInfo.userName},#{accountInfo.password},
                #{accountInfo.gender},#{accountInfo.email},#{accountInfo.createDate}
         from dual
     </foreach>
 )
 </insert>
 
 <!-- 批量插入清算对账 oracle  防止null，加jdbcType。或者在mybatis的setting中设置jdbcTypeForNull-->
  <insert id="batchInsertBkCompList" parameterType="java.util.List">
     <foreach collection="list" index="index" item="item" separator=";" open="begin" close=";end;">
         insert into t_bkcomp(trans_date,to_host_serial,bk_serial,host_trans_code,amt,curr_type,bank_acc)values
         (#{item.transDate, jdbcType=INTEGER},
          #{item.toHostSerial, jdbcType=VARCHAR},
          #{item.bkSerial, jdbcType=VARCHAR},
          #{item.hostTarnsCode, jdbcType=VARCHAR},
          #{item.amt, jdbcType=DECIMAL},
          #{item.currType, jdbcType=VARCHAR},
          #{item.bankAcc, jdbcType=VARCHAR})
     </foreach>
 </insert>
</mapper>
```

```xml
<insert id="addBatch" parameterType="java.util.List">
        BEGIN
        <foreach collection="list" item="item" index="index" separator="">
            insert into blacklist
            (id, userid, deviceid, createdate, updatedate, "LEVEL")
            VALUES
            (
            USER_INFO_SEQ.NEXTVAL,#{item.userId,jdbcType=INTEGER},#{item.deviceId,jdbcType=VARCHAR},
            #{item.createDate,jdbcType=DATE},sysdate, #{item.level,jdbcType=INTEGER} );
        </foreach>
        COMMIT;
        END;
    </insert>
```

#### 分页

```sql
---https://my.oschina.net/Sheamus/blog/389358
SELECT *

  FROM (SELECT tt.*, ROWNUM AS rowno

          FROM (  SELECT t.*

                    FROM emp t

                   WHERE hire_date BETWEEN TO_DATE ('20060501', 'yyyymmdd')

                                       AND TO_DATE ('20060731', 'yyyymmdd')

                ORDER BY create_time DESC, emp_no) tt

         WHERE ROWNUM < 20) table_alias

 WHERE table_alias.rowno >= 10
```



### mysql

```xml
<!-- mysql的批量插入方式 -->
<insert id="simpleInsertUserData" parameterType="java.util.List">
    INSERT INTO puser
      (userId, username, password, address, sex)
    VALUES
    <foreach collection ="list" item="item" index= "index" separator =",">
        (
        #{item.userId},#{item.username},#{item.password},
        #{item.address},#{item.sex}
        )
    </foreach>
</insert>
```

