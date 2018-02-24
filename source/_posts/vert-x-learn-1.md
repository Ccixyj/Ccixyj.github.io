---
title:  Vert.x的数据报套接字（UDP）
date: 2017-09-30 15:21:52
tags: Vert.x
---

UDP是无连接的传输，这意味着您与远程客户端没有建立持续的连接。

所以，您发送和接收的数据包都要包含有远程的地址。

除此之外，UDP不像TCP的使用那样安全，这也就意味着不能保证发送的数据包一定会被对应的接收端（Endpoint）接收。

<!-- more -->

创建一个 `client`

```kotlin
vertx.createDatagramSocket().listen(1234,"127.0.0.1"){
        if (it.succeeded()){
            logger.info("bind success ${it.result()}")
            it.result().handler {
                logger.info("from ${it.sender()} data : ${it.data()}")
            }
        }else{
            logger.warn("bind error : ${it.cause()}")
        }
    }
```
启动后输出  
    ` 15:56:30.371 [vert.x-eventloop-thread-0] INFO main - bind success io.vertx.core.datagram.impl.DatagramSocketImpl@4f7e8c07`
说明成功了

接下来保持监听,并创建一个`server`

```kotlin
   val socket = vertx.createDatagramSocket()
    runBlocking {
        launch(CommonPool){
            repeat(10) {
                socket.send(Buffer.buffer("hello : $it"), 1234, "127.0.0.1") {
                    if (it.succeeded()) {
                        logger.info("send ok :${it.result()}")
                    } else {
                        logger.warn(it.cause())
                    }
                }
                delay(100)
            }

        }
    }

```
server端每隔100毫秒发送一段数据.执行10次.

client输出结果
```
16:04:26.003 [vert.x-eventloop-thread-0] INFO main - from 127.0.0.1:63328 data : hello : 0
16:04:26.063 [vert.x-eventloop-thread-0] INFO main - from 127.0.0.1:63328 data : hello : 1
16:04:26.167 [vert.x-eventloop-thread-0] INFO main - from 127.0.0.1:63328 data : hello : 2
...
16:04:26.783 [vert.x-eventloop-thread-0] INFO main - from 127.0.0.1:63328 data : hello : 8
16:04:26.884 [vert.x-eventloop-thread-0] INFO main - from 127.0.0.1:63328 data : hello : 9
```

并且无论client是否绑定成功,server端发送数据永远是成功的.