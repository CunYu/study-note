### 转发和重定向

转发是服务端内部的行为，重定向是客户端行为。

转发是服务端内部的跳转，客户端不会有感知。

重定向是客户端行为，客户端发送请求后，服务端会返回相应的状态码（301，302）并告知相关URL，客户端会发起新的请求。

转发可以隐藏内部细节，根据传参的不同，服务端内部跳转到相应的模块，重定向一般用于注销完毕后的跳转。

### GET和POST

GET是从服务端获取资源，POST是向服务端提交数据。

GET传输的数据存放在URL中，POST的数据存放在请求头或请求数据中。

GET传输的数据受URL最大长度影响（2048个字符），GET使用MIME类型（application/x-www-form-urlencoded）的URL编码（百分号编码）。

### Session和Cookie

##### 概述

Session和Cookie是用来存储Web访问过程中的相关数据的，Session存储在服务端，Cookie存储在客户端。

##### Session

Session将数据保存在服务端，只有服务端才能读取，相对Cookie来说，更加的安全，减少了网络带宽的压力，Session不能在多台服务器之间共享。

Session默认保存在服务器的文件内，也可以保存在内存，数据库中。

当Session在Session超时时间内没有收到任何请求，该Session会被删除。

##### Cookie

###### Cookie版本

* Version 0

* Version 1

两个版本的属性项有所不同，设置响应头的标识的方式也不同，Set-Cookie，Set-Cookie2。

###### Cookie注意事项

* 创建Cookie的NAME不能和Set-Cookie和Set-Cookie2的属性值一样。

* 创建Cookie的NAME和VALUE的值不能设置成非ASSIC字符，如果用中文需要进行编码。

* 当Cookie的NAME和VALUE的值出现TOKEN字符（\ , 等）时，该Cookie的Version的版本会变为1。

* 如果不设置Cookie超时时间，Cookie是保存在内存中，浏览器关闭，Cookie就消失，如果设置了Cookie超时时间，Cookie则保存在硬盘中。

* Cookie以地址进行管理，访问时只会发送该地址对应的Cookie。

* 不同的浏览器对Cookie有大小和数量的限制。

* 为了减少带宽，可以对Cookie进行压缩。