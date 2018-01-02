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

