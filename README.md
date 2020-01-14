# Web基础
## Web工作方式
### HTTP请求包
```sh
GET http://domains/example/ HTTP/1.1		#请求行: 请求方法 请求URI HTTP协议/协议版本
Host：www.iana.org				#服务端的主机名
#浏览器信息
User-Agent：Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.4 (KHTML, like Gecko) Chrome/22.0.1229.94 Safari/537.4		
Accept：text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8	#客户端能接收的MIME
Accept-Encoding：gzip,deflate,sdch		#是否支持流压缩
Accept-Charset：UTF-8,*;q=0.5		#客户端字符编码集
#空行,用于分割“请求头”和“消息体”
#消息体,请求资源参数,例如POST传递的参数，GET包的消息体为空
#GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditPosts.aspx?name=test1&id=123456。POST方法是把提交的数据放在HTTP包的body中。
```

### HTTP响应包（服务器信息）
```sh
HTTP/1.1 200 OK						#状态行
Server: nginx/1.0.8					#服务器使用的WEB软件名及版本
Date:Date: Tue, 30 Oct 2012 04:14:25 GMT		#发送时间
Content-Type: text/html				#服务器发送信息的类型
Transfer-Encoding: chunked			#表示发送HTTP包是分段发的
Connection: keep-alive				#保持连接状态
Content-Length: 90					#主体内容长度
#空行 用来分割"消息头"和"主体"
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"... #消息体
``` 
- 状态码由三位数组成，第一个数字定义了响应的类别：
    - 1XX 提示信息-表示请求已被成功接收，继续处理
    - 2XX 成功 - 表示请求已被成功接收，理解，接受
    - 3XX 重定向 - 要完成请求必须进行更进一步的处理
    - 4XX 客户端错误 - 请求有语法错误或请求无法实现
    - 5XX 服务器端错误 - 服务器未能实现合法的请求

### 浏览器请求
一次完整浏览器URL请求会从服务器获得很多资源：
    - 第一次请求url
    - 服务器返回html页面
    - 浏览器开始渲染
    - 解析到DOM里面的图片链接，css脚本或者js脚本的链接
    - 浏览器自动发送一个静态资源的HTTP请求给服务器
    - 服务器返回静态资源给浏览器
    - 浏览器将所有资源整合，渲染到屏幕上
为了减少一次url请求的HTTP请求次数，可做的优化就是：
> 尽量减少网页请求静态资源的次数，可参考`gatsby`的做法。

## Go如何使得Web工作
* [go-web-hello.go](./code/web/hello/main.go)
* 三个问题
    - 如何监听端口？
    `ListenAndServe(addr)` -> `net.Listen(addr)` -> `listener.Accept()` -> 阻塞在一个循环中等待客户端连接
    - 如何接收客户端请求？<br>
    一个客户端连接服务器 -> 服务器阻塞解除 -> 新建一个连接：`srv.newConn(conn)` -> 开启新线程处理新连接`go conn.serve()`
    - 如何分配handler？<br>
    连接建立成功后，服务器的新新线程需要处理客户端的请求。其中，最关键的一步是要找到处理函数（如何分配handler），即所谓的`Router`。Go的HTTP包有两种方法：<br>
        - 利用默认Router-`DefaultServeMux`解析用户传来的URI，并分派到指定处理函数。
        - 利用自定义的Router解析用户传来的URI，并分派到指定处理函数。

## 自定义Router
* [custom-router.go](./code/router/custom/main.go)
可以和默认Router-`DefaultServeMux`对比一下。默认Router的优势在于从`ServeHTTP`抽出了匹配的逻辑，并和对应的处理的函数放在了一起。并且`ServeHTTP`不负责匹配的逻辑，交给了`ServeMux`。

# 表单
## 处理表单的输入

