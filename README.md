# linux-docker-he

## Introduce

仅以此仓库记录个人学习docker过程 以及compose.yaml文件的相关配置

容器：软件在不同的运行环境（例如开发环境、测试环境、生产环境）之间切换时，通常会遇到依赖不一致、系统配置差异等问题，这可能导致软件无法正常运行。容器通过将应用程序及其所有依赖（包括库、配置文件等）打包在一起，提供了一个一致的、隔离的运行环境。

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
sudo yum-config-manager \
 --add-repo \
 https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

$ sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo



官方源

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
            "https://docker.m.daocloud.io/",
             "https://docker.registry.cy",
            "https://docker.registry.cyou",
            "https://docker-cf.registry.cyou",
            "https://dockercf.jsdelivr.fyi",
            "https://docker.jsdelivr.fyi",
            "https://dockertest.jsdelivr.fyi",
            "https://mirror.aliyuncs.com",
            "https://dockerproxy.com",
            "https://mirror.baidubce.com",
            "https://docker.m.daocloud.io",
            "https://docker.nju.edu.cn",
            "https://docker.mirrors.sjtug.sjtu.edu.cn",
            "https://docker.mirrors.ustc.edu.cn",
            "https://mirror.iscas.ac.cn",
            "https://docker.rainbond.cc"
     ]
    }
    ```

12. 重启
    
    > systemctl daemon-reload 
    > 
    > systemctl restart docker

## Update

2024-10-15

运行`docker compose`命令时，Compose 会在当前工作目录中按以下优先顺序寻找配置文件：

1. **compose.yaml**（首选）
2. **compose.yml**

这两个是推荐的文件命名规则。如果这两个文件都存在，Docker Compose 会优先选择 **compose.yaml**，因为这是 Docker Compose 规范的最新标准。

此外，为了向后兼容较早版本的 Compose，Compose 还会支持以下文件名：

1. **docker-compose.yaml**
2. **docker-compose.yml**

如果你的项目中同时存在 `compose.yaml` 和 `docker-compose.yaml`，系统也会优先选择 `compose.yaml`，因为它是最新的规范。

这个机制确保了项目的配置文件可以兼容不同版本的 Docker Compose，同时遵循最新的命名规范。



2024-10-09

删除卷的时候出现volume is in use

这通常意味着该卷仍然被某个容器占用或与某个容器绑定。因此，在卷未被释放或解除绑定之前，Docker 不允许删除该卷。

> docker ps -a --filter volume=<volume_name>
> 
> docker stop <container_name_or_id>
> 
> docker rm <container_name_or_id> #直接强制删除就行
> 
> docker volume rm -f <volume_name>
> 
> docker volume prune #移除未使用的卷
> 
> docker rm -v #顺手删除卷

docker不能仅仅通过depends_on确保依赖的零一个容器完全启动

健康检查

> healthcheck: test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
>  interval: 10s # 每10秒执行一次健康检查 timeout: 5s # 每次检查的超时时间为5秒 retries: 5 # 连续5次健康检查失败后视为容器不可用 start_period: 30s

2024-10-07

桥接网络和NAT网络

桥接网络之下虚拟机直接连接到宿主机的物理网卡 虚拟机可以访问外界网络 有自己的IP

NAT网络之下的虚拟机不需要占用额外的IP 虚拟机可以访问外部网络 但是外部不可以直接访问虚拟机 更安全 虚拟机不能直接与局域网内其他设备通信，除非配置了端口转发 

<br/>

常用命令：

1. 清屏
   
   > clear

2. 快速删除
   
   > ctrl+U

3. 列出目录内容
   
   > ls

4. 显示当前目录文件和子目录
   
   > ls -l

5. 罗列出详细信息 权限大小 修改时间
   
   > ls -a

6. 进入目录
   
   > cd + 目录名

7. 进入指定目录
   
   > cd /etc/docker

8. 退出目录
   
   > cd ..

9. 返回上一级
   
   > cd -

10. 查看当前目录
    
    > pwd

11. 创建目录
    
    > mkdir + 目录名

12. 删除目录
    
    > rmdir

13. 删除空目录
    
    > rm -r 目录名

14. 重命名
    
    > mv 就目录 新目录

15. 查看操作系统版本
    
    > cat /etc/os-release

16. 查看内核版本
    
    > uname -r

17. 查看CPU
    
    > lscpu

18. 内存使用情况
    
    > free -h

19. 磁盘占用
    
    > df -h

20. 删除整行
    
    > dd

2024-10-06

如果你在你的本地计算机上运行了一台虚拟机，这台计算机就是**宿主机**。即使你通过 SSH 远程连接到这台虚拟机，虚拟机所需的资源（CPU、内存、磁盘等）仍然来自你的本地计算机（宿主机）。只是通过 SSH，你在远程设备上操作虚拟机，而不直接使用宿主机的图形界面。

1. 节约开销 减少图形界面开销

2. 仅仅处理命令行 清量

3. SSH 允许你从任何地方通过网络远程管理虚拟机，无需物理接触宿主机或虚拟机所在的设备。即便你不在宿主机旁边，也能通过网络完成一切管理任务。

4. 即使虚拟机和宿主机位于远端（如服务器上），你也能远程连接虚拟机进行操作，不需要在设备旁边。

<hr/>

2024-10-02.docker compose

**镜像是独立于Docker引擎的，即使卸载了Docker，镜像文件仍然会存在于本地的文件系统中**

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

3. 获取 GPG 密钥失败：[Errno 14] curl#35 - "TCP connection reset by peer"
   
   1. 原因分析:网络问题
   
   2. 解决方法： 鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。
      
      执行下面的命令添加 `yum` 软件源：(这个只会影响软件包的管理 并不会影响docker拉取镜像的速度)
      
      ```vim
       sudo yum-config-manager \
          --add-repo \
          https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
      
      $ sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo
      ```

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