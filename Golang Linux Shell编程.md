# Golang Linux Shell编程

## 1.调用系统命令 

exec包执行外部命令，它将os.StartProcess进行包装使得它更容易映射到stdin和stdout，并且利用pipe连接i/o

```go
func Command(name string, arg ...string) *Cmd {}
```

调用系统命令：

```go
package main

import (
   "os/exec"
   "log"
   "fmt"
)

func main() {
   cmd := exec.Command("ls","-l")
   out,err := cmd.CombinedOutput()//标准输出 标准错误 组合
   //out, err := cmd.Output()
   //Output()和CombinedOutput()不能够同时使用，
   // 因为command的标准输出只能有一个，同时使用的话便会定义了两个，便会报错
   if err != nil {
      log.Fatal(err)
   }
   fmt.Println(string(out))
}
```

## 2.交互式调用系统命令

```go
package main

import (
   "os/exec"
   "bufio"
   "fmt"
   "log"
   "time"
)

func main() {
   cmd := exec.Command("ls","-l")
   out,_ :=cmd.StdoutPipe() //StdoutPipe返回一个连接到command标准输出的管道pipe
   if err := cmd.Start();err != nil {
      log.Fatal("start error:%v",err)
   }

   f := bufio.NewReader(out)
   for {
      line,err := f.ReadString('\n')
      if err != nil {
         break
      }
      fmt.Print(line)
   }
   time.Sleep(time.Hour)
   //cmd.Wait()
   //Wait等待command退出，他必须和Start一起使用，
   //如果命令能够顺利执行完并顺利退出则返回nil，否则的话便会返回error，其中Wait会是放掉所有与cmd命令相关的资源
}
```

不加wait()会产生僵尸进程，3466 defunct 僵尸进程，wait收尸

```shell
go build cmd.go
./cmd
[root@greg02 ]#ps -ef |grep cmd2
root       3466   2539  0 20:37 pts/0    00:00:00 ./cmd2
[root@greg02 ]#ps -ef |grep ls
root        683      1  0 18:43 ?        00:00:21 /usr/bin/vmtoolsd
root       3470   3466  0 20:37 pts/0    00:00:00 [ls] <defunct>
```

## 3.自制bash

```go
package main

import (
   "bufio"
   "fmt"
   "os"
   "os/exec"
   "strings"
)

func main() {
   host, _ := os.Hostname()
   prompt := fmt.Sprintf("[ningxin@%s]$ ", host)
   r := bufio.NewScanner(os.Stdin)
   //r := bufio.NewReader(os.Stdin)
   for {
      fmt.Print(prompt)
      if !r.Scan() {
         break
      }
      line := r.Text()
      // line, _ := r.ReadString('\n')
      // line = strings.TrimSpace(line)
      if len(line) == 0 {
         continue
      }
      args := strings.Fields(line)
      cmd := exec.Command(args[0], args[1:]...)
      cmd.Stdin = os.Stdin
      cmd.Stdout = os.Stdout
      cmd.Stderr = os.Stderr
      err := cmd.Run()
      if err != nil {
         fmt.Println(err)
      }
   }
}
```

## 4.管道pipe

```go
package main

import (
   "fmt"
   "io"
   "log"
   "os"
   "os/exec"
   "strings"
)

func main() {

   line := "ls | grep f"
   cmds := strings.Split(line, "|")
   s1 := strings.Fields(cmds[0])
   s2 := strings.Fields(cmds[1])
   fmt.Println(s1)
   fmt.Println(s2)
   r, w := io.Pipe()
   cmd1 := exec.Command(s1[0], s1[1:]...)
   cmd2 := exec.Command(s2[0], s2[1:]...)
   //cmd1 := exec.Command("ls")
   //cmd2 := exec.Command("grep","f")
   cmd1.Stdin = os.Stdin
   cmd1.Stdout = w
   cmd2.Stdin = r
   cmd2.Stdout = os.Stdout
   cmd1.Start()
   cmd2.Start()
   log.Print("start")

   cmd1.Wait()
   cmd2.Wait()
}
```

## 5.shell.go

