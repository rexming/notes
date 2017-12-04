# 测试
+ 测试命令（go test）忽略已“_”或“.”开头的测试文件
+ 正常编译操作（go build/install）会忽略测试文件

# T 类型控制测试结果和行为
方法|说明|相关
---|---|---
Fail|失败：继续执行当前测试函数
FailNow|失败：立即终止执行当前测试函数|Failed
SkipNoe|跳过：停止执行当前测试函数|Skip,Skipf,Skipped