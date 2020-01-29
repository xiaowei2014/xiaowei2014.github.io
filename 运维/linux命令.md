# linux命令

查看打开的句柄数 ：ll /proc/{pid}/fd | wc -l 

查看线程数：ll /proc/{pid}/task | wc -l

找出占用CPU最高的线程：top -Hp pid、printf "%x" threadid、jstack pid | grep '线程id'