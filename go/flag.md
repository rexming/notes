# flag 用来解析命令行参数
一般样式 -flagName=param
param 参数带与不带""完全等价
原因：os中解析进去都是不带""字符的参数
```
fmt.Println(os.Args[1:])
>>
E:\golangProject\demo\src>go run main.go -age=2.1
page: 2.1
[-age=2.1]

E:\golangProject\demo\src>go run main.go -age="2.1"
page: 2.1
[-age=2.1]

```