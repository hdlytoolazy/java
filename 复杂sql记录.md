一，对某一结果集进行数量统计

例：SELECT COUNT(*) FROM (SELECT COUNT(*) FROM t_collect_user GROUP BY professor_id) AS temp


二，多表查询，以A字段的结果集进行分组且按数量进行倒序，还以B字段正序
SELECT tcu.* , COUNT(*) AS fansNum , (SELECT display_name FROM t_ldap_user WHERE id = tcu.professor_id) AS displayName
FROM t_collect_user tcu
GROUP BY professor_id
ORDER BY COUNT(professor_id) DESC , displayName