一，对某一结果集进行数量统计

例：SELECT COUNT(*) FROM (SELECT COUNT(*) FROM t_collect_user GROUP BY professor_id) AS temp


二，多表查询，以A字段的结果集进行分组且按数量进行倒序，还以B字段正序

SELECT tcu.* , COUNT(*) AS fansNum , (SELECT display_name FROM t_ldap_user WHERE id = tcu.professor_id) AS displayName
FROM t_collect_user tcu
GROUP BY professor_id
ORDER BY COUNT(professor_id) DESC , displayName


三，多表查询并给新增字段按照一定方式赋值

ALTER TABLE t_user_integration_record 
MODIFY COLUMN score DOUBLE(20,1) COMMENT '操作积分';

ALTER TABLE t_user_integration 
MODIFY COLUMN user_integral DOUBLE(20,1) COMMENT '用户总积分' DEFAULT 0;

ALTER TABLE t_user_integration 
ADD COLUMN honor_integral DOUBLE(20,1) COMMENT '荣誉积分' DEFAULT 0;

ALTER TABLE t_user_integration 
ADD COLUMN activity_integral DOUBLE(20,1) COMMENT '活动积分' DEFAULT 0;

ALTER TABLE t_user_integration 
ADD COLUMN professor_integral DOUBLE(20,1) COMMENT '专家积分' DEFAULT 0;

ALTER TABLE t_user_integration 
ADD COLUMN flow_integral DOUBLE(20,1) COMMENT '提问积分' DEFAULT 0;

UPDATE t_user_integration AS tui SET honor_integral = 0 , activity_integral = 0 , professor_integral = 0 , flow_integral = 0;

UPDATE t_user_integration AS tui , t_user_integration_record AS tuir SET tui.professor_integral = (
	SELECT IFNULL(SUM(score) , 0) AS p_i FROM t_user_integration_record AS tuir WHERE TYPE IN (3,4) AND tui.user_id = tuir.user_id
) WHERE tui.user_id = tuir.user_id;

UPDATE t_user_integration AS tui , t_user_integration_record AS tuir SET tui.flow_integral = (
	SELECT IFNULL(SUM(score) , 0) AS p_i FROM t_user_integration_record AS tuir WHERE TYPE IN (2) AND tui.user_id = tuir.user_id
) WHERE tui.user_id = tuir.user_id;

UPDATE t_user_integration AS tui , t_user_integration_record AS tuir SET tui.activity_integral = (
	SELECT IFNULL(SUM(score) , 0) AS p_i FROM t_user_integration_record AS tuir WHERE TYPE IN (0,1) AND tui.user_id = tuir.user_id
) WHERE tui.user_id = tuir.user_id;

UPDATE t_user_integration AS tui SET tui.honor_integral = (
    SELECT score FROM (
	SELECT IFNULL(FORMAT(SUM(tu.`flow_integral` * 0.3 + tu.`professor_integral` * 0.7), 1) , 0) AS score ,
	user_id
	FROM t_user_integration AS tu GROUP BY user_id
    ) AS obj WHERE obj.`user_id` = tui.`user_id`
);

UPDATE t_user_integration AS tui SET tui.user_integral = (
    SELECT score FROM (
	SELECT IFNULL(SUM(tu.`flow_integral` + tu.`professor_integral` + tu.`activity_integral`) , 0) AS score ,
	user_id
	FROM t_user_integration AS tu GROUP BY user_id
    ) AS obj WHERE obj.`user_id` = tui.`user_id`
);


四，多表查询并以某个字段做排序

SELECT rank , FLOOR(honorIntegral) as honorIntegral , displayName
    FROM (
    SELECT @rownum := @rownum + 1 AS rank , displayName , honor_integral AS honorIntegral , user_id AS userId
    FROM(
    SELECT
    tui.* ,
    (SELECT display_name FROM t_ldap_user WHERE id = tui.user_id) AS displayName
    FROM t_user_integration tui , t_user_usage tuu
    WHERE tui.user_id = tuu.user_id
    <![CDATA[ AND tuu.type >= 0 ]]>
    GROUP BY tuu.user_id
    ORDER BY honor_integral DESC
    ) AS obj ,
    (SELECT @rownum := 0) r
    )
    AS objRank WHERE userId IN
    <foreach collection="list" index="index" item="userId" open="(" separator="," close=")">
      #{userId,jdbcType=INTEGER}
    </foreach>
	
	
五，快速插入100W条测试数据
最好建表时使用MyISAM，后期再改为InnoDB，这样插入时快很多
CREATE TABLE `t_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` tinyint(4) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

DROP PROCEDURE IF EXISTS proc_batch_insert;
DELIMITER //
CREATE PROCEDURE proc_batch_insert()
BEGIN
DECLARE pre_name BIGINT;
DECLARE ageVal INT;
DECLARE i INT;
SET pre_name=187635267;
SET ageVal=100;
SET i=1;
WHILE i < 1000000 DO
        INSERT INTO t_user(`name`,age,create_time,update_time) VALUES(CONCAT(pre_name,'@qq.com'),(ageVal+1)%30,NOW(),NOW());
SET pre_name=pre_name+100;
SET i=i+1;
END WHILE;
END //
 

CALL proc_batch_insert();

DELIMITER ;


六，千万级数据查询优化技巧
CREATE TABLE `t_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` tinyint(4) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

EXPLAIN SELECT * FROM t_user WHERE NAME > '1000000000@qq.com' LIMIT 1000000 , 10

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20180417183551.png)

真实执行时间为11s左右，可以看出相当的慢。

1）使用子查询优化

EXPLAIN SELECT * FROM t_user WHERE
id>=(SELECT id FROM t_user WHERE NAME > '1000000000@qq.com' LIMIT 1000000,1) 
LIMIT 10;

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20180417183620.png)

真实执行时间不到0.1s，可以看出已经提升了不止一个数量级的查询速度。这种方式先定位偏移位置的 id，然后往后查询，
这种方式适用于 id 递增的情况。

2）使用 id 限定优化

这种方式假设数据表的id是连续递增的，则我们根据查询的页数和查询的记录数可以算出查询的id的范围，
可以使用 id between and 来查询,基本能够在几十毫秒内完成。

EXPLAIN SELECT * FROM t_user WHERE id BETWEEN x1 AND x2 AND NAME > '1000000000@qq.com' LIMIT 10


3）使用临时表优化

对于使用 id 限定优化中的问题，需要 id 是连续递增的，但是在一些场景下，比如使用历史表的时候，或者出现过数据缺失问题时，
可以考虑使用临时存储的表来记录分页的id，使用分页的id来进行 in 查询。这样能够极大的提高传统的分页查询速度，
尤其是数据量上千万的时候。


一般情况下，在数据库中建立表的时候，强制为每一张表添加 id 递增字段，这样方便查询。
如果像是订单库等数据量非常庞大，一般会进行分库分表。这个时候不建议使用数据库的 id 作为唯一标识，
而应该使用分布式的高并发唯一 id 生成器来生成，并在数据表中使用另外的字段来存储这个唯一标识。
使用先使用范围查询定位 id （或者索引），然后再使用索引进行定位数据，能够提高好几倍查询速度。
即先 select id，然后再 select *；
