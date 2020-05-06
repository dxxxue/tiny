## sync包学习
### 1. 对应包的主要内容
- Locker
- Cond
- Map         // 并发安全的map类型，由atomic包进行实现，对应的方法，Store(添加), Load(读取), Delete(删除), Range(遍历)
- Mutex       // 互斥锁，解决goroutine的并发问题，Lock方法用来获取令牌(token,也叫上锁)， Unlock方法是释放令牌
- Once        // 延时初始化，调用多次，只会执行一次，也就是Once.Do方法只会执行一次，可以用来实现单例
- Pool
- RWMutex     // 读写互斥锁，有些场景只存在读操作，或者写操作   
- WaitGroup   // 用来等待goroutine的结束, 对应的wait(等待), Add(加), Done(减)  



### 2. 并发安全问题的排除(竞态检测器)
- 通过 go build main.go -race其中的race可以用来分析竞争关系