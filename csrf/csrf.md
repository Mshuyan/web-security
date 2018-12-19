# csrf（跨站请求伪造）

## 介绍及security防御策略

> 以下内容摘自[Mshuyan/security](https://github.com/Mshuyan/security/blob/master/spring4all/README.md#csrf) 

- 什么是CSRF攻击

  > 资料参见：[浅谈CSRF攻击方式](https://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html) 

  - CSRF攻击全称`跨站请求攻击`

    当用户同时打开两个网站的网页A、B；A网页登录后产生cookie存放于浏览器；此时操作B网页，如果网页B是来自于1个黑客的网站，该网页上包含了请求网页A的代码，则用户在网页B上的操作很有可能触发这个发往网页A的服务器的请求，此时浏览器会自动携带前面产生的cookie，此时网页B就可以伪装成用户在网站A上进行操作了

  - 防范措施

    网页B只能向网站A发送请求，携带cookie是浏览器携带的，网页B本身并不能拿到网页A的coockie，基于这点，可以在网页A的表单中添加1个隐藏的参数，该参数中的值就是将cookie中的值，在服务端对该值进行验证。

- `spring security`中针对`CSRF`攻击的处理

  - 请求方式

    security默认仅对`GET`、`HEAD`、`TRACE`、`OPTIONS`不进行`csrf`攻击验证，其他的请求均会进行验证

  - 处理方式

    > 参见：[spring security的跨域保护(CSRF Protection)](https://www.jianshu.com/p/672b6390c25f) 

    security会在后端生成1个用于校验`csrf`攻击的`cookie`字符串，存储在session中，然后将该字符串以**直接设置到页面**或**设置到request的cookie中（cookie名称：XSRF-TOKEN）**的方式返回给前端，下次请求时会校验**表单参数（_csrf）**或**请求头（X-XSRF-TOKEN）**的值是否与session保存的值相同，来防止csrf攻击。

  - 默认实现方式

    默认情况下，security在生成名为`XSRF-TOKEN`的cookie时，进行了两种处理：

    - 通过`request.setAttribute`方法设置到request的属性中，这样jsp页面就可以轻松的获得这个值；
    - 将名为`XSRF-TOKEN`的cookie的`cookieHttpOnly`属性设置为`true`，以防止XSS攻击

    security自动生成的登录页面就是通过`request.getAttribute`获取这个属性值并设置到form中，然后将页面返回给前端的。

    但是通过`getAttribute`获取该值有1个限制，必须使用jsp，而对于前后端分离来说，并不适用。

- 从cookie中获取`XSRF-TOKEN`

  由于security的名为`XSRF-TOKEN`的cookie的`cookieHttpOnly`属性默认为`true`，所以需要先在后端将该属性设置为`false`，js才能从cookie中获取`XSRF-TOKEN`

  后端配置代码如下：

  ```java
  protected void configure(HttpSecurity http) throws Exception {
  	http.csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
  }
  ```

  然后就可以通过如下任意一种方式通过csrf验证了

  - 通过form表单处理

    将从cookie中获取的`XSRF-TOKEN`的值设置到from表单中，该值对应的参数名称为`_csrf`

  - 通过请求头处理

    将从cookie中获取的`XSRF-TOKEN`的值设置到请求头中，该值对应的请求头名称为`X-XSRF-TOKEN`

- `CookieCsrfTokenRepository`

  > 该实现类是通过cookie方式防止csrf攻击，方案是将浏览器发来的请求中的表单参数或请求头，与自己下发的cookie进行比对，相等则通过

## 问题整理

### 同源策略并不能防止CSRF攻击

> 同源策略有一个限制：仅能限制`ajax`请求
>
> 如果攻击网站页面上直接嵌入1个form表单，向要攻击的网站发送请求，则同源策略是无法制止的

### jwt请求头方式防止csrf攻击

- 因为csrf攻击主要是利用了浏览器向攻击网站发送请求时自动携带cookie的特性，来进行伪造身份的，所以只要我们避免通过cookie来识别身份，一样也可以防止csrf攻击

- 根据这个原理，我们在使用jwt时，将jwt放在请求头中，服务端从请求头中获取到jwt，并根据jwt中信息来识别用户身份，这样一样可以防止csrf攻击