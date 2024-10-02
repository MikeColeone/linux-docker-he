# linux-docker-he

## Introduce

仅以此仓库记录个人学习docker过程 以及compose.yaml文件的相关配置

## Enviroments

- [centos-7.9.2009-isos-x86_64安装包下载_开源镜像站-阿里云](https://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/)

- [docker](https://docs.docker.com/engine/install/centos/)

## Install

### 旧版本要卸载

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

### 使用yum安装

安装相关依赖包

```
sudo yum install -y yum-utils
```

切换到国内源 添加yum软件源

```

```

sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

$ sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

# 官方源

# $ sudo yum-config-manager \

# --add-repo \

# https://download.docker.com/linux/centos/docker-ce.repo

```
可能会报错**Error: Cannot find a valid baseurl for repo: base**

1. 检查网络： ping www.baidu.com

2. 网络不通：可能是配置的时候没有设置网络连接 重新安装

3. 网络正常

4. ```vim
   vi /etc/yum.repos.d/CentOS-Base.repo
```

5. 注释掉mirrorlist

6. 打开baseurl 并修改路径
   
   1. baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
   
   2. baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
   
   3. baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
   
   4. baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/

7. 安装docker
   
   ```vim
   sudo yum install docker
   ```

8. 启动docker
   
   > sudo systemctl enable docker
   > 
   > sudo systemctl start docker

9. 测试docker
   
   > docker run -rm hello-world

10. 测试失败修改daemon.json文件 添加加速地址
    
    > vi /etc/docker/daemon.json

11. 添加加速镜像地址 部分可能已经不可用 阿里云镜像服务可以申请
    
    （我的镜像地址）
    
    ```vim
    {
        "registry-mirrors": [
            "https://ustc-edu-cn.mirror.aliyuncs.com/",
            "https://ccr.ccs.tencentyun.com/",
            "https://docker.m.daocloud.io/"
        ]
    }
    ```

## Update

2024-10-02.docker compose

1. 在运行compose.yaml文件时出现该报错

> (root) Additional property ***** is not allowed



解决方法: docker-compose2.x之后需要修改docker-compose.yaml文件第一行加上services: 字段

> unknown shorthand flag: 'e' in -ersion

意思是输入命令时有有误，少输入了v,但是

![](C:\Users\Mike\AppData\Roaming\marktext\images\2024-10-02-17-01-38-QQ_1727859693534.png)

难绷... 初步鉴定为bug

2. 显示容器已经up 但是通过端口映射 浏览器无法访问
   
   1. docker logs [containner]
   
   2. 出现 `Host '172.19.0.3' is blocked because of many connection errors` 错误的原因是 MySQL 服务器由于过多连接错误阻止了特定 IP 地址的访问。
   
   3. 



<hr/>

2024-10-01

- docker仓库：docker hub 或私有的仓库

- docker镜像仓库：存放docker镜像

<hr/>

2024-09-31

- docker相关命令

- ‘’通过--help‘’ 查看细节 参数
  
  1. 检索
     
     > docker search
  
  2. 下载(或者官方镜像下载)
     
     > docker pull
  
  3. 列表
     
     > docker images
  
  4. 删除镜像
     
     > docker rmi
  
  5. 查看运行容器 运行的容器不能直接删除 除非使用强制删除
     
     > docker ps
  
  6. 启动容器
     
     > docker start
  
  7. 停掉应用
     
     > docker stop
  
  8. 运行容器 可以加参数设置 启动方式 运行在自己环境之内 是隔离的 要向外部访问 需要做端口映射 映射到自己主机 及通过主机访问
     
     > docker run
     > 
     > -d 后台启动 防止阻塞控制台
     > 
     > -- name 命名 并且不能重名
     > 
     > -p 端口
  
  9. 进入容器 修改应用 容器有自己的linux系统
     
     > docker exec
     > 
     > -it 以交互模式
     > 
     > /bin/bash 交互模式控制台
  
  10. 提交镜像
      
      > docker commit 提交镜像
  
  11. 保存镜像
      
      > docker save 保存镜像为文件
      > 
      > -o 压缩文件名
  
  12. 加载镜像
      
      > docker load
      > 
      > -i 压缩包
  
  13. 分享社区
      
      > docker login
  
  14. 命名
      
      > docker
  
  15. ci
      
      > docker
  
  16. 改名
      
      > docker tag
  
  17. 授权查看docker compose版本
      
      > docker-compose version
      > 
      > chmod +x /usr/local/bin/docker-compose
  
  18. 挂载和卷映射
  
  19. 查看容器细节
      
      > docker containner inspect 容器名
  
  20. 容器之间相互访问
      
      > 容器ip+端口 curl [http://ip+port](http://ip+port)
  
  21. 创建自动逸网络 实现容器名就是域名ip 直接使用容器名加容器端口就可以访问
  
  22. 通过自定义网络运行容器

<hr/>

2024-09-24

mysql:8.0.22

- 启动docker 并查看docker状态
  
  > sudo systemctl start docker
  > 
  > sudo systemctl status docker

- 搜索 拉取mysql镜像
  
  > docker search mysql
  > 
  > docker pull mysql:8

- 创建启动容器
  
  > docker run -di --name=容器名称 \
  > 
  > -p 3306:3306 \
  > 
  > -e MYSQL_ROOT_PASSWORD=密码 \
  > 
  > -d mysql:8

- 检查容器状态
  
  > docker ps

- 连接到Mysql
  
  > docker exec -it mysql-container mysql -uroot -p密码