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

```
	a := make(chan int)
	b := make(chan int)
	go func() {
		a <- 1
	}()
	go func() {
		b = nil
		//close(b)
	}()
	for {
		select {
		case <-a:
			fmt.Println("a")
			break
		case <-b:
			fmt.Println("b")
		default:
			fmt.Println("break")
			break
		}
	}
	// break 不会跳出此for 循环,因为它在select内，要跳出，需要加个标签在for循环外
	
```

# 利用 default 特性， 我们可以使用 select 语句来检测 chan 是否已经满了
```
	ch := make(chan int, 1)
	ch <- 1
	select {
	case ch <- 2:
	default:
		fmt.Println("channel is full !")
	}
	//因为 ch 插入 1 的时候已经满了， 当 ch 要插入 2 的时候，发现 ch 已经满了（case1 阻塞住）， 则 select 执行 default 语句。 这样就可以实现对 channel 是否已满的检测， 而不是一直等待。
	
	//比如我们有一个服务， 当请求进来的时候我们会生成一个 job 扔进 channel， 由其他协程从 channel 中获取 job 去执行。 但是我们希望当 channel 满了的时候， 将该 job 抛弃并回复 【服务繁忙，请稍微再试。】 就可以用 select 实现该需求。

```


