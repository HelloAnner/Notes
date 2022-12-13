```shell
# 使用CPU最多的3个进程 或 top （然后按下P，注意大写)
ps -aux | sort -k3nr | head -3

# 使用内存最多的10个进程 或 top （然后按下M，注意大写)
ps -aux | sort -k4nr | head -10
```

