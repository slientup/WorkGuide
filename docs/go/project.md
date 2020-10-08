- Going Infinite, handling 1M websockets connections in Go 看设计思想

### 项目
1. [简单的web项目](https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/19.1.md)

  简介：包含了go的各种特性，详细的设计思路
  - 版本 1： 利用映射和结构体，与 sync 包的 Mutex 一起使用，以及一个结构体工厂。
  - 版本 2： 数据以 gob 格式写入文件以实现持久化。
  - 版本 3： 利用协程和通道重写应用（见 14章）。
  - 版本 4： 如果我们要使用 json 格式的文件该如何修改？
  - 版本 5： 用 rpc 协议实现的分布式版本。
  分布式架构：![](https://raw.githubusercontent.com/unknwon/the-way-to-go_ZH_CN/master/eBook/images/19.8_fig19.5.jpg)
  
2. [build-web-application-with-golang](https://github.com/astaxie/build-web-application-with-golang)
  这是一个用go语言写的web项目，涉及一个项目各个方面的处理，手册

3. [Going Infinite, handling 1M websockets connections in Go](https://speakerdeck.com/eranyanay/going-infinite-handling-1m-websockets-connections-in-go)
  this is design doc that handling 1M websockets connections in Go. well worth reading,include low-level knowledge,such as epoll 
  
