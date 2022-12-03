建表
```sql
CREATE TABLE IF NOT EXISTS `runoob_tbl`(
   `runoob_id` INT UNSIGNED AUTO_INCREMENT,
   `runoob_title` VARCHAR(100) NOT NULL,
   `runoob_author` VARCHAR(40) NOT NULL,
   `submission_date` DATE,
   PRIMARY KEY ( `runoob_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

搜索重复字段
```sql
select distinct ROLEID,USERID, ROLETYPE, count(*) from FINE_USER_ROLE MIDDLE group by ROLEID, USERID, ROLETYPE
having count(*) >1
```

