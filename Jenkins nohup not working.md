
使用 Jenkins 调用脚本在本机运行编译好的程序，类似于下面的命令，直接编译执行 java 程序。

```
	nohup mvn clean compile exec:java > $(date +%F).log 2>&1 &
```

此时 java 程序会先运行起来，日志也显示是运行成功的，但是会在 Jenkins 构建完成后自动 kill 此程序，导致程序又自动停止。

具体原因请参考：[ProcessTreeKiller](https://wiki.jenkins.io/display/JENKINS/ProcessTreeKiller)

根据官网指示，在调用命令时先定义 `BUILD_ID=dontKillMe`，即可解决：

```
	BUILD_ID=dontKillMe nohup mvn clean compile exec:java > $(date +%F).log 2>&1 &
```


如果是 Jenkisn 使用 ssh 调用其他服务器上执行脚本，则没有此问题：

```
	/usr/bin/ssh  -i ~/.ssh/jenkins_rsa username@192.168.1.10 '/app/home/app/deploy.sh'
```


<p align="right">——运维支持部 </p>
<p align="right">2018.03.05 </p>
（完）
