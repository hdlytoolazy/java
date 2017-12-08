关于URL中带空格问题，其实现在用的诸多框架都不需要自己处理，但是知道总比不知道好。

给测试的同事写一个demo，懒得再一顿封装，所以就用了原始的HttpRequest，URL也是直接用字符串拼起来的（String.format），

自己run的时候没问题，给出去之后发现居然报异常IllegalArgumentException，mmp。

经仔细检查，发现使用这种方式拼出的URL中带有空格，导致不能识别。

造成这种混乱局面的原因在于：W3C标准规定，当Content-Type为application/x-www-form-urlencoded时，URL中查询参数名和参数值中空格
要用加号+替代，所以几乎所有使用该规范的浏览器在表单提交后，URL查询参数中空格都会被编成加号+。而在另一份规范(RFC 2396，定义
URI)里, URI里的保留字符都需转义成%HH格式(Section 3.4 Query Component)，因此空格会被编码成%20，加号+本身也作为保留字而被编成
%2B，对于某些遵循RFC 2396标准的应用来说，它可能不接受查询字符串中出现加号+，认为它是非法字符。所以一个安全的举措是URL中统一
使用%20来编码空格字符。

所以在拼参数的时候，先使用URLEncoder.encode，再使用str.replaceAll("\\+","%20")，例如：

str = URLEncoder.encode(fileName,"UTF-8");  
str = fileName.replaceAll("\\+","%20");

将最后得到的str放进参数中就万无一失了。