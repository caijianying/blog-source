---
title: maven打包案例
date: 2016-10-24 21:20:13
tags: 开发笔记
categories:
- java
---
## 一、spring-boot-maven-plugin
使用案例如下：
```
<plugin>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-maven-plugin</artifactId>
		<executions>
			<execution>
				<goals>
					<goal>repackage</goal>
				</goals>
			</execution>
		</executions>
		<configuration>
			<mainClass>com.aibibang.AppMain</mainClass>
			<addResources>true</addResources>
			<excludes>
				<exclude>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-devtools</artifactId>
				</exclude>
			</excludes>
		</configuration>
</plugin>
```
## 二、自定义打包
在开发过程中经常需要将依赖包与代码包分离，配置文件与代码包分离，这样更便于部署与修改参数。依次案例基于以上场景展开，借助于maven-jar-plugin与maven-assembly-plugin。

pom内容如下：
```
<build>
		<plugins>
			<plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-jar-plugin</artifactId>  
                <configuration>  
                    <archive>    
                        <manifest>  
                            <mainClass>com.aibibang.AppMain</mainClass>  
                            <addClasspath>true</addClasspath>  
                            <classpathPrefix>lib/</classpathPrefix>  
                        </manifest>
                    </archive>  
                    <excludes>  
                       <exclude>*.properties</exclude>
                       <exclude>*.xml</exclude>
                    </excludes>  
                </configuration>  
            </plugin> 
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-assembly-plugin</artifactId>
				<configuration>
					<descriptors>
						<descriptor>package.xml</descriptor>
					</descriptors>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```
package.xml内容：
```
<assembly>
    <id>bin</id>
    <!-- 最终打包成一个用于发布的zip文件 -->
    <formats>
        <format>tar.gz</format>
    </formats>

    <fileSets>
    	<!-- 打包jar文件 -->
        <fileSet>
            <directory>${project.build.directory}</directory>
            <outputDirectory></outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>
        <!-- 把项目相关的启动脚本，打包进zip文件的bin目录 -->
        <fileSet>
            <directory>${project.basedir}/src/main/bash</directory>
            <outputDirectory>/</outputDirectory>
            <includes>
                <include>*</include>
            </includes>
        </fileSet>
        
        <!-- 把项目的配置文件，打包进zip文件的config目录 -->
        <fileSet>
            <directory>${project.build.directory}/classes</directory>
            <outputDirectory>conf</outputDirectory>
            <includes>
                <include>*.properties</include>
                <include>*.xml</include>
            </includes>
        </fileSet>
    </fileSets>
    <!-- 把项目的依赖的jar打包到lib目录下 -->
     <dependencySets>  
        <dependencySet>  
            <outputDirectory>lib</outputDirectory>  
            <scope>runtime</scope>  
            <excludes>  
                <exclude>${groupId}:${artifactId}</exclude>  
            </excludes>  
        </dependencySet>  
    </dependencySets>  
</assembly>
```
maven-jar-plugin只是讲代码打成一个jar,而对部署包的构建是由assembly插件完成的。

结合启动、停止脚本即可高效便捷的部署一个项目。

start.sh:
```
#!/bin/sh
export JAVA_HOME=$JAVA_HOME
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
PIDFILE=service.pid
ROOT_DIR="$(cd $(dirname $0) && pwd)"
CLASSPATH=./*:$ROOT_DIR/lib/*:$ROOT_DIR/conf/
JAVA_OPTS="-Xms512m -Xmx1024m -XX:+UseParallelGC"
MAIN_CLASS=com.aibibang.AppMain


if [ ! -d "logs" ]; then
   mkdir logs
fi

if [ -f "$PIDFILE" ]; then
    echo "Service is already start ..."
else
    echo "Service  start ..."
    nohup java $JAVA_OPTS -cp $CLASSPATH $MAIN_CLASS 1> logs/server.out 2>&1  &
    printf '%d' $! > $PIDFILE
fi

```
stop.sh:
```
#!/bin/sh
PIDFILE=service.pid

if [ -f "$PIDFILE" ]; then
     kill -9 `cat $PIDFILE`
     rm -rf $PIDFILE
else
    echo "Service is already stop ..."
fi
```