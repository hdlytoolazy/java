本地启动项目两种方式：tomcat  application

今天在本地启动项目时使用tomcat方式，发现始终无法logback.log文件，并且因此而导致项目不能成功启动。
exception如下：
ch.qos.logback.core.rolling.RollingFileAppender[file] openFile(./logback.log）FileNotFoundException（访问拒绝）

导致十分郁闷。经排查，之前用application方式启动时可在项目根目录中直接产生logback.log文件，
所以想到是否是权限不够，导致在本地tomcat中的临时文件中无法创建文件。
于是以管理员重新启动Idea，问题就解决了。

以后一定注意权限问题！！！