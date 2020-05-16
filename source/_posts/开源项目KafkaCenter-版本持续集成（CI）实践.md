---
title: 开源项目KafkaCenter 版本持续集成（CI）实践
tags:
  - 经验
originContent: ''
categories:
  - java
  - docker
toc: false
date: 2020-05-16 23:11:35
---

#  开源项目KafkaCenter 版本持续集成（CI）实践

## 开篇

本文简单介绍开源项目KafkaCenter 版本持续集成（CI）实践方案，主要解决了三个问题：
1. 前后端项目编译 
2. 发布Github release包 
3. 制作docker镜像
 
希望能给你带来一点参考。

详细信息可以参考 [https://github.com/xaecbd/KafkaCenter](https://github.com/xaecbd/KafkaCenter)

## 正文

## 版本管理

KafkaCenter 后端服务是java,使用maven管理的，有多个module，为了做到版本一致，我们使用了`${revision}`。这个是**maven3.5+** 才支持，主要是为了对CI友好。

例如：

父pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project >
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.nesc.ec.bigdata</groupId>
	<artifactId>KafkaCenter</artifactId>
	<version>${revision}</version>
	<packaging>pom</packaging>
	<name>KafkaCenter</name>
	<url>https://github.com/xaecbd/KafkaCenter</url>
	<description>Kafka Center Platform</description>
    ...
  <properties>
    <revision>1.0.0-SNAPSHOT</revision>
  </properties>

	<modules>
		<module>KafkaCenter-Base</module>
		<module>KafkaCenter-Core</module>
	</modules>
</project>
```

子module

```
<?xml version="1.0"?>
<project>
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.nesc.ec.bigdata</groupId>
		<artifactId>KafkaCenter</artifactId>
		<version>${revision}</version>
	</parent>
	<artifactId>KafkaCenter-Base</artifactId>
	<name>KafkaCenter-Base</name>
	<url>https://github.com/xaecbd/KafkaCenter</url>

</project>

```



通过如下命令，可以在打包的时候指定版本号：

```
mvn -Drevision=2.1.0 -Dchangelist= clean package
```

## Docker镜像

在项目根目录下新建docker文件夹，包含三个文件：

`docker-build-release.sh` build镜像及发布镜像的脚本

```
#!/usr/bin/env bash

DOCKER_IMAGE_NAME="xaecbd/kafka-center"
VERSION=${TRAVIS_TAG#v}
echo "KafkaCenter version: $VERSION"
echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
cp $TRAVIS_BUILD_DIR/KafkaCenter-Core/target/*.jar $TRAVIS_BUILD_DIR/docker/
docker build  -t $DOCKER_IMAGE_NAME:$VERSION $TRAVIS_BUILD_DIR/docker/
docker images
docker push $DOCKER_IMAGE_NAME:$VERSION
```



`Dockerfile` docker 镜像定义文件

```
FROM adoptopenjdk/openjdk8:jre8u252-b09-alpine
LABEL author="Turman.P.Du"
ENV PROJECT_BASE_DIR /opt/app/kafka-center/
WORKDIR ${PROJECT_BASE_DIR}

COPY *.jar ${PROJECT_BASE_DIR}/
COPY *.sh ${PROJECT_BASE_DIR}/

RUN chmod +x *.sh
ENTRYPOINT ["sh","start.sh"]
```



`start.sh` 应用启动脚本，非必须，只是我们习惯放这么个脚本，可以在应用启动前做一些工作。推荐在启动java应用前增加exec命令，这样可以让spring容器在docker容器停止运行前执行一些操作（可以用作应用停止前执行收尾工作，例如保存数据，停止不可中断的任务）。

```
#!/bin/sh
echo "PROJECT_BASE_DIR :"$PROJECT_BASE_DIR
#cd $APP_ROOT_DIR
cd $PROJECT_BASE_DIR

appName=`ls|grep .jar$`
echo start to run $appName

if [ -n "$JAVA_OPTIONS" ];then
	exec java $JAVA_OPTIONS -jar $appName   $@
else
    exec java -jar $appName   $@
fi
```

## travis 

github action已经很好了，我没有采用的原因是需要**熟悉成本**，**有些操作可能暂时做不到**。而可以预见性的travis都能很好的做到。因此目前的方案是采用travis。

**实现要求** ：

通过新建tag(只能是tag触发),自动编译前后端代码，发布github release,构建docker镜像，发布镜像。

 ```
language: java
jdk:
  - openjdk8

cache:
  directories:
    - $HOME/.m2
    - $HOME/.npm
    - $TRAVIS_BUILD_DIR/

dist: trusty
jobs:
  include:
    - stage: ui_build
      language: node_js
      node_js: 10.15.2
      script: cd KafkaCenter-Frontend && npm install && npm run build

    - stage: GitHubRelease
      install:
        - echo GitHubRelease
      script: mvn -Drevision=${TRAVIS_TAG#v} clean package -Dmaven.test.skip=true
      before_deploy:
        - mkdir -p $TRAVIS_BUILD_DIR/deploy
        - cp $TRAVIS_BUILD_DIR/KafkaCenter-Core/target/*.gz $TRAVIS_BUILD_DIR/deploy
        - rm -f $TRAVIS_BUILD_DIR/KafkaCenter-Core/target/*.gz
        - ls -l $TRAVIS_BUILD_DIR/deploy
      deploy:
        provider: releases
        api_key: $API_KEY
        file_glob: true
        skip_cleanup: true
        file: deploy/*.tar.gz
        on:
          tags: true
      after_deploy: rm -rf $TRAVIS_BUILD_DIR/deploy

    - stage: BuildDockerImageforRelease
      install:
        - echo Build Docker Image for Release
      before_script:
        - chmod +x ./docker/docker-build-release.sh
      script: ./docker/docker-build-release.sh

stages:
  - name: ui_build
    if: tag =~ /^v\d+\.\d+\.\d+.*$/
  - name: GitHubRelease
    if: tag =~ /^v\d+\.\d+\.\d+.*$/
  - name: BuildDockerImageforRelease
    if: tag =~ /^v\d+\.\d+\.\d+.*$/

notifications:
  email: true
 ```

在travis管理页面需要配置`API_KEY` 、`DOCKER_USERNAME` 、`DOCKER_PASSWORD`

## 参考

1. [Maven CI Friendly Versions](https://maven.apache.org/maven-ci-friendly.html)
2. [spring-boot-docker](https://spring.io/guides/topicals/spring-boot-docker)
3. [docs.travis-ci.com](https://docs.travis-ci.com/)

