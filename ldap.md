������Ŀ������ԭ�򣬵�½ϵͳ��Ҫ�Խ�xxx��ldap��������ʹ��openDS����openLDAP���ֶ�������������ΪDirectory Service��

���ｫһЩ����Ϊ��Ҫ�ĵط���������


1.Spring xml����

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20171212110048.png)

����base���Լ������baseOrgUnit���Ǵ������󣬿������ΪĿ¼�㼶��


2.ldap�û�У��

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20171212141918.png)

�������֤�����������˺ܶ࣬��һ��������ΪString���Բ�д��Ҳ����������Name������LdapUtils.newLdapName(dn)������

�˴���dnΪdistinguishedName���ڴ�ϵͳ�У���Ա����İ���Ϣ��������ԡ��ڶ��������ǹ��������������ǰ����û����ض����ԣ���

�������������������ˣ������������ԭ���룬���Խ���client��server�����ܽ��ܣ�Ŀǰʹ�õ���RSA����


3.��ȡgroup��user�б�

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20171212145554.png)

����IMConstants.baseOrgUnitΪgroup�ľ�������Ŀ¼������OU=C,OU=B,OU=A����Ϊbase/A/B/C��

![Alt text](https://raw.githubusercontent.com/hdlytoolazy/java/master/raw/Screenshots/20171212150403.png)

����IMConstants.baseOUs�Ƕ��OU���������Ϊ�Ӷ��Ŀ¼�����û��б�


���ڴ˴���ldap��xxx��������ֻ�ܡ��顱�����ܡ���ɾ�ġ���������ʱ��Ὣ����ɾ�ġ�ʾ����˵������ϸ��¼��
