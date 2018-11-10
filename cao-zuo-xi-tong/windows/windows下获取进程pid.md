# 获取进程pid

1.tasklist  
2.wmic  
命令网上链接  
[http://www.cnblogs.com/archoncap/p/5400769.html](http://www.cnblogs.com/archoncap/p/5400769.html)  
dos 下输入 wmic process get name,processid  
便可以得到所有程序名称和pid

# go获取进程和打开新进程

    //获取进程ID
    func getPid(processName string) (int, error) {
        //通过wmic process get name,processid | findstr server.exe获取进程ID
        buf := bytes.Buffer{};
        cmd := exec.Command("wmic", "process", "get", "name,processid");
        cmd.Stdout = &buf;
        cmd.Run();
        cmd2 := exec.Command("findstr", processName);
        cmd2.Stdin = &buf;
        data, _ := cmd2.CombinedOutput();
        if len(data) == 0 {
            return -1, errors.New("not find");
        }
        info := string(data);
        //这里通过正则把进程id提取出来
        reg := regexp.MustCompile(`[0-9]+`);
        pid := reg.FindString(info);
        return strconv.Atoi(pid);
    }

```
//启动进程
func startProcess(exePath string, args []string) error {
    attr := &os.ProcAttr{
        //files指定新进程继承的活动文件对象
        //前三个分别为，标准输入、标准输出、标准错误输出
        Files: []*os.File{os.Stdin, os.Stdout, os.Stderr},
        //新进程的环境变量
        Env: os.Environ(),
    }

    p, err := os.StartProcess(exePath, args, attr);
    if err != nil {
        return err;
    }
    fmt.Println(exePath, "进程启动");
    p.Wait();
    return nil;
}
```



