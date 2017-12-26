# 代码如下：
```
	queryTasks := doFetchQueryTask(order)
	var queryTasksCopy []*util.Query
	err = util.DeepCopy(&queryTasksCopy, queryTasks)
	if err != nil {
		log.Println(err)
		return
	}
	

	chOfficalResult := make(chan []*util.QueryResult) //
	chTravelResult := make(chan []*util.QueryResult)  //
	
	timeout := time.After(time.Minute * 2) //超时时间2分钟
	go queryOfficial.QueryOfficial(queryTasks, chOfficalResult)
	go query.QueryTravel(queryTasksCopy, chTravelResult)
```

当如果不对queryTasks 进行深拷贝，直接传入两个go 协程中，会造成无法预计错误，因为queryTask 为[]*util.Query 指针数组，属于引用类型，可以直接操作引用值，故当一个协程中对其进行改变操作，另一个会受影响。故用gob进行深拷贝后传入，或者直接传值类型，当传引用时，多线程还会引起竞争