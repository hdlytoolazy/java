��Ŀ�����ϴ��ļ��Ĺ��ܣ��ļ�����webapps����Ŀ¼�£�����һ�����Ǻ��鷳������ȱ��һ����ӡ�

ÿ����Ŀ������ʱ��֮ǰ�ϴ����ļ��ͻ�Ī���������ʧ���������ʵ���Ǻܵ��ۡ�������λ���ⷢ����ʵ��tomcatֹͣ��ʱ��

�ļ����Ѿ���ʧ�ˡ�����������и��Ŭ�������ڷ������������ڡ�

file.deleteOnExit()��API����Ϊ��

Requests that the file or directory denoted by this abstract pathname be deleted when the virtual machine terminates. Files (or directories) are deleted in the reverse order that they are registered. Invoking this method to delete a file or directory that is already registered for deletion has no effect. Deletion will be attempted only for normal termination of the virtual machine, as defined by the Java Language Specification.
Once deletion has been requested, it is not possible to cancel the request. This method should therefore be used with care.

Note: this method should not be used for file-locking, as the resulting protocol cannot be made to work reliably. The FileLock facility should be used instead.

��JVM�����˳�ʱ��ɾ���ļ�������

�ӱ���������ִ����ܻ��пӣ���һ��Ҫ��ס������⣬��ϤAPI����������