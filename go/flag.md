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
一篇flag配置全局参数的文章：http://tonybai.com/2018/01/13/the-problems-i-encountered-when-writing-go-code-issue-1st/?f=tt&hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io