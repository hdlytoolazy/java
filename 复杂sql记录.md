һ����ĳһ�������������ͳ��

����SELECT COUNT(*) FROM (SELECT COUNT(*) FROM t_collect_user GROUP BY professor_id) AS temp


��������ѯ����A�ֶεĽ�������з����Ұ��������е��򣬻���B�ֶ�����

SELECT tcu.* , COUNT(*) AS fansNum , (SELECT display_name FROM t_ldap_user WHERE id = tcu.professor_id) AS displayName
FROM t_collect_user tcu
GROUP BY professor_id
ORDER BY COUNT(professor_id) DESC , displayName


��������ѯ���������ֶΰ���һ����ʽ��ֵ

ALTER TABLE t_user_integration_record 
MODIFY COLUMN score DOUBLE(20,1) COMMENT '��������';

ALTER TABLE t_user_integration 
MODIFY COLUMN user_integral DOUBLE(20,1) COMMENT '�û��ܻ���' DEFAULT 0;

ALTER TABLE t_user_integration 
ADD COLUMN honor_integral DOUBLE(20,1) COMMENT '��������' DEFAULT 0;

ALTER TABLE t_user_integration 
ADD COLUMN activity_integral DOUBLE(20,1) COMMENT '�����' DEFAULT 0;

ALTER TABLE t_user_integration 
ADD COLUMN professor_integral DOUBLE(20,1) COMMENT 'ר�һ���' DEFAULT 0;

ALTER TABLE t_user_integration 
ADD COLUMN flow_integral DOUBLE(20,1) COMMENT '���ʻ���' DEFAULT 0;

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


�ģ�����ѯ����ĳ���ֶ�������

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
	
	
�壬���ٲ���100W����������
��ý���ʱʹ��MyISAM�������ٸ�ΪInnoDB����������ʱ��ܶ�
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


����ǧ�����ݲ�ѯ�Ż�����
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

��ʵִ��ʱ��Ϊ11s���ң����Կ����൱������

1��ʹ���Ӳ�ѯ�Ż�

EXPLAIN SELECT * FROM t_user WHERE
id>=(SELECT id FROM t_user WHERE NAME > '1000000000@qq.com' LIMIT 1000000,1) 
LIMIT 10;

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20180417183620.png)

��ʵִ��ʱ�䲻��0.1s�����Կ����Ѿ������˲�ֹһ���������Ĳ�ѯ�ٶȡ����ַ�ʽ�ȶ�λƫ��λ�õ� id��Ȼ�������ѯ��
���ַ�ʽ������ id �����������

2��ʹ�� id �޶��Ż�

���ַ�ʽ�������ݱ��id�����������ģ������Ǹ��ݲ�ѯ��ҳ���Ͳ�ѯ�ļ�¼�����������ѯ��id�ķ�Χ��
����ʹ�� id between and ����ѯ,�����ܹ��ڼ�ʮ��������ɡ�

EXPLAIN SELECT * FROM t_user WHERE id BETWEEN x1 AND x2 AND NAME > '1000000000@qq.com' LIMIT 10


3��ʹ����ʱ���Ż�

����ʹ�� id �޶��Ż��е����⣬��Ҫ id �����������ģ�������һЩ�����£�����ʹ����ʷ���ʱ�򣬻��߳��ֹ�����ȱʧ����ʱ��
���Կ���ʹ����ʱ�洢�ı�����¼��ҳ��id��ʹ�÷�ҳ��id������ in ��ѯ�������ܹ��������ߴ�ͳ�ķ�ҳ��ѯ�ٶȣ�
��������������ǧ���ʱ��


һ������£������ݿ��н������ʱ��ǿ��Ϊÿһ�ű���� id �����ֶΣ����������ѯ��
������Ƕ�������������ǳ��Ӵ�һ�����зֿ�ֱ����ʱ�򲻽���ʹ�����ݿ�� id ��ΪΨһ��ʶ��
��Ӧ��ʹ�÷ֲ�ʽ�ĸ߲���Ψһ id �����������ɣ��������ݱ���ʹ��������ֶ����洢���Ψһ��ʶ��
ʹ����ʹ�÷�Χ��ѯ��λ id ��������������Ȼ����ʹ���������ж�λ���ݣ��ܹ���ߺü�����ѯ�ٶȡ�
���� select id��Ȼ���� select *��
