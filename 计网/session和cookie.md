# session和cookie

## Cookie

**给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理**。

客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie。客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。服务器还可以根据需要修改Cookie的内容。

==**cookie具有不可跨域名性**==Google只会携带Google的Cookie，并不会携带Baidu的，也不能操作Baidu的Cookie。

**如果不设置有效期，默认关闭窗口，cookie消失**

### 跨域

正常情况下，同一个一级域名下的两个二级域名如www.helloweenvsfei.com和images.helloweenvsfei.com也不能交互使用Cookie，因为二者的域名并不严格相同。如果想所有helloweenvsfei.com名下的二级域名都可以使用该Cookie，需要设置Cookie的domain参数，例如：

```
Cookie cookie = new Cookie("time","20080808"); // 新建Cookie
cookie.setDomain(".helloweenvsfei.com");           // 设置域名
cookie.setPath("/");                              // 设置路径
cookie.setMaxAge(Integer.MAX_VALUE);               // 设置有效期
response.addCookie(cookie);                       // 输出到客户端
```

## Session

客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上。这就是Session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。

==**Session是存在内存中的**==

由于会有越来越多的用户访问服务器，因此Session也会越来越多。**为防止内存溢出，服务器会把长时间内没有活跃的Session从内存删除。这个时间就是Session的超时时间。如果超过了超时时间没访问过服务器，Session就自动失效了。**

虽然Session保存在服务器，对客户端是透明的，它的正常运行仍然需要客户端浏览器的支持。这是因为Session需要使用Cookie作为识别标志。HTTP协议是无状态的，Session不能依据HTTP连接来判断是否为同一客户，因此服务器向客户端浏览器发送一个名为JSESSIONID的Cookie，它的值为该Session的id（也就是HttpSession.getId()的返回值）。Session依据该Cookie来识别是否为同一用户。

## 不同

1）存储位置不同，cookie是保存在客户端的数据；session的数据存放在服务器上

2）存储容量不同，单个cookie保存的数据小，一个站点最多保存20个Cookie；对于session来说并没有上限

3）存储方式不同，cookie中只能保管ASCII字符串；session中能够存储任何类型的数据

4）隐私策略不同，cookie对客户端是可见的；session存储在服务器上，对客户端是透明对

5）有效期上不同，cookie可以长期有效存在；session依赖于名为JSESSIONID的cookie，过期时间默认为-1，只需关闭窗口该session就会失效

不清楚：6）跨域支持上不同，cookie支持跨域名访问；session不支持跨域名访问

## 禁用cookie了怎么办

1）URL重写，就是把sessionId直接附加在URL路径的后面。

2）表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把session id传递回服务器。