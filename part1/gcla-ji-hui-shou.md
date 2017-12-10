# GC模式
三色标记法
判断一个对象是不是垃圾需不需要标记，就看是否能从当前栈或全局数据区 直接或间接的引用到这个对象。这个初始的当前goroutine的栈和全局数据区称为GC的root区。扫描从这里开始，通过markroot将所有root区域的指针标记为可达，然后沿着这些指针扫描，递归地标记遇到的所有可达对象。
set GODEBUG=gctrace=1/2
然后运行会显示出gc信息

如何清空map（slice同理）:
1.自动gc:
a := map[string]string{"hello": "world"}
a = make(map[string]string)

the original map will be garbage-collected eventually; you don't need to clear it manually. But remember that maps (and slices) are reference types; you create them with make(). The underlying map will be garbage-collected only when there are no references to it. Thus, when you do:

a := map[string]string{"hello": "world"}
b := a
a = make(map[string]string)

the original array will not be garbage collected (until b is garbage-collected or b refers to something else).
2.循环delete删除

3：不规范示例：
map =  nil
The value of an uninitialized map is nil.
这样当使用时需要重新make()
同理适用于数组，但是append
