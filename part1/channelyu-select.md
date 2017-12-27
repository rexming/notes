# channel = nil
```
	var ch chan int
	fmt.Println(ch)
	fmt.Println(ch == nil)
	>> <nil>
	>> true
```
还未用make初始化时，ch 为nil

* 当chan = nil时，select不会选中这个通道
* 当chan 被关闭时，select会一直选中这个通道

# channel <- nil
chan 可以传入nil,不要与上面弄错

# tip
* 主线程可以单独使用select进行阻塞
* 主线程可以单独使用<-(chan struct{})(nil) 进行阻塞
* 上面两点的共同点是前提保证内部有其它协程一直存在运行


