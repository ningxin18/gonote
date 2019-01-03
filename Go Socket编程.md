## Go Socket编程

在Go语言中编写网络程序时，我们将看不到传统的编码形式。以前我们使用Socket编程时，会按照如下步骤展开。
(1) 建立Socket：使用socket()函数。
(2) 绑定Socket：使用bind()函数。
(3) 监听：使用listen()函数。或者连接：使用connect()函数。
(4) 接受连接：使用accept()函数。
(5) 接收：使用receive()函数。或者发送：使用send()函数。
Go语言标准库对此过程进行了抽象和封装。无论我们期望使用什么协议建立什么形式的连接，都只需要调用net.Dial()即可。 

```go
func Dial(net, addr string) (Conn, error)
```

net参数是网络协议的名字， addr参数是IP地址或域名，而端口号以“ :”的形式跟随在地址
或域名的后面，端口号可选。如果连接成功，返回连接对象，否则返回error。 

在成功建立连接后，我们就可以进行数据的发送和接收。发送数据时，使用conn的Write()
成员方法，接收数据时使用Read()方法。 

## 客户端

建立连接
发送和接收数据
关闭连接 

```go
package main

import (
   "fmt"
   "io"
   "log"
   "net"
   "os"
)

func main() {
   addr := "www.baidu.com:80"
   // connection
   conn, err := net.Dial("tcp", addr)
   if err != nil {
      log.Fatal(err)
   }
   fmt.Println(conn.RemoteAddr().String())
   fmt.Println(conn.LocalAddr().String())
   n, err := conn.Write([]byte("GET / HTTP/1.1\r\n\r\n"))
   if err != nil {
      log.Fatal(err)
   }
   fmt.Println("write size", n)
   /*串行指定读取客户端返回内容大小
    buf := make([]byte, 10)
    n, err = conn.Read(buf)

    if err != nil && err != io.EOF {
       log.Fatal(err)
    }
    fmt.Println(string(buf[:n]))
   */
  
  /*按行读取
  r := bufio.NewReader(conn)
	//将这个链接（connection）包装以下,将conn的内容都放入r中，但是没有进行读取
	for  {
		line,err := r.ReadString('\n') //将r的内容也就是conn的数据按照换行符进行读取。
		if err == io.EOF {
			conn.Close()
		}
		fmt.Print(line)
	}
  */
   io.Copy(os.Stdout, conn)//io.Copy不会主动调用close，io.Copy结束的条件是reader得到EOF
   conn.Close()
}
```

## 服务端

监听端口
接受新的连接
启动协程
发送接收数据
断开连接 

```go
package main

import (
   "log"
   "net"
)

var content = `HTTP/1.1 200 OK
Date: Sat, 29 Jul 2017 06:18:23 GMT
Content-Type: text/html
Connection: Keep-Alive
Server: BWS/1.1
X-UA-Compatible: IE=Edge,chrome=1
BDPAGETYPE: 3
Set-Cookie: BDSVRTM=0; path=/

<html>
<body>
<h1 style="color:red">hello golang</h1>
</body>
</html>
`

func handleConn(conn net.Conn) {
   conn.Write([]byte(content))
   conn.Close()
}

func main() {
   addr := ":8021" //"0.0.0.0"
   listener, err := net.Listen("tcp", addr)
   if err != nil {
      log.Fatal(err)
   }
   defer listener.Close()

   for {
      conn, err := listener.Accept()
      if err != nil {
         log.Fatal(err)
      }
      go handleConn(conn)
   }
}
```

