# mtdhb服务端docker

服务端包含redis,mysql,api项目,get项目,目的为一键化部署项目, 如果已经在本地部署了对应的项目(如已安装了mysql等)，可以在docker-compose.yml注释掉对应的项目,然后修改对应的配置文件

## docker部署

### 环境部署

 在linux或者windows上安装docker和docker-compose 我这里以centos7为例子 
  - 安装docker `sudo yum install docker`
  - 安装docker-compose  `sudo pip install docker-compose`
  - 启动docker `sudo systemctl start docker` 要是出现selinux的错误 关掉selinux就好

### 下载文件

整个文件大小在1.5g左右,放到docker环境中

### 执行文件

 进入service目录下执行`sh run.sh`,整个过程大概3-5分钟，完成后可以通过`docker ps`查看运行中的镜像，或者通过`docker logs <镜像id>`来查单个镜像的log，注意这里的get项目还没有启动的所以`curl 127.0.0.1:3333`是返回失败的
 
### 运行get项目

 get项目由于我使用的centos的镜像，我就直接开放了ssh服务，开放的端口为12266，root初始密码为`qq1203943228`，可以自行在docker-compose.yml修改端口。
 
 这个项目由于对nodejs研究不深,dockerfile没能做成直接运行的docker，被迫用centos的镜像做成了载体，需要进入载体手动开启get项目。用ssh服务进入get 的docker 镜像中后执行`sh /run1.sh`就开启了get项目（docker exec 进入会导致项目不能运行，只能用ssh进入），退出镜像`exit`,退出镜像后再执行`curl 127.0.0.1:3333`返回成功的话说明get项目运行成功

## docker更新

 服务端使用的是docker-compose.yml文件启动的多个docker镜像,2个项目的镜像文件都是已经打包好的，但是关于更新也是做了对应预留，需要更新的话请注释掉service目录下run.sh中docker load的部分

### api项目更新
 
 该docker项目是默认连接的本地docker化的mysql和redis，application.yml中需要添加
```
spring:   
    redis:
        host: redis
        database: 0
        port: 6379              
```

 数据库连接部分也需要修改

```
spring:
    datasource:
        url: jdbc:mysql://db:3306/api?useUnicode=true&characterEncoding=UTF-8&useSSL=false
        username: root
        password: 123456
    jpa:
        show-sql: true
```

 如果需要修改数据库密码，在api项目下的sql目录中的ddl.sql中可以对应修改。
 简单说就是springboot项目桥接了mysql与redis，对应的host只要改成container名称就可以连接到服务，如果是要连接其他的mysql或者redis，修改对应host部分即可。
 api项目更新的话需要修改docker-compose.yml文件，找到api项目，放开build注释，然后注释掉image，然后在api文件夹中将更新后jar包放入，再执行一次service目录下的run.sh即可

### get项目更新

 get项目放在了 get 镜像中的 /opt/get下，在镜像运行的情况下直接ssh进去然后替换文件即可。 

 docker-compose.yml文件中还有关于nginx，如果有需要放开即可。如果布置在生产中，需要开放对应端口号。

欢迎提出意见 [@fulln](https://github.com/fulln)
 
