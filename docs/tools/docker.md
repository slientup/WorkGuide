参考链接：
- Docker 基本概念解读 https://snailclimb.gitee.io/javaguide/#/docs/tools/Docker
- 一文搞懂 Docker 镜像的常用操作！ https://snailclimb.gitee.io/javaguide/#/docs/tools/Docker-Image


#### 工作项目中工作中使用：(如果hub中已有的镜像就不用这么dockerfile了)  
`第一步`： 创建一个dockerfile容器文件(自己项目的一些基础步骤)    
`第二步`：通过该文件生成对应的镜像文件 docker image build -t  
`第三步`：配置对应的docker-compose.yaml文件   
`第四步`：启动容器 docker-compose up -d  
#### 容器迁移
1. 将该镜像文件上传到hub上  
2. 其他机器从hub中下载文件  
3. 启动容器 docker-compose up -d  

#### 相关案例  
1. dockerfile文件的大体命令：  
```
  FROM docker.io/ubuntu:latest
  LABEL version="1.0" maintainer="Allen <zxzeng@foxmail.com>"
  # essential python  packages list
  COPY ./requirements/  /opt/requirements/
  RUN pip install -r /opt/requirements/requirements.txt
  #  set env for db, mq, supervisor
  ENV  NRDB_SERVICE_HOST  8.8.8.8
  ENV  NRDB_SERVICE_USER  zxzeng
  CMD ["/usr/bin/supervisord"]
```
2. docker image build -t    //生产镜像文件

#### 测试环境模拟 基于已有镜像创建容器

    [root@localhost docker]# `cat Dockerfile`  》》》 创建dockerfile文件
    FROM nginx         # 基于nginx基础
    RUN mkdir -p /opt/django/    # 在这个基础上在镜像文件中添加目录
    [root@localhost docker]# `docker build -t nginx:test`   》》创建镜像文件
    [root@localhost docker]# `docker images`   》》查看镜像文件的ID号 
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    nginx_test          0.1                 97666d6131e2        8 minutes ago       127 MB
    [root@localhost docker]# `docker run -it 97666d6131e2   /bin/bash`   》》 运行容器 并进入
    root@f4384e31d51a:/# cat index.html   》》》 创建index.html文件
    test
    testdadfas
    root@f4384e31d51a:/# exit   》》》 退出容器  记住`f4384e31d51a` ID 
    [root@localhost docker]docker container commit -m "Add index.html" -a "zxzeng" `f4384e31d51a` nginx_test:0.1  》》》根据现有容器的内容
    创建新的`image镜像文件`  这里ID与前面容器的ID是一致的
    [root@localhost docker]# docker images   》》》查看最新的镜像文件 
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    nginx_test          0.1                 97666d6131e2        17 minutes ago      127 MB







