由于项目的特殊原因，登陆系统需要对接xxx的ldap，服务器使用openDS（与openLDAP在字段上略有区别）作为Directory Service。

这里将一些自认为重要的地方记下来。


1.Spring xml配置

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20171212110048.png)

其中base，以及后面的baseOrgUnit都是从右向左，可以理解为目录层级。


2.ldap用户校验

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20171212141918.png)

这里的验证方法被重载了很多，第一个参数若为String可以不写，也可以是类型Name，是由LdapUtils.newLdapName(dn)得来，

此处的dn为distinguishedName，在此系统中，人员与组的绑定信息是这个属性。第二个参数是过滤条件（这里是按照用户的特定属性）。

第三个参数就是密码了（这里的密码是原密码，所以建议client与server做加密解密，目前使用的是RSA）。


3.获取group、user列表

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20171212145554.png)

其中IMConstants.baseOrgUnit为group的具体所在目录，例：OU=C,OU=B,OU=A，即为base/A/B/C。

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20171212150403.png)

其中IMConstants.baseOUs是多个OU，可以理解为从多个目录下拿用户列表。


鉴于此处的ldap由xxx管理，我们只能“查”，不能“增删改”，后面有时间会将“增删改”示例及说明再详细记录。
