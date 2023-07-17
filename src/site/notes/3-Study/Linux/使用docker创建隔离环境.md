---
{"dg-publish":true,"permalink":"/3-Study/Linux/使用docker创建隔离环境/"}
---

# 目的
- [ ] 电脑A访问远程服务器B
- [ ] 在B上有docker创建的容器C
- [ ] 电脑A，服务器B能连外网，容器C不允许连外网
- [ ] 在电脑A上打开容器C中带UI的程序
# 方法
- [ ] 在服务器B上安装SSH服务端并启用X11转发
```
vim /etc/ssh/sshd_config
X11Forwarding yes
X11UseLocalhost no

sudo service ssh restart
```
- [ ] 在服务器B上使用docker创建虚拟隔离网络
```
docker network create --internal isolated_network
```
- [ ] 使用特定参数打开docker 容器
```
打开本地docker的图形界面
xhost +
docker run -d \

  -v /etc/localtime:/etc/localtime:ro \

  -v /tmp/.X11-unix:/tmp/.X11-unix \        #共享本地unix端口

  -e DISPLAY=unix$DISPLAY \                 #修改环境变量DISPLAY

  -e GDK_SCALE \                            #这两个可能是与显示效果相关的环境变量

  -e GDK_DPI_SCALE \

  --name libreoffice \

  jess/libreoffice
  
xhost -

打开远程docker的图形界面
xhost +
docker run -d \\

  -v /etc/localtime:/etc/localtime:ro \\

  --net=host or isolated_network \\              #必须

  -e DISPLAY=$DISPLAY \\                        #必须

  -v $HOME/slides:/root/slides \\

  -v $HOME/.Xauthority:/root/.Xauthority \\      #必须

  --name libreoffice \\

  jess/libreoffice
  
xhost -
```

# 示例
- [ ] Dockerfile示例
```
FROM ubuntu:latest
#安装必要的软件包   
RUN apt-get update && apt-get install -y \ 
x11-apps \ 
&& rm -rf /var/lib/apt/lists/*   
#设置环境变量以使用主机系统的X11服务器   
ENV DISPLAY=:0   
CMD \["xeyes"\]
```


