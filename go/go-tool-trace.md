原文：https://making.pusher.com/go-tool-trace/
### go tool trace 
可以将go程序在运行时的所有事件可视化，是Go自带工具链中最有用的工具之一，用于诊断性能问题，如延迟，并行化和竞争。
# Tour of the go tool trace interface
go tool trace 会显示很多信息，所以很难知道从哪里开始
图示：
来源https://making.pusher.com/images/2017-03-22-go-tool-trace/tour.svg

下面是一个并行化的快排图示：
