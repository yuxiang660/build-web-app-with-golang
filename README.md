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
*[form-input](./code/form/input/main.go)
    - `login.gtpl`是Go的HTML模板，用于动态生成静态网页（接收go文件传过来的各种内容）
    - `r *http.Request`中存了所有用户传过来的所有信息
    - `request.Form`是一个`url.Values`的类型，在掉了`r.ParseForm`后，里面的内容就初始化了

## 验证表单的输入
* [form-verify](./code/form/verify/main.go)
    - 感觉大部分的用户输入检查应该在前端都处理了比较好

## 处理文件上传
* [form-file-upload](./code/form/file/main.go)
    - 表单中的两个属性：
        - `enctype="multipart/form-data"`
        - `type="file"`
    - 用`r.FormFile`获取文件句柄

# 访问数据库
## database/sql接口
### sql.go中定义的接口或者函数
* `Register`函数，一般在驱动的`init`函数执行，注册驱动名称
### driver.go中定义的接口或函数
* `Driver`接口，定义了`Open`接口函数，返回一个`Conn`
* `Conn`接口，定义了数据库操作的前期和后期的接口动作，其中
    - `Prepare`会返回一个准备好的`Stmt`
    - `Begin`会返回一个事务`Tx`
* `Stmt` 接口，操作数据库，包括
    - `Exec`函数（update/insert数据库），返回`Result`数据
    - `Query`函数（select数据库），返回`Rows`结果
* `Tx`接口，定义了递交和回滚接口函数
* `Execer`接口，只有一个Exec接口，如果没有定义，就按照上面的接口执行，`Prepare`返回`Stmt`，然后执行`Stmt`的`Exec`，然后关闭`Stmt`。
* `Result`接口，是`Exec`返回的对象，定义了
    - `LastInsertId`接口函数，返回插入操作得到的自增ID。
    - `RowsAffected`接口函数，返回查询操作影响的数据条目数。
* `Rows`接口，返回
    - `Columns`接口函数，返回查询数据库表的字段信息
    - `Close`接口函数，关闭`Rows`迭代器
    - `Next`接口函数，返回下一条数据
* `Vaule`接口，是一个空接口，可以容纳任何数据，但是必须是driver能操作的常见类型
* `ValueConverter`接口，定义了如何把一个普通的值转化成`Value`的接口

## 使用MySQL数据库
* [mysql-main-operation](./code/database/mysql/main.go)
* 启动示例代码之前需要做的准备工作
    - 按照mysql
    - 创建用户`ben`，密码是`123456`
    - 创建数据库`test`
    - 用以下命令在数据库`test`下创建两个表单
    ```sql
    CREATE TABLE `userinfo` (
	    `uid` INT(10) NOT NULL AUTO_INCREMENT,
	    `username` VARCHAR(64) NULL DEFAULT NULL,
	    `department` VARCHAR(64) NULL DEFAULT NULL,
	    `created` DATE NULL DEFAULT NULL,
	    PRIMARY KEY (`uid`)
    );

    CREATE TABLE `userdetail` (
	    `uid` INT(10) NOT NULL DEFAULT '0',
	    `intro` TEXT NULL,
	    `profile` TEXT NULL,
	    PRIMARY KEY (`uid`)
    );
    ```
* `sql.Open()`函数，支持如下格式：
    ```
    user@unix(/path/to/socket)/dbname?charset=utf8
    user:password@tcp(localhost:5555)/dbname?charset=utf8
    user:password@/dbname
    user:password@tcp([de:ad:be:ef::ca:fe]:80)/dbname
    ```<br>
    需要注意，如果密码错误Open动作不会失败，后续操作才会失败。
* `sql.Prepare()`函数准备要执行的sql操作
* `stmt.Exec()`函数用来执行`stmt`准备好的SQL语句
* `db.Query()`函数用来直接执行Sql操作，并返回Rows结果

## 使用Beego orm库进行ORM开发
* [begoo-orm-mysql](./code/database/orm/main.go)

## NOSQL数据库操作
### Redis
* [go-redis](./code/database/redis/main.go)
    - `Pool *redis.Pool`是一个redis连接的线程池，每次`Pool.Get()`会新建一个连接
    - `Pool`的`Close`放在`Init`里面利用`chan`实现，很巧妙
    - `Get("test")`其中的`test`是redis一个默认的key，会返回`It's working!`.
### Mongodb
* [go-mongodb](./code/database/mongodb/main.go)
    - 在执行此实例代码前，请先安装`mongodb`。在ubuntu上安装好mongodb后，可以直接无账号密码登录，默认端口号是`:27017`
    - 执行实例代码后，会往`mongodb`中加入两条document，可以用以下命令查看：<br>
    ```
    > mongo
    mongo> show dbs
    mongo> use test
    mongo> db.people.find()
    ```

# session和数据存储
## Cookie
* 客户端机制保存用户信息，一起发送给服务器
* cookie分为：
    - 会话cookie，浏览器关闭就消失了
    - 永久cookie，只要没有超时，一直有效
* Go设置cookie
往reponse写入cookie
```
expiration := time.Now()
expiration = expiration.AddDate(1, 0, 0)
cookie := http.Cookie{Name: "username", Value: "astaxie", Expires: expiration}
http.SetCookie(w, &cookie)
```
* Go读取cookie
通过request获取cookie
```
for _, cookie := range r.Cookies() {
	fmt.Fprint(w, cookie.Name)
}
```
## Go如何使用session
* 服务器端的机制
    - 生成全局唯一标识sessionid
    - 开辟数据存储空间，永久化session
    - 将session的全局唯一标识符发送给客户端，两种方法：cookie和URL重写
        - cookie方法：设置Set-cookie头，并设置会话cookie
        - URL重写：在返回给用户的页面里的URL后面追加session标识符
* Go实现session管理
    - `SessionManager`: 用与Session的管理，连接用户和服务器存储的session
        - `SessionStart`
    - `Session`: 里面维护了一个`map`，用户拿到自己的session后，可以操作这个`map`
    - `Provider`：存储`Session`的接口，隐藏在底层，可以用不同是数据库实现

