# linux命令

查看打开的句柄数 ：ll /proc/{pid}/fd | wc -l 

查看线程数：ll /proc/{pid}/task | wc -l

找出占用CPU最高的线程：top -Hp pid、printf "%x" threadid、jstack pid | grep '线程id'

查看系统当前使用的fd:cat /proc/sys/fs/file-nr  

每个进程允许最大fd:ulimit -n

二进制dump成16禁止 hexdump -C -v 二进制文件＞mytest.txt

查看端口被哪个进程占用：lsof -i:端口号

netstat -tunlp

   -t：tcp

   -u：udp

   -n：拒绝显示别名

   -l：仅列出有在listener的服务状态

  -p：程序名