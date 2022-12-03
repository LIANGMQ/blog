---
layout: photo
title: Nginx反向代理机制解决Ajax跨域问题
date: 2016-9-18
tags: [前端,跨域]
categories: 跨域
---
>-  这篇文章主要介绍了Nginx服务器中处理AJAX跨域请求的配置方法讲解,需要的朋友可以参考下

#### 1. 什么是跨域以及产生原因
　　跨域是指a页面想获取b页面资源，如果a、b页面的协议、域名、端口、子域名不同，或是a页面为ip地址，b页面为域名地址，所进行的访问行动都是跨域的，而浏览器为了安全问题一般都限制了跨域访问，也就是不允许跨域请求资源。

<!--more-->

跨域情况如下：
<img src="/img/1478182149955.png" alt="跨域情况解析" style="margin:0 auto;display:block;">

#### 2. 跨域的常见解决方法
目前来讲没有不依靠服务器端来跨域请求资源的技术

　　1. jsonp 需要目标服务器配合一个`callback`函数。

　　2. window.name+iframe 需要目标服务器响应`window.name`。

　　3. window.location.hash+iframe 同样需要目标服务器作处理。

　　4. html5的` postMessage+ifrme `这个也是需要目标服务器或者说是目标页面写一个`postMessage`，主要侧重于前端通讯。

　　5. CORS  需要服务器设置`header ：Access-Control-Allow-Origin`。

　　6. nginx反向代理 这个方法一般很少有人提及，但是他可以不用目标服务器配合，不过需要你搭建一个中转nginx服务器，用于转发请求。

#### 3. nginx反向代理解决跨域

　　上面已经说到，禁止跨域问题其实是浏览器的一种安全行为，而现在的大多数解决方案都是用标签可以跨域访问的这个漏洞或者是技巧去完成，但都少不了目标服务器做相应的改变，而我最近遇到了一个需求是，目标服务器不能给予我一个`header`，更不可以改变代码返回个`script`，所以前5种方案都被我否决掉。最后因为我的网站是我自己的主机，所以我决定搭建一个nginx并把相应代码部署在它的下面，由页面请求本域名的一个地址，转由`nginx代理`处理后返回结果给页面，而且这一切都是同步的。
　　
<p>
	下面直接讲解如何配置一个反向代理:
</p>

　首先找到`nginx.conf`或者`nginx.conf.default `或者是`default`里面的这部份
```
#gzip   on;
server{
	listen   80;
	server_name   localhost;

	#charset koi8-r;
	#c access_log   /var/log/host.access.log;

	location / {
		root html;
		index index.html  index.htm;
	}
}
```
　其中`server`代表启动的一个服务，`location` 是一个定位规则。
```
location /｛    #所有以/开头的地址，实际上是所有请求
	root  html     #去请求../html文件夹里的文件,其中..的路径在nginx里面有定义，安装的时候会有默认路径
	index  index.html index.htm  #首页响应地址
｝
```

　　从上面可以看出`location`是`nginx`用来路由的入口，所以我们接下来要在location里面完成我们的反向代理。
　　假如我们我们是www.a.com/html/msg.html 想请求www.b.com/api/?method=1&para=2；
　　
　　我们的ajax：
```
　var url = 'www.b.com/api/msg?method=1&para=2'；
     <br>$.ajax({
     type: "GET",
     url:url,
     success: function(res){..},
     ....
  })
```
　　上面的请求必然会遇到跨域问题，这时我们需要修改一下我们的请求url，让请求发在nginx的一个url下。
```
var url = 'www.b.com/api/msg?method=1&para=2'；
	var proxyurl ＝ 'msg?method=1&para=2'；
	//假如实际地址是 www.c.com/proxy/html/api/msg?method=1&para=2; www.c.com是nginx主机地址
	 $.ajax({
	type: "GET",
	url:proxyurl,
	success: function(res){..},
	....
})　
```
　　再在刚才的路径中匹配到这个请求，我们在location下面再添加一个location。
```
location ^~/proxy/html/{
	rewrite ^/proxy/html/(.*)$ /$1 break;
	proxy_pass http://www.b.com/;
}
```
#### <span style="color:coral;">以下做一个解释：</span>
1. '^~ /proxy/html/ '
　　就像上面说的一样是一个匹配规则，用于拦截请求，匹配任何以 `/proxy/html/`开头的地址，匹配符合以后，停止往下搜索正则。

2. rewrite ^/proxy/html/(.\*)$ /$1 break;
　　1. 代表重写拦截进来的请求，并且只能对域名后边的除去传递的参数外的字符串起作用，例如`www.c.com/proxy/html/api/msg?method=1&para=2`重写。只对`/proxy/html/api/msg`重写
　　2. rewrite后面的参数是一个简单的正则` ^/proxy/html/(.*)$ ,$1`代表正则中的第一个(),`$2`代表第二个()的值,以此类推
　　3. `break`代表匹配一个之后停止匹配。

3. proxy_pass

	** <span style="color:lightblue;">既是把请求代理到其他主机，其中` www.b.com/ `写法和` www.b.com`写法的区别如下:</span>**
	不带/
	```
	location /html/
	{
		proxy_pass http://b.com:8300;
	}
	```
	带/
	```
	location /html/
	{
		proxy_pass http://b.com:8300/;
	}
	```

上面两种配置，区别只在于proxy_pass转发的路径后是否带 “/”。

　　针对情况1，如果访问`url = server/html/test.jsp`，则被nginx代理后，请求路径会便问`proxy_pass/html/test.jsp`，将`test/ `作为根路径，请求test/路径下的资源。

　　针对情况2，如果访问`url = server/html/test.jsp`，则被nginx代理后，请求路径会变为 `proxy_pass/test.jsp`，直接访问server的根资源。
　　
修改配置后重启nginx代理就成功了。

*<p style="text-align:right;"> 转自：http://www.cnblogs.com/gabrielchen/p/5066120.html <p>*
