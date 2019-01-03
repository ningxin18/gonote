## golang安全编程

信息数据化和传输网络化对数据和数据传输的安全提出了要求。我们需要对数据进行加密，并使用安全的数据传输体系。Go是为网络时代设计的语言，对网络的支持也已融入其设计中，因此网络数据安全及其相应的体系就成了必须探讨的话题。 

### 对称加密

采用单密钥的加密算法，整个系统由如下几部分构成：需要加密的明文、加密算法和密钥。在加密和解密中，使用的密钥只有一个。常见的单密钥加密算法有DES、 AES、RC4等。

```go
package main

import (
   "crypto/md5"
   "crypto/rc4"
   "flag"
   "io"
   "log"
   "os"
)

var (
   key = flag.String("k", "", "secret key")
)

func crypto(w io.Writer, r io.Reader, key string) {
   // 创建cipher
   md5sum := md5.Sum([]byte(key))
   cipher, err := rc4.NewCipher(md5sum[:])
   if err != nil {
      log.Fatal(err)
   }
   buf := make([]byte, 4096)
   for {
      // 从r里面读取数据到buf
      n, err := r.Read(buf)
      if err == io.EOF {
         break
      }
      // 加密buf
      cipher.XORKeyStream(buf[:n], buf[:n])
      // 把buf写入到w里面
      w.Write(buf[:n])
   }
}

func main() {
   flag.Parse()
   crypto(os.Stdout, os.Stdin, *key)
}
// echo hello | ./rc4
/*
[root@greg02 day10]#go run rc4.go -k 123 <ftp.txt >1111
[root@greg02 day10]#ls
1111  ftp_cli  ftp_sevr.go  ftp.txt  rc4  rc4.go
[root@greg02 day10]#cat 1111
o쌎T򇸙񓙢A\¦[root@greg02 day10]#go run rc4.go -k 123 <1111 >22
[root@greg02 day10]#ls
1111  22  ftp_cli  ftp_sevr.go  ftp.txt  rc4  rc4.go
[root@greg02 day10]#cat 22
this is a ftp test
1234

[root@greg02 day10]#tar czf - * | ./rc4 -k 123 > go.tar.gz
[root@greg02 day10]#ls
1111  22  ftp_cli  ftp_sevr.go  ftp.txt  go.tar.gz  rc4  rc4.go
[root@greg02 day10]#./rc4 -k 123 <go.tar.gz | tar tzf -
1111
22
ftp_cli/
ftp_cli/ftp_cli.go
ftp_cli/ftp.txt
ftp_sevr.go
ftp.txt
go.tar.gz
rc4
rc4.go
*/
```

### 非对称加密

采用双密钥的加密算法，整个系统由如下几个部分构成：需要加密的明文、加密算法、私钥和公钥。在该系统中，私钥和公钥都可以被用作加密或者解密，但是用私钥加密的明文，必须要用对应的公钥解密，用公钥加密的明文，必须用对应的私钥解密。常见的双密钥加密算法有RSA等。 

在对称加密中，私钥不能暴露，否则在算法公开的情况下，数据等同于明文，而在非对称加密中，公钥是公开的，私钥是保密的。这样任何人都可以把自己的信息通过公钥和算法加密，然后发送给公钥的发布方，只有公钥发布方才能解开密文。在对称加密和非对称加密中，它们有一个共同的特点，即数据可以加密，也可以解密。

### 哈希算法

实际上，我们还有一种加密需求，只需要加密，形成一个密文，而不需要解密，可以使用哈希算法等。
哈希算法是一种从任意数据中创建固定长度摘要信息的办法。对于不同的数据，要求产生的摘要信息也是唯一的。常见的哈希算法包括MD5、 SHA-1等。 

```go
package main

import(
   "fmt"
   "crypto/sha1"
   "crypto/md5"
)

func main(){
   TestString:="hello golang"
   Md5Inst:=md5.New()
   Md5Inst.Write([]byte(TestString))
   Result:=Md5Inst.Sum([]byte(""))
   fmt.Printf("%x\n",Result)

   Sha1Inst:=sha1.New()
   Sha1Inst.Write([]byte(TestString))
   Result=Sha1Inst.Sum([]byte(""))
   fmt.Printf("%x\n",Result)
}
```

对文件内容计算SHA1 

```go
package main

import (
   "io"
   "fmt"
   "os"
   "crypto/md5"
   "crypto/sha1"
)

func main() {
   TestFile := "123.txt"
   infile, inerr := os.Open(TestFile)
   if inerr == nil {
      md5h := md5.New()
      io.Copy(md5h, infile)
      fmt.Printf("%x %s\n",md5h.Sum([]byte("")), TestFile)

      sha1h := sha1.New()
      io.Copy(sha1h, infile)
      fmt.Printf("%x %s\n",sha1h.Sum([]byte("")), TestFile)
   } else {
      fmt.Println(inerr)
      os.Exit(1)
   }
}
```

