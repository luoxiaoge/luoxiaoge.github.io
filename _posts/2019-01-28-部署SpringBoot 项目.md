###　测试部署SpringBoot 项目

>相对之前的SSM项目来说，Springboot就简单许多了，只需要配置JDK环境，也不需要Tomcat
>
>本文就记录一下运行脚本吧

#### run.sh

```shell
#!/bin/sh
nohup  java  -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=7999  - jar  /root/advertising/advertising-web-0.0.1-SNAPSHOT.jar &
echo $! > /var/run/ad.pid
tail -f nohup.out
```

#### stop.sh

```shell
#!/bin/sh
PID=$(cat /var/run/ad.pid)
kill -15 $PID
```

其他的以后详细在做补充，先记录一下