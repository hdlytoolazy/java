项目中有上传文件的功能，文件放在webapps的子目录下，本来一个不是很麻烦的事情缺有一个大坑。

每次项目重启的时候，之前上传的文件就会莫名其妙的消失，这个问题实在是很蛋疼。后来定位问题发现其实在tomcat停止的时候，

文件就已经消失了。后来经过不懈的努力，终于发现了问题所在。

file.deleteOnExit()，API内容为：

Requests that the file or directory denoted by this abstract pathname be deleted when the virtual machine terminates. Files (or directories) are deleted in the reverse order that they are registered. Invoking this method to delete a file or directory that is already registered for deletion has no effect. Deletion will be attempted only for normal termination of the virtual machine, as defined by the Java Language Specification.
Once deletion has been requested, it is not possible to cancel the request. This method should therefore be used with care.

Note: this method should not be used for file-locking, as the resulting protocol cannot be made to work reliably. The FileLock facility should be used instead.

在JVM进程退出时就删除文件！！！

从别人手里接手代码总会有坑，但一定要记住这个问题，熟悉API才是王道。