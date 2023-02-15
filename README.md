### MIT 6.824 LAB 1

**逻辑实现：**

首先启动一个master并且将master注册RPC，设置path建立Unix domain socket实现worker与master的RPC通信。此外master根据不同文件名创建不同的map task并将其push到通过channel实现的阻塞队列中等待worker唤醒。

启动worker，通过RPC通信请求在阻塞队列中的map task，根据状态机将其分配给mapper（或者reducer）处理。mapper通过mapf方法将文件中单词切割并计数（计数类型用string表示）。将获得的单词string通过ihash()映射到不同文件中，此时通知master，map task完成，更新master中map task状态。

master检查所有master是否全部完成，若完成则master切换到reduce状态，将map task更新为reduce task并push到阻塞队列。worker请求reduce task（worker时刻都在请求，若阻塞队列中无task，则处于waiting状态）分配给reducer做排序和计数的操作并写到本地，告诉master， reduce task完成更新状态。当所有的reduce task完成，更新master状态为Exit并退出。

##### 测试结果：

```shell
$ ./test-mr.sh        
*** Starting wc test.
--- wc test: PASS
*** Starting indexer test.
--- indexer test: PASS
*** Starting map parallelism test.
--- map parallelism test: PASS
*** Starting reduce parallelism test.
--- reduce parallelism test: PASS
*** Starting crash test.
--- crash test: PASS
*** PASSED ALL TESTS
```

