# Redis

NoSQL(Not Only SQL)，指的是非关系型的数据库。随着Web2.0的兴起，传统的关系数据库在应付Web2.0网站，特别是超大规模和高并发的SNS类型的Web2.0纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。

而Go语言作为21世纪的C语言，对NoQL的支持也是很好，目前流行的NoSQL主要有redis、mongoDB、Cassandra和Membase等。这些数据库都有高性能、高并发读写等特点，目前已经广泛应用于各种应用中。



redis是一个key-value存储系统，类似还有Memcached。它支持存储的value类型相对更多，包括字符串（strings）、哈希（hashes）、列表（lists）、集合（sets）、带范围查询的排序集合（sorted sets）、位图（bitmaps）、hyperloglogs、带半径查询和流的地理空间索引等数据结构（geospatial indexes）。

## 安装

本地docker启动redis server实例

```bash
docker run --name redis507 -p 6379:6379 -d redis:5.0.7
```

本地启动redis-cli连接上⾯的Redis server

```bash
docker run -it --network host --rm redis:5.0.7 redis-cli
```

## 连接

Conn接口是与Redis协作的主要接口，可以使用`Dial,DialWithTimeout或者NewConn`函数来创建连接，当任务完成时，应用程序必须调用Close函数来完成操作。

新建go文件(demo10_redis_open.go)，示例代码：

```go
package main

import (
	"fmt"
	"github.com/gomodule/redigo/redis"
)

func main() {
	conn, err := redis.Dial("tcp", "127.0.0.1:6379")
	if err != nil {
		fmt.Println("connect redis error :", err)
		return
	}
	fmt.Println("connect success ...")
	defer conn.Close()
}
```

##  命令操作

通过使用Conn接口中的do方法执行redis命令，常用redis命令：http://doc.redisfans.com/

go中发送与响应对应类型：

Do函数会必要时将参数转化为二进制字符串

| Go Type         | Conversion                          |
| :-------------- | :---------------------------------- |
| []byte          | Sent as is                          |
| string          | Sent as is                          |
| int, int64      | strconv.FormatInt(v)                |
| float64         | strconv.FormatFloat(v, 'g', -1, 64) |
| bool            | true -> "1", false -> "0"           |
| nil             | ""                                  |
| all other types | fmt.Print(v)                        |

Redis 命令响应会用以下Go类型表示：

| Redis type    | Go type                                    |
| :------------ | :----------------------------------------- |
| error         | redis.Error                                |
| integer       | int64                                      |
| simple string | string                                     |
| bulk string   | []byte or nil if value not present.        |
| array         | []interface{} or nil if value not present. |

可以使用Go的类型断言或者reply辅助函数将返回的interface{}转换为对应类型。

###  get、set操作

新建go文件(demo11_redis_getset.go)，示例代码：

```go
package main

import (
    "github.com/gomodule/redigo/redis"
    "fmt"
)

func main()  {
    conn,err := redis.Dial("tcp","127.0.0.1:6379")
    if err != nil {
        fmt.Println("connect redis error :",err)
        return
    }
    fmt.Println("connect success ...")
    defer conn.Close()
    _, err = conn.Do("SET", "name", "lianshi")
    if err != nil {
        fmt.Println("redis set error:", err)
    }
    name, err := redis.String(conn.Do("GET", "name"))
    if err != nil {
        fmt.Println("redis get error:", err)
    } else {
        fmt.Printf("Got name: %s \n", name)
    }
}
```

### 设置key过期时间

示例代码：

```go
_, err = conn.Do("expire", "name", 10) // 10秒过期
if err != nil {
    fmt.Println("set expire error: ", err)
    return
}
```

### 批量获取mget、批量设置mset

新建go文件(demo12_redis_mgetmset.go)，示例代码：

```go
package main

import (
    "github.com/gomodule/redigo/redis"
    "fmt"
    "reflect"
)

func main()  {
    conn,err := redis.Dial("tcp","127.0.0.1:6379")
    if err != nil {
        fmt.Println("connect redis error :",err)
        return
    }
    fmt.Println("connect success ...")
    defer conn.Close()
    _, err = conn.Do("MSET", "name", "lianshi","age", 18)
    if err != nil {
        fmt.Println("redis mset error:", err)
    }
    res, err := redis.Strings(conn.Do("MGET", "name","age"))
    if err != nil {
        fmt.Println("redis get error:", err)
    } else {
        res_type := reflect.TypeOf(res)
        fmt.Printf("res type : %s \n", res_type)
        fmt.Printf("MGET name: %s \n", res)
        fmt.Println(len(res))
    }
    //结果：
    //connect success ...
    //res type : []string
    //MGET name: [lianshi 18]
    //2
}
```

### 列表操作

新建go文件(demo13_redis_list.go)，示例代码：

```go
package main

import (
    "github.com/gomodule/redigo/redis"
    "fmt"
    "reflect"
)

func main() {
    conn, err := redis.Dial("tcp", "127.0.0.1:6379")
    if err != nil {
        fmt.Println("connect redis error :", err)
        return
    }
    fmt.Println("connect success ...")
    defer conn.Close()
    _, err = conn.Do("LPUSH", "list1", "ele1", "ele2", "ele3", "ele4")
    if err != nil {
        fmt.Println("redis mset error:", err)
    }
    //res, err := redis.String(conn.Do("LPOP", "list1"))//获取栈顶元素
    //res, err := redis.String(conn.Do("LINDEX", "list1", 3)) //获取指定位置的元素
    res, err := redis.Strings(conn.Do("LRANGE", "list1", 0, 3)) //获取指定下标范围的元素
    if err != nil {
        fmt.Println("redis POP error:", err)
    } else {
        res_type := reflect.TypeOf(res)
        fmt.Printf("res type : %s \n", res_type)
        fmt.Printf("res  : %s \n", res)
    }
    //结果：
    //  connect success ...
    //  res type : string
    //  res  : [ele4 ele3 ele2 ele1]
}
```

### hash操作

新建go文件(demo14_redis_hash.go)，示例代码：

```go
package main

import (
    "github.com/gomodule/redigo/redis"
    "fmt"
    "reflect"
)

func main() {
    conn, err := redis.Dial("tcp", "127.0.0.1:6379")
    if err != nil {
        fmt.Println("connect redis error :", err)
        return
    }
    fmt.Println("connect success ...")
    defer conn.Close()
    _, err = conn.Do("HSET", "user","name", "lianshi","age",18)
    if err != nil {
        fmt.Println("redis mset error:", err)
    }
    res, err := redis.Int64(conn.Do("HGET", "user","age"))
    if err != nil {
        fmt.Println("redis HGET error:", err)
    } else {
        res_type := reflect.TypeOf(res)
        fmt.Printf("res type : %s \n", res_type)
        fmt.Printf("res  : %d \n", res)
    }
    //结果：
    //  connect success ...
    //  res type : int64
    //res  : 18
}
```

### Pipeline(管道)

管道操作可以理解为并发操作，并通过`Send()，Flush()，Receive()`三个方法实现。客户端可以使用send()方法一次性向服务器发送一个或多个命令，命令发送完毕时，使用flush()方法将缓冲区的命令输入一次性发送到服务器，客户端再使用`Receive()`方法依次按照**先进先出**的顺序读取所有命令操作结果。

```go
Send(commandName string, args ...interface{}) error
Flush() error
Receive() (reply interface{}, err error)
```

- Send：发送命令至缓冲区
- Flush：清空缓冲区，将命令一次性发送至服务器
- Recevie：依次读取服务器响应结果，当读取的命令未响应时，该操作会阻塞。

新建go文件(demo15_redis_pipelining.go)，示例代码：

```go
package main

import (
    "github.com/gomodule/redigo/redis"
    "fmt"
)

func main() {
    conn, err := redis.Dial("tcp", "127.0.0.1:6379")
    if err != nil {
        fmt.Println("connect redis error :", err)
        return
    }
    fmt.Println("connect success ...")
    conn.Send("HSET", "user","name", "lianshi","age","30")
    conn.Send("HSET", "user","sex","female")
    conn.Send("HGET", "user","age")
    conn.Flush()

    res1, err := conn.Receive()
    fmt.Printf("Receive res1:%v \n", res1)
    res2, err := conn.Receive()
    fmt.Printf("Receive res2:%v\n",res2)
    res3, err := conn.Receive()
    fmt.Printf("Receive res3:%s\n",res3)
    //结果：
    //  connect success ...
    //  res type : string
    //  res  : [ele4 ele3 ele2 ele1]
}
```

### 发布/订阅

不推荐，推荐 rmbq，kafka

视频直播常用



redis本身具有发布订阅的功能，其发布订阅功能通过命令`SUBSCRIBE(订阅)／PUBLISH(发布)`实现，并且发布订阅模式可以是多对多模式还可支持正则表达式，发布者可以向一个或多个频道发送消息，订阅者可订阅一个或者多个频道接受消息。

操作示例，示例中将使用两个goroutine分别担任发布者和订阅者角色进行演示：

新建go文件(demo16_redis_pub_sub.go)：

```go
package main

import (
    "github.com/gomodule/redigo/redis"
    "fmt"
    "time"
)

func Subs() { //订阅者
	conn, err := redis.Dial("tcp", "127.0.0.1:6379")
	if err != nil {
		fmt.Println("connect redis error :", err)
		return
	}
	defer conn.Close()
	psc := redis.PubSubConn{conn}
	psc.Subscribe("channel1") //订阅channel1频道
	for {
		switch v := psc.Receive().(type) {
		case redis.Message:
			fmt.Printf("%s: message: %s\n", v.Channel, v.Data)
		case redis.Subscription:
			fmt.Printf("%s: %s %d\n", v.Channel, v.Kind, v.Count)
		case error:
			fmt.Println(v)
			return
		}
	}
}

func Push(message string) { //发布者
	conn, _ := redis.Dial("tcp", "127.0.0.1:6379")
	_, err1 := conn.Do("PUBLISH", "channel1", message)
	if err1 != nil {
		fmt.Println("pub err: ", err1)
		return
	}

}

func main() {
	go Subs()
	go Push("this is lianshi")
	time.Sleep(time.Second * 5)
}
// channel1: subscribe 1
// channel1: message: this is lianshi
```

### 事务操作

`MULTI, EXEC,DISCARD,WATCH`是构成Redis事务的基础，当然我们使用go语言对redis进行事务操作的时候本质也是使用这些命令。

- `MULTI`：开启事务
- `EXEC` ：执行事务
- `DISCARD`：取消事务
- `WATCH`：监视事务中的键变化，一旦有改变则取消事务。

#### MULTI操作

新建go文件(demo17_redis_transaction.go)，示例代码：

```go
package main

import (
    "github.com/gomodule/redigo/redis"
    "fmt"
)

func main()  {
    conn, err := redis.Dial("tcp", "127.0.0.1:6379")
    if err != nil {
        fmt.Println("connect redis error :", err)
        return
    }
    fmt.Println("connect success ...")
    defer conn.Close()

    conn.Send("MULTI")
    conn.Send("INCR", "foo")
    conn.Send("INCR", "bar")
    r, err := conn.Do("EXEC")
    fmt.Println(r)
}
//connect success ...
//[1 1]
```

#### watch操作

WATCH命令可以监控一个或多个键,一旦其中有一个键被修改(或删除),之后的事务就不会执行。

```go
package main

import (
    "github.com/gomodule/redigo/redis"
    "fmt"
)

func main()  {
    conn, err := redis.Dial("tcp", "127.0.0.1:6379")
    if err != nil {
        fmt.Println("connect redis error :", err)
        return
    }
    fmt.Println("connect success ...")
    defer conn.Close()

	conn.Do("SET", "xo", 10)
	conn.Do("WATCH", "xo")
	v, _ := redis.Int64(conn.Do("GET", "xo"))
	v = v + 1 // 这里可以基于值做一些判断逻辑
	conn.Send("MULTI")
	conn.Send("SET", "xo", v)
	r, err := conn.Do("EXEC")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(r)
}
```

在获取`xo`的值之前先通过`WATCH`命令监控了该键，此后又将`set`命令包围在事务中，这样就可以有效的保证每个连接在执行EXEC之前，如果当前连接获取的`xo`的值被其它连接的客户端修改，那么当前连接的EXEC命令将执行失败。这样调用者在判断返回值后就可以获悉val是否被重新设置成功。

**注意事项**

- 由于WATCH命令的作用只是当被监控的键值被修改后阻止之后一个事务的执行，而不能保证其他客户端不修改这一键值，所以在一般的情况下我们需要在EXEC执行失败后重新执行整个函数。
- 执行EXEC命令后会取消对所有键的监控，如果不想执行事务中的命令也可以使用`UNWATCH`命令来取消监控。

###  连接池使用

redis连接池是通过pool结构体实现，以下是源码定义，相关参数说明已经备注：

```go
type Pool struct {
    // Dial is an application supplied function for creating and configuring a
    // connection.
    //
    // The connection returned from Dial must not be in a special state
    // (subscribed to pubsub channel, transaction started, ...).
    Dial func() (Conn, error) //连接方法

    // TestOnBorrow is an optional application supplied function for checking
    // the health of an idle connection before the connection is used again by
    // the application. Argument t is the time that the connection was returned
    // to the pool. If the function returns an error, then the connection is
    // closed.
    TestOnBorrow func(c Conn, t time.Time) error

    // Maximum number of idle connections in the pool.
    MaxIdle int  //最大的空闲连接数，即使没有redis连接时依然可以保持N个空闲的连接，而不被清除，随时处于待命状态

    // Maximum number of connections allocated by the pool at a given time.
    // When zero, there is no limit on the number of connections in the pool.
    MaxActive int //最大的激活连接数，同时最多有N个连接

    // Close connections after remaining idle for this duration. If the value
    // is zero, then idle connections are not closed. Applications should set
    // the timeout to a value less than the server's timeout.
    IdleTimeout time.Duration  //空闲连接等待时间，超过此时间后，空闲连接将被关闭

    // If Wait is true and the pool is at the MaxActive limit, then Get() waits
    // for a connection to be returned to the pool before returning.
    Wait bool  //当配置项为true并且MaxActive参数有限制时候，使用Get方法等待一个连接返回给连接池

    // Close connections older than this duration. If the value is zero, then
    // the pool does not close connections based on age.
    MaxConnLifetime time.Duration
    // contains filtered or unexported fields
}
```

新建go文件(demo18_redis_pool.go)，示例代码：

```go
package main

import (
	"fmt"
	"github.com/gomodule/redigo/redis"
	"time"
)

var Pool redis.Pool

func init() { // init 用于初始化一些参数，先于main执行
	Pool = redis.Pool{
		MaxIdle:     16,
		MaxActive:   32,
		IdleTimeout: 120,
		Dial: func() (redis.Conn, error) {
			c, err := redis.Dial("tcp", "127.0.0.1:6379",
				redis.DialConnectTimeout(30 * time.Millisecond),
				redis.DialReadTimeout(5 * time.Millisecond),
				redis.DialWriteTimeout(5 * time.Millisecond))
			if err != nil {
				fmt.Println(err)
				return nil, err
			}
			return c, nil
		},
	}
}

func someFunc() {
	conn := Pool.Get()
	defer conn.Close()
	res, err := conn.Do("HSET", "user", "name", "lianshi")
	fmt.Println(res, err)
	res1, err := redis.String(conn.Do("HGET", "user", "name"))
	fmt.Printf("res:%s,error:%v\n", res1, err)
	//...
}
func main() {

	someFunc()

}
//0 <nil>
//res:lianshi,error:<nil>
```

#### 密码认证和选择指定数据库

```go
pool := &redis.Pool{
  // 其他配置同上，这里就省略了...
  Dial: func () (redis.Conn, error) {
    c, err := redis.Dial("tcp", server)
    if err != nil {
      return nil, err
    }
    // auth认证
    if _, err := c.Do("AUTH", password); err != nil {
      c.Close()
      return nil, err
    }
    // 使用select指令选择数据库
    if _, err := c.Do("SELECT", db); err != nil {
      c.Close()
      return nil, err
    }
    return c, nil
  },
}
```

#### 连接检测

在将连接返回给应用程序之前，使用`TestOnBorrow`函数检查空闲连接的健康状况。

下面的示例是对空闲超过一分钟的连接执行`ping`命令检测连接是否可用:

```go
pool := &redis.Pool{
  // 其他配置同上，这里就省略了...
  // 
  TestOnBorrow: func(c redis.Conn, t time.Time) error {
    if time.Since(t) < time.Minute {
      return nil
    }
    _, err := c.Do("PING")
    return err
  },
}
```

## redis集群

`codis`   `redis-cluster`



## 总结（5分钟）

### redis场景

也是一个应用类工具，一定想办法把它用起来。。。

ZSET:榜单、排行榜、定时任务

INCR点赞

LIST 管理后台发通知邮件/短信 

去重

缓存 --> 把热点数据缓存起来，缩短响应时间



想获取更多的redis应用场景，就要多看书《Redis开发与运维》 《redis实战》，多看网上大厂的技术分享



### 面试会问的：

redis 常用的数据结构底层实现是什么？  ziplist、跳跃表

缓存雪崩 、缓存击穿。。。



### redis优化

key的优化

key尽量都要设置过期时间

一级key数量不要太多

bigkey尽量少

key分级

- shop_userinfo_1000 

- shop_userinfo_2000

集群

持久化

AOF 和 RDB



### GORM 

> 说明：
>
> 回顾本堂课所有知识点；
>
> Go如何设计database/sql接口，然后介绍了第三方关系型数据库驱动的使用，比如MySQL。接着介绍了NOSQL的一些知识，我们使用的是Redis，大家也可以自己学习一下其他的的NOSQL数据库，目前Go对于NOSQL支持还是不错，因为Go作为21世纪的C语言，那么对于21世纪的数据库也是支持的相当好。

