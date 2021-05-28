# Linux进程和性能相关



## ps

```bash
# show processes
ps aux | grep KEYWORDS
```



## kill, pkill

```bash
# kill process by id.
# Default using TERM, give the target process a chance to clean up before exit.
kill ID
# or
kill -15 ID
# or
kill -s TERM ID

# kill process using SIGKILL
kill -9 ID
# or
kill -s KILL ID

# kill process by name
pkill NAME

# kill process by keywords of name
pkill -f KEYWORDS
```

https://unix.stackexchange.com/questions/8916/when-should-i-not-kill-9-a-process



## ctrl z, bg, fg, jobs, disown, screen

```bash
# 前台进程挂起
ctrl+Z

# 挂起进程继续运行在后台
bg

# 挂起进程/后台进程移动到前台运行
fg

# 查看后台任务列表
jobs

# 后台任务脱离ssh session，退出ssh后继续运行
disown -h
```

实例

```bash
# 前台任务保持运行并退出ssh
ctrl+Z
bg
disown
```

```bash
screen
CMD
ctrl+A
ctrl+D

# logout
# login

# resume CMD screen
screen -r
```



https://www.cjavapy.com/article/587/

https://askubuntu.com/questions/8653/how-to-keep-processes-running-after-ending-ssh-session



## top

```bash
# 查看进程
top

# 排序
top -o %CPU
top -o %MEM
```





## iotop

```bash
sudo apt install -y iotop

# 查看IO活动
sudo iotop
```



## iftop

```bash
apt install -y iftop

# 查看网络活动
sudo iftop
```



## lsof

```bash
# 查看打开的文件/设备等
lsof

lsof /mnt/disk
```

[3. lsof 一切皆文件 — Linux Tools Quick Tutorial (linuxtools-rst.readthedocs.io)](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/lsof.html)

