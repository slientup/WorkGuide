# API网关常用限流算法
#### 参考链接
- [如何限流？在工作中是怎么做的？说一下具体的实现？](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/huifer-how-to-limit-current.md)
- [死磕nginx系列--nginx 限流配置](https://www.cnblogs.com/biglittleant/p/8979915.html)

#### 令牌桶算法和漏桶算法
**令牌算法**：限制的是平均流入流量，拿到令牌才能进入系统进行处理；
**漏桶算法**：限制的是平均流出流量
**两者较大区别**：漏桶和令牌桶算法最明显的区别就是是否允许突发流量(burst)的处理，漏桶算法能够**强行限制数据**的实时传输（处理）速率，对突发流量不做额外处理；
而令牌桶算法能够在限制数据的平均传输速率的同时允许某种程度的突发传输(主要看桶里面是否积累了多余的令牌)
