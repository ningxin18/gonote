golang linux shell编程（二）面向接口编程

tar包解压

```go
package main

import (
   "archive/tar"
   "os"
   "io"
   "fmt"
   "log"
   "compress/gzip"
)

func main() {
   uncompress,err := gzip.NewReader(os.Stdin)  //传入的文件解压传给“uncompress”
   if err != nil {
      log.Fatal(err)  //意思是当程序解压失败时，就立即终止程序，“log.Fatal”一般用于程序初始化。
   }
   tr := tar.NewReader(uncompress)
   /*
   从 “*.tar”文件中读出数据是通过“tar.Reader”完成的，所以首先要创建“tar.Reader”。
   也就是说这个文件是可读类型的。"tar.NewReader"只接受一个io.reader类型。
   可以通过“tar.NewReader”方法来创建它，该方法要求提供一个“os.Reader”对象，以便从该对象中读出数据。
   */
   for  {
      hdr,err := tr.Next()
      //此时，我们就拥有了一个“tar.Reader”对象 tr，可以用“tr.Next()”来遍历包中的文件
      if err != nil {
         return
      }
      fmt.Printf("已解压：\033[31;1m%s\033[0m\n",hdr.Name)
      //io.Copy(ioutil.Discard,tr) //表示将读取到到内容丢弃，"ioutil.Discard"可以看作是Linux中的：/dev/null
      info := hdr.FileInfo()  // 获取文件信息
      if info.IsDir() { //判断文件是否为目录
         os.Mkdir(hdr.Name,0755) //创建目录并赋予权限。
         continue //创建目录后就要跳过当前循环，继续下一次循环了。
      }
      f,_ := os.Create(hdr.Name)  //如果不是目录就直接创建该文件
      io.Copy(f,tr) //最终将读到的内容写入已经创建的文件中去。
      f.Close()
      /*
      不建议写成“defer f.Close()”因为“f.Close()”会将缓存中的数据写入到文件中，同时“f.Close()”
      还会向“*.tar”文件的最后写入结束信息，如果不关闭“f”而直接退出程序，那么将导致“.tar”文件不完整。而
        "defer f.Close()”是在函数结束后再执行关闭文件，那么在这个过程中，内存始终会被占用着，浪费这不必要的资源。
      */
   }
}
```

gzip解压

```go
package main

import (
   "compress/gzip"
   "os"
   "log"
   "archive/tar"
   "fmt"
   "io"
   "io/ioutil"
)

func main()  {
   //面向接口编程
   //只关注数据流，可读的数据流
   uncompress,err := gzip.NewReader(os.Stdin)
   if err != nil {
      log.Fatal(err)
   }
   tr := tar.NewReader(uncompress)
   for {
      hdr,err := tr.Next()
      if err != nil {
         return
      }
      fmt.Println(hdr.Name)
      io.Copy(ioutil.Discard,tr)
   }
}
```

gzip -c http.go > http.go.gz
greg@greg:day6$ file http.go.gz 
http.go.gz: gzip compressed data, was "http.go", last modified: Wed Jan 10 14:55:15 2018, from Unix
zcat http.go.gz 

zcat读取gzip压缩的文件

下面的代码相当于zcat，./uncompress < http.go.gz

```go
package main

import (
   "os"
   "compress/gzip"
   "log"
   "io"
)

func main() {
   //过滤器对象，中间件
   // 数据不落地，他也不是发源地
   // ls |grep|sort|uniq|tee a.txt
   r,err := gzip.NewReader(os.Stdin)
   if err != nil {
      log.Fatal(err)
   }
   io.Copy(os.Stdout,r)
}
```

生成tar文件

```go
package main

import (
   "archive/tar"
   "compress/gzip"
   "fmt"
   "io"
   "os"
   "path/filepath"
)

func tarFun(desc, src string) error {
   fd, err := os.Create(desc)
   if err != nil {
      return err
   }
   defer fd.Close()

   gw := gzip.NewWriter(fd)
   defer gw.Close()

   tr := tar.NewWriter(gw)
   defer tr.Close()

   err = filepath.Walk(src, func(path string, info os.FileInfo, err error) error {
      //遍历指定目录下所有文件
      if err != nil {
         return err
      }

      hdr, err := tar.FileInfoHeader(info, "")
      /*“tar.FileInfoHeader”其实是调用“os.FileInfo ”方法获取文件的信息的，你要知道文件有两个属性，
      一个是文件信息，比如大小啊，编码格式，修改时间等等，还有一个就是文件内容，就是我们所看到的具体内容。 */
      if err != nil {
         return err
      }

      hdr.Name = path
      err = tr.WriteHeader(hdr)
      //由于是目录，里面的内容我们就不用管理，只记录目录的文件信息。
      if err != nil {
         return err
      }

      fs, err := os.Open(path)
      if err != nil {
         return err
      }
      defer fs.Close()

      if info.Mode().IsRegular() {
         io.Copy(tr, fs)
      }
      return nil
   })

   if err != nil {
      return err
   }
   return nil
}

func main() {
   if len(os.Args) != 3 {
      fmt.Println("Usage: mytar [desc] [src]")
      return
   }
   tarFun(os.Args[1], os.Args[2])
}
```



