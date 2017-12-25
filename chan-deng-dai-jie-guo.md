# 需求是两个协程去执行，等待两个结果，存在超时限制
```
func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go func(chan int) {
		//do something
		time.Sleep(time.Second * 8)
		ch1 <- 1
	}(ch1)

	go func(chan int) {
		//do something
	}(ch2)

	//方案1
	timeOut := time.After(time.Second * 10)
	select {
	case <-ch1:
		timeOut2 := time.After(time.Second * 3)
		select {
		case <-ch2:
			fmt.Println("两个结果都等待到了")
		case <-timeOut2:
			fmt.Println("ch2 结果超时")
		}
	case <-ch2:
		timeOut1 := time.After(time.Second * 4)
		select {
		case <-ch1:
			fmt.Println("两个结果都等待到了")
		case <-timeOut1:
			fmt.Println("ch1 结果超时")
		}

	case <-timeOut:
		fmt.Println("两个结果都超时")

	}

	//方案2
	var chArr = make([]int, 0)
Lable:

	for {
		select {
		case t1 := <-ch1:
			chArr = append(chArr, t1)
		case t2 := <-ch2:
			chArr = append(chArr, t2)
		case <-timeOut:
			fmt.Println("timeout")
			break Lable //break 如果在for中select中，break无法跳出for,故需要借助Lable标签跳出

		}
		if len(chArr) == 2 {
			break Lable
		}
	}

}
```
方案1中的错误是，如果进入select选择结构，那样最外面的timeout超时设置就失去了作用，程序还是应多写测试
