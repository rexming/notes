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
```