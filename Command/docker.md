<!--
 * @Author: your name
 * @Date: 2020-07-01 09:19:32
 * @LastEditTime: 2020-07-01 09:24:10
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \StupidBirdFliesFirst\Command\docker.md
--> 
# Docker
## Docker模板
```
FROM nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04

#更改apt源
COPY sources.list etc/apt
RUN apt-get update
#必装
RUN apt-get install python3 -y && apt install python3-pip -y
RUN apt-get install openssl -y && apt-get install libssl-dev -y
RUN apt-get install git -y
RUN apt-get install wget -y
RUN apt-get install mlocate -y && updatedb
RUN apt-get install vim -y

RUN /bin/bash -c "echo alias python=python3 >> /root/.bashrc && echo alias pip=pip3 >> ~/.bashrc && source /root/.bashrc"

RUN apt-get install zsh -y
RUN git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh \
    && cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc \
    && chsh -s /bin/zsh
RUN /bin/bash -c "echo alias python=python3 >> /root/.zshrc && echo alias pip=pip3 >> ~/.zshrc && source /root/.zshrc"

## 个人操作

#安装ssh，可以进行远程调试
# RUN DEBIAN_FRONTEND=noninteractive apt-get install xorg -y - if you get select keyboard config - use this
RUN apt-get install xorg -y
RUN apt-get install openbox -y
RUN apt-get install -y openssh-server
#RUN mkdir /var/run/sshd
RUN echo 'root:wei' | chpasswd
RUN echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
RUN echo 'Port 22' >> /etc/ssh/sshd_config
RUN mkdir /run/sshd
#RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
#RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
#ENV NOTVISIBLE "in users profile"
#RUN echo "export VISIBLE=now" >> /etc/profile
EXPOSE 22

WORKDIR /home
CMD ["/usr/sbin/sshd", "-D"]
```

## Docker命令
1. 创建镜像
```
docker built -t test/test:1 .
```
2. 创建容器
```
docker run --runtime=nvidia -it -P test/test:1
``` 
3. 查看镜像
```
docker images
```
4. 删除镜像
```
docker image rm test/test:1
```
5. 查看容器
```
docker ps
```