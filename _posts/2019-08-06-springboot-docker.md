---
layout:     post
title:      springboot项目docker部署
subtitle:   docker打包部署springboot 
date:       2019-08-06
author:     initpassion
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - java
    - springboot
    - docker
    - registry server 
---

# 前言

springboot项目搭建, docker 构建后发送到远程服务器, 实现远程仓库docker push;


# springboot-k8s-docker服务总结

## Docker安装

- ```
  //安装依赖包
  sudo yum install -y yum-utils  device-mapper-persistent-data  lvm2
  ```

- ```
  //设置稳定版仓库
  sudo yum-config-manager   --add-repo  https://download.docker.com/linux/centos/docker-ce.repo
  ```

- ```
  //安装最新版本
  sudo yum install docker-ce
  ```

- ```
  //或者安装指定版本
  yum list docker-ce --showduplicates | sort -r
  ```

- ```
  //指定一个版本进行安装
  sudo yum install docker-ce-<VERSION STRING>
  ```

- ```
  sudo systemctl start docker
  ```

- ```
  sudo docker run hello-world
  ```

- ```
  //卸载
  sudo yum remove docker-ce
  ```

- ```
  //开机启动
   sudo service docker start
  ```

##registry server镜像仓库

  #### localhost为私有仓库服务器ip地址

- ```
  docker run -d -p 5000:5000 --restart always --name registry registry:2
  ```

- ```
   vim /etc/docker/daemon.json
   //{"insecure-registries" :[localhost:5000] }
  ```

- ```
  docker pull nginx
  ```

- ```
  docker tag nginx localhost:5000/pnginx
  ```

- ```
  docker push localhost:5000/pnginx
  ```

- ```
  docker pull hyper/docker-registry-web
  ```

- ```
  docker run -it -p 8080:8080 --name registry-web --link registry -e REGISTRY_URL=http://registry:5000/v2 -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web 
  ```
  
- 
```
# docker完全卸载
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine

```  

## Docker之开启远程访问

- ```
  vim /usr/lib/systemd/system/docker.service
  // ExecStart=/usr/bin/dockerd  -H tcp://0.0.0.0:2375  -H unix:///var/run/docker.sock
  ```

- ```
  #重新加载配置文件
  systemctl daemon-reload 
  ```

- ```
  #重启服务
  systemctl restart docker.service 
  ```

- ```
  #查看端口是否开启
  netstat -nptl
  ```

- ```
  #直接curl看是否生效
  curl http://127.0.0.1:2375/info
  ```

## maven配置

-  ```
   <plugins>
   	 <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
              </plugin>
              <plugin>
                  <groupId>com.spotify</groupId>
                  <artifactId>docker-maven-plugin</artifactId>
                  <configuration>
                      <!-- 镜像名称  -->
                      <imageName>${project.artifactId}</imageName>
                      <!-- Dockerfile 文件目录 -->
                 <dockerDirectory>${project.basedir}/src/main/resources</dockerDirectory>
                      <!-- docker远程服务地址 ，前提是docker服务器需开启远程访问-->
                      <dockerHost>http://39.107.14.174:2375</dockerHost>
                      <resources>
                          <resource>
                              <targetPath>/</targetPath>
                              <!-- 资源所在目录 -->
                              <directory>${project.build.directory}</directory>
                              <!-- 生成的.jar文件 -->
                              <include>${project.build.finalName}.jar</include>
                          </resource>
                          <resource>
                              <targetPath>/</targetPath>
                              <!-- 资源所在目录 -->
                              <directory>${project.build.outputDirectory}</directory>
                          </resource>
                      </resources>
                  </configuration>
              </plugin>
   </plugins>
   ```

- ```
  #Dockerfile
  
  # 基础镜像使用java
  FROM java:8
  # VOLUME 指定了临时文件目录为/tmp。
  # 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
  VOLUME /tmp
  # 将jar包添加到容器中并更名为app.jar
  ADD boot-0.0.1-SNAPSHOT.jar app.jar
  # 运行jar包
  RUN bash -c 'touch /app.jar'
  ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
  ```

- ```
  # build
  clean install docker:build
  ```

- ```
  docker run -itd -p 80:8080 boot 
  ```

  