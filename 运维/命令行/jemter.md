### 参数介绍

-n 非 GUI 模式 -> 在非 GUI 模式下运行 JMeter
-t 测试文件 -> 要运行的 JMeter 测试脚本文件

-l 日志文件 -> 记录结果的文件
-r 远程执行 -> 在Jmter.properties文件中指定的所有远程服务器
-H 代理主机 -> 设置 JMeter 使用的代理主机  
-P 代理端口 -> 设置 JMeter 使用的代理主机的端口号

```shell
./jmeter -n -t examples/CSVSample.jmx -l result/res.jtl -j log/1.log
```

```text
Creating summariser <summary>

Created the tree successfully using examples/CSVSample.jmx

Starting standalone test @ December 5, 2022 10:08:13 PM CST (1670249293027)

Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445

summary =     12 in 00:00:03 =    4.2/s Avg:   217 Min:   109 Max:   352 Err:     0 (0.00%)

Tidying up ...    @ December 5, 2022 10:08:15 PM CST (1670249295906)

... end of run
```


```shell
jmeter.bat -n -t testscript/Baidu.jmx -R 192.168.182.129:1100,192.168.182.130:1200 -l testresult/01-result.jtl -j report\01-log.log
```

### 测试脚本



### 参考
https://jmeter.apache.org/usermanual/get-started.html#non_gui
https://www.jianshu.com/p/ab1c64cd1e98
