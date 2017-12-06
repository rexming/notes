# 错误示例
```
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := doStuff()
	fmt.Println(err)
}

func pressButton() {
	fmt.Println("I'm Mr. Meeseeks, look at me!!")
	// other stuff then happens, but if Jerry asks to
	// remove 2 strokes from his golf game...
	panic("It's gettin' weird!")
}

func doStuff() error {
	var err error
	// If there is a panic we need to recover in a deferred func
	defer func() {
		if r := recover(); r != nil {
			err = errors.New("the meeseeks went crazy!")
		}
	}()

	pressButton()
	return err
}
打印结果：
>>I'm Mr. Meeseeks, look at me!!
>><nil>
```

# 正确示例
```
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := doStuff()
	fmt.Println(err)
}

func pressButton() {
	fmt.Println("I'm Mr. Meeseeks, look at me!!")
	// other stuff then happens, but if Jerry asks to
	// remove 2 strokes from his golf game...
	panic("It's gettin' weird!")
}

func doStuff() (err error) {
	// If there is a panic we need to recover in a deferred func
	defer func() {
		if r := recover(); r != nil {
			err = errors.New("the meeseeks went crazy!")
		}
	}()

	pressButton()
	return err
}

打印输出
>>I'm Mr. Meeseeks, look at me!!
>>the meeseeks went crazy!	
```

# 原因
panic 会立即中断当前函数流程，（return 不会被执行），所以它实际上永远不会返回这个特定的err变量，而在我们的延迟函数内部改变它却变得毫无意义。