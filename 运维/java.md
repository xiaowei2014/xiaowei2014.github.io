jmap 查看堆内存的各项配置：jmap -heap pid

jstat -gc pid 采样间隔毫秒数

`jstat -gccapacity vmid` ：显示各个代的容量及使用情况；

`jstat -gcutil vmid` ：显示垃圾收集信息；

jmap -dump:format=b,file=文件名 [pid]