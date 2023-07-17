---
{"dg-publish":true,"permalink":"/3-Study/Linux/docker中安装带有vnc server的centos7 image/"}
---

#  创建一个名为`Dockerfile`的文件，内容如下：
> # 使用官方的 CentOS 7 基础镜像
> FROM centos:7
> 
> # 安装必要的软件包
> RUN yum -y update && \
>     yum -y install epel-release && \
>     yum -y groupinstall "X Window System" && \
>     yum -y install tigervnc-server xterm xorg-x11-fonts-Type1 firefox && \
>     yum clean all
> 
> # 设置环境变量
> ENV USER=root
> ENV HOME=/root
> ENV DISPLAY=:1
> ENV GEOMETRY=1280x800
> 
> # 添加 VNC 服务器配置文件
> COPY xstartup $HOME/.vnc/xstartup
> RUN chmod 755 $HOME/.vnc/xstartup
> 
> # 添加启动脚本
> COPY start-vnc.sh /start-vnc.sh
> RUN chmod 755 /start-vnc.sh
> 
> # 暴露 VNC 服务器端口
> EXPOSE 5901
> 
> # 启动 VNC 服务器
> CMD ["/start-vnc.sh"]

#  创建一个名为`xstartup`的文件，内容如下：
> #!/bin/sh
> unset SESSION_MANAGER
> unset DBUS_SESSION_BUS_ADDRESS
> [ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
> [ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
> xsetroot -solid grey
> vncconfig -iconic &
> xterm -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
> twm &

#  创建一个名为`start-vnc.sh`的文件，内容如下：
> #!/bin/sh
> 
> # 创建密码文件
> mkdir -p $HOME/.vnc
> echo "your-password" | vncpasswd -f > $HOME/.vnc/passwd
> chmod 600 $HOME/.vnc/passwd
> 
> # 启动 VNC 服务器
> vncserver $DISPLAY -geometry $GEOMETRY -rfbauth $HOME/.vnc/passwd
> tail -f /root/.vnc/*$DISPLAY.log

#  将这三个文件（Dockerfile、xstartup 和 [start-vnc.sh](http://start-vnc.sh/)）放在同一个目录中，然后在该目录中运行以下命令以构建Docker镜像：
> docker build -t centos7-vnc .

# 构建完成后，您可以使用以下命令运行一个新的容器：
> docker run -d -p 5901:5901 --name centos7-vnc centos7-vnc