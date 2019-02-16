---
title: 分布式方案下Session实现一致性的四种方法
tags: [session,分布式,Nginx]
grammar_cjkRuby: true
categories: [服务端]
date: 2019-02-16
---


### 方案一：session复制
 

思路：多个web-server之间相互同步session，这样每个web-server之间都包含全部的session

![enter description here](./images/1550325346670.png)

**优点：**
- web-server(Tomcat）支持的功能，应用程序不需要修改代码

**不足：**

- session的同步需要数据传输，占内网带宽，有时延

- 所有web-server都包含所有session数据，数据量受内存限制，无法水平扩展

有更多web-server时要歇菜

 

### 方案二：客户端存储法 


思路：服务端存储所有用户的session，内存占用较大，可以将session存储到浏览器cookie中，每个端只要存储一个用户的数据了

![enter description here](./images/1550325392877.png)

**优点：**
- 服务端不需要存储

**缺点：**

- 每次http请求都携带session，占外网带宽

- 数据存储在端上，并在网络传输，存在泄漏、篡改、窃取等安全隐患

- session存储的数据大小受cookie限制

 

### 方案三：反向代理hash一致性

 

#### ip hash方案

反向代理层使用用户ip来做hash，以保证同一个ip的请求落在同一个web-server上

``` nginxconf
upstream sesiontest {
    ip_hash;
    server 192.168.11.115:8080;
    server 192.168.11.120:8080; 
}
```

 **优点：**
- 只需要改nginx配置，不需要修改应用代码

- 负载均衡，只要hash属性是均匀的，多台web-server的负载是均衡的

- 可以支持web-server水平扩展（session同步法是不行的，受内存限制）

**不足：**
- web-server水平扩展，rehash后session重新分布，也会有一部分用户路由不到正确的session

#### url hash方案

反向代理使用http协议中的某些业务属性来做hash，例如sid，city_id，user_id等，能够更加灵活的实施hash策略，以保证同一个浏览器用户的请求落在同一个web-server上
``` nginxconf
upstream www_load {
	    #根据参数进行Hash分配
		hash $defurlkey;
		server localhost:5000;
		server localhost:5001;
		server localhost:5002;
		}
```
**优点：**
- 只需要改nginx配置，不需要修改应用代码

- 负载均衡，只要hash属性是均匀的，多台web-server的负载是均衡的

- 可以支持web-server水平扩展（session同步法是不行的，受内存限制）

**不足：**
- web-server水平扩展，rehash后session重新分布，也会有一部分用户路由不到正确的session

 

### 方案四：后端统一集中存储
 

思路：将session存储在web-server后端的存储层，数据库或者缓存

![enter description here](./images/1550325803329.png)

```xml
<!-- Spring-Session+Redis实现session共享依赖 start--> 
<dependency>
    <groupId>org.springframework.session</groupId>
	<artifactId>spring-session-data-redis</artifactId> 
	<version>1.2.1.RELEASE</version>
</dependency>

<dependency>
    <groupId>redis.clients</groupId> 
    <artifactId>jedis</artifactId>
    <version>2.8.1</version> 
</dependency> 
<!-- Spring-Session+Redis实现session共享依赖 end-->
```

**优点：**

- 没有安全隐患

- 可以水平扩展，数据库/缓存水平切分即可

- web-server重启或者扩容都不会有session丢失

**不足：**
- 增加了一次网络调用，并且需要修改应用代码


### 结论
- 优选方案三
- 其次方案四
- 不用方案二
- 少用方案一