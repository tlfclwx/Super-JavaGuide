https://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html
# CSRF攻击
## 原理

![](https://gitee.com/super-jimwang/img/raw/master/img/20210409142244.png)


由于未关闭浏览器，cookie仍然有效，黑客通过黑客网站模拟用户的get和post请求，从而达到目的

从上图可以看出，要完成一次CSRF攻击，受害者必须依次完成两个步骤：

　　1.登录受信任网站A，并在本地生成Cookie。

　　2.在不登出A的情况下，访问危险网站B。

　　看到这里，你也许会说：“如果我不满足以上两个条件中的一个，我就不会受到CSRF的攻击”。

## CSRF防御
1. cookie hashing。表单中增加一项hash值，然后在服务端进行校验
   - 比如对'myweb'+当前时间 字符串进行hash，然后服务端进行校验
2. 验证码