今天在做项目的时候，需要有一个复杂查询，原生sql为：
SELECT * FROM apk_info WHERE project_id = ? AND type_id = ? ORDER BY version_code ASC LIMIT 1


问题一：JPA的@Query中不支持limit
limit只是mysql中分页的关键字，但JPQL中是没有limit的，所以这里需要用到javax.persistence.EntityManager。

EntityManager--实体管理器
EntityManager管理实体生命周期，并公开了多个在实体上执行CRUD操作的方法。EntityManager API在事务上下文中调用。
可以在EJB容器外部（例如，从一个Web应用程序）调用它，而无需会话bean外观。具体使用如下：
Query query = entityManager.createQuery("select a from ApkInfo a where a.project = :project and a.type = :type ORDER BY a.versionCode ASC ")
                .setParameter("project", project)
                .setParameter("type", apkType)
                .setMaxResults(1);
ApkInfo apkInfo = (ApkInfo) query.getSingleResult();

若setMaxResults(n)，n>1，则可以使用query.getResultList()来返回结果集。


问题二：org.hibernate.PropertyAccessException: IllegalArgumentException
@Entity
@Data
public class ApkInfo {
    @Id
    @GeneratedValue
    Long id;
    //类型
    @OneToOne(cascade = CascadeType.ALL)
    ApkType type;
    @ApiModelProperty(value = "版本号数字10000", required = true)
    Integer versionCode;
    @OneToOne
    Project project;
}

@Entity
@Data
public class ApkType {
    @Id
    @GeneratedValue
    Long id;
    //安装包类型，android、ios、ipad
    @Column()
    Integer type;
}

开始使用select a from ApkInfo a where a.projectId = :projectId and a.typeId = :typeId ORDER BY a.versionCode ASC ，
（注：projectId是Project中属性，typeId是ApkType中属性）
报如题的错误。后来发现，虽然在数据库中是project_id，但是在此处查询时，projectId是project的属性，映射时依然需要使用
对象。最后改成select a from ApkInfo a where a.project = :project and a.type = :type ORDER BY a.versionCode ASC ，就
正常了。
