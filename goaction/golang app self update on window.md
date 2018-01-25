# golang windows下exe程序自动更新

倒是在github上找到两个自动更新的，可是。。。

# 更新模式
CS模式，有一个服务器，每一次更新的版本都会放在服务器上，client里面开一个协程用于循环检查是否有更新，如果有就将新的版本程序下载下来，打开运行代替原来的

# 服务器端下载代码
1.直接用io输出流
```
w.Header().Set(`Content-Type`,`application/octet-stream`) //代表二进制流
w.Header().Set(`Content-Disposition`,`attachment;filename="time.exe"`) //程序名叫time.exe
io.Copy(w, file)
```
2.使用静态文件
