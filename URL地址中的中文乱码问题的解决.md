## URL地址中的中文乱码问题的解决

引言：在Restful类的服务设计中，经常会碰到需要在URL地址中使用中文作为参数的情况，这种情况下，一般都需要正确的设置和编码中文字符信息。乱码问题就此产生了，该如何解决呢？且听本文详细道来。

#### 问题引出

在Restful的服务设计中，查询某些信息的时候，一般的URL地址设计为：`get/basic/service?keywork=历史`之类的URL地址。但是在实际开发和使用中后台读取keyword信息为乱码，无法正确读取。

#### 乱码是如何产生的

由于我们利用URL传递参数这种方式是依赖浏览器环境的，也就是说URL及URL中包含的各个key=value格式的传递参数键值对是在浏览器地址栏中处理相应编码后传递至后台的。由于我们没有进行任何处理，此时javascript请求URL参数存在中文时，对URL中的中文参数进行编码是按照浏览器的编码机制进行编码的，所以存在乱码问题。

#### 初次编码, javascript中利用encodeURI()方法进行编码

利用encodeURI()在javascript中对中文URL参数进行编码时，“测试”二字会被转换为“%E6%B5%8B%E8%AF%95”。但是问题依然存在，原因是在编码后的字符串信息，浏览器会认为“%”是个转义字符，会把“%”与“%”之间的已转义字符进行处理传递到后台中。这会造成与实际经过encodeURI()编码后的URL不符。

#### 二次编码，使用encodeURI

操作： `encodeURI(encodeURI("/order?name=" + name));`

处理后的URL不再是通过一次encodeURI()转换后的字符串“%E6%B5%8B%E8%AF%95”，而是两层encodeRUI()处理后的字符串“%25E6%B255%258B%25E8%AF%2595”。通过再次编码，原来被浏览器解析为转义字符的“%”被再次编码，转换成了普通字符“%25”。

此时前端js代码对带有中文的URL编码已经完成，并通过URL传递参数的方式传递到后台等待处理，Action获取到正常无乱码的参数为”%25E6%B255%258B%25E8%AF%2595“，次字符串对应的中文正是我们输入的“测试”二字。

#### 后台如何正确解析中文字符信息

进入后台的信息，在经过两次encodeURI()之后，直接读取是无法获取正确信息的。需要继续如下处理：

~~~
URLDecoder.decode("chinese string","UTF-8")
~~~

URLDecoder的decode(String str, String ecn)方法有两个参数，第一个参数为待解码的字符串，第二个参数为解码时对应的编码。

#### 另外一种处理方案

请求端的中文字符用encodeURI进行一次转码，如：

~~~
var url="/ajax?name="+encodeURI(name);
~~~

服务端代码：

~~~
name=new String(name.getBytes("iso8859-1"),"UTF-8");
~~~

name为获取到的字符串，iso8859-1为项目默认字符编码，如果为中文编码gbk,gb2312等则不用这一步进行处理。

经过验证，结果可行。由此可知，浏览器本身默认的编码方式是iso8859-1，即使使用了encodeURI进行了utf-8编码处理，主要的字符串内容，比如ASCII字符和可见字符都还是基于iso8859-1浏览器自身的字符。原因就是这些字符在编马上和UTF-8字符串是重合的。而encodeURI之类的转义函数主要解决特殊字符%./的转义问题。