**online-schema-change**是percona推出的一个针对mysql在线ddl的工具

官网下载地址为：http://www.percona.com/redir/downloads/percona-toolkit/2.2.1/percona-toolkit-2.2.1.tar.gz



附：

mysql 5.6之前在线ddl(加字段、加索引等修改表结构之类的操作）过程如下：

 A.对表加锁(表此时只读)

B.复制原表物理结构

C.修改表的物理结构

D.把原表数据导入中间表中，数据同步完后，锁定中间表，并删除原表

E.rename中间表为原表

F.刷新数据字典，并释放锁

