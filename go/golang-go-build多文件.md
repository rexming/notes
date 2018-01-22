# 问题
今天将main主程序中函数拆分到main包下其它文件，在main程序中进行调用,编译报错，提示函数未被定义，项目结构
--- src
	--------main.go
	--------demo.go
```
main.go
func main() {
  fmt.Println("in main")
  demo() 
}

demo
func demo()  {
  fmt.Println("in demo")
}
```
运行go build main.go 即报错误
正常编译为go build main.go demo.go 或者go build main.go和demo.go的上一级目录
和gcc 编译多文件类似

go install 如果是main源文件可以在gobin下生成一个可执行的exe文件，
			如果main文件中引入其他包，会在pk目录下生成编译好的.a文件
			方便以后如果在引用这个包，不需要从新编译

官方建议最好将GOBIN目录也加到PATH路径下，以方便gobin下exe文件可以随时运行
windows下添加方法：PATH = %PATH%;%GOPATH%\bin
linux下添加方法：$ **export PATH=\$PATH:\$(go env GOPATH)/bin**

