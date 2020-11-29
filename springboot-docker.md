# 使用spotify的docker-maven-plugin插件将SpringBoot项目打包为Docker镜像

此项目的代码已经放到GitHub中，地址[https://github.com/twingao/springboot-docker](https://github.com/twingao/springboot-docker)。先展示SpringBoot项目的目录结构。

    tree springboot-docker/
    springboot-docker/
    ├── pom.xml
    └── src
        └── main
            ├── docker
            │   └── Dockerfile
            ├── java
            │   └── com
            │       └── twingao
            │           ├── controller
            │           │   └── DemoController.java
            │           └── SpringbootDockerApplication.java
            └── resources
                └── application.yml
    
    8 directories, 5 files

pom.xml中引入docker-maven-plugin插件。

    cat pom.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.2.4.RELEASE</version>
            <relativePath/>
        </parent>
    
        <groupId>com.twingao</groupId>
        <artifactId>springboot-docker</artifactId>
        <version>1.0.0</version>
        <name>springboot-docker</name>
        <description>springboot docker demo</description>
    
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>
    
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>com.spotify</groupId>
                    <artifactId>docker-maven-plugin</artifactId>
                    <version>0.4.13</version>
                    <configuration>
                        <imageName>twingao/${project.name}:${project.version}</imageName>
                        <dockerDirectory>src/main/docker</dockerDirectory>
                        <resources>
                            <resource>
                                <targetPath>/</targetPath>
                                <directory>${project.build.directory}</directory>
                                <include>${project.build.finalName}.jar</include>
                            </resource>
                        </resources>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </project>

在src/main下创建docker目录，Dockerfile就放在该目录下。

    cat src/main/docker/Dockerfile
    FROM openjdk:8-jdk-alpine
    
    VOLUME /opt/tmp
    
    ADD springboot-docker-1.0.0.jar springboot-docker.jar
    
    ENV JAVA_OPTS=""
    
    ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /springboot-docker.jar

编译并构建Docker镜像。

    mvn clean package docker:build
    [INFO] Scanning for projects...
    [INFO]
    [INFO] -------------------< com.twingao:springboot-docker >--------------------
    [INFO] Building springboot-docker 1.0.0
    [INFO] --------------------------------[ jar ]---------------------------------
    [INFO]
    [INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ springboot-docker ---
    [INFO] Deleting /root/code/springboot-docker/target
    [INFO]
    [INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ springboot-docker ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] Copying 1 resource
    [INFO] Copying 0 resource
    [INFO]
    [INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ springboot-docker ---
    [INFO] Changes detected - recompiling the module!
    [INFO] Compiling 2 source files to /root/code/springboot-docker/target/classes
    [INFO]
    [INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ springboot-docker ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] skip non existing resourceDirectory /root/code/springboot-docker/src/test/resources
    [INFO]
    [INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ springboot-docker ---
    [INFO] No sources to compile
    [INFO]
    [INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ springboot-docker ---
    [INFO] No tests to run.
    [INFO]
    [INFO] --- maven-jar-plugin:3.1.2:jar (default-jar) @ springboot-docker ---
    [INFO] Building jar: /root/code/springboot-docker/target/springboot-docker-1.0.0.jar
    [INFO]
    [INFO] --- spring-boot-maven-plugin:2.2.4.RELEASE:repackage (repackage) @ springboot-docker ---
    [INFO] Replacing main artifact with repackaged archive
    [INFO]
    [INFO] --- docker-maven-plugin:0.4.13:build (default-cli) @ springboot-docker ---
    [INFO] Copying /root/code/springboot-docker/target/springboot-docker-1.0.0.jar -> /root/code/springboot-docker/target/docker/springboot-docker-1.0.0.jar
    [INFO] Copying src/main/docker/Dockerfile -> /root/code/springboot-docker/target/docker/Dockerfile
    [INFO] Building image twingao/springboot-docker:1.0.0
    Step 1/5 : FROM openjdk:8-jdk-alpine
    
     ---> a3562aa0b991
    Step 2/5 : VOLUME /opt/tmp
    
     ---> Using cache
     ---> ce2415a919e7
    Step 3/5 : ADD springboot-docker-1.0.0.jar springboot-docker.jar
    
     ---> 2717f349a049
    Step 4/5 : ENV JAVA_OPTS=""
    
     ---> Running in e7292f45d4eb
    Removing intermediate container e7292f45d4eb
     ---> 5afb54716215
    Step 5/5 : ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /springboot-docker.jar
    
     ---> Running in e5751ecddf88
    Removing intermediate container e5751ecddf88
     ---> 8b3409a5562e
    ProgressMessage{id=null, status=null, stream=null, error=null, progress=null, progressDetail=null}
    Successfully built 8b3409a5562e
    Successfully tagged twingao/springboot-docker:1.0.0
    [INFO] Built twingao/springboot-docker:1.0.0
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  36.895 s
    [INFO] Finished at: 2020-02-08T15:36:10+08:00
    [INFO] ------------------------------------------------------------------------

查看生成的镜像。

    docker images
    REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
    twingao/springboot-docker   1.0.0               8b3409a5562e        48 seconds ago      122MB
    openjdk                     8-jdk-alpine        a3562aa0b991        9 months ago        105MB
    
运行容器，验证一下。

    docker run -d --name springboot-docker -p 8080:8080 twingao/springboot-docker:1.0.0
    b6fe11c8726a44b24c90691b48cd6f63c9548eab98161261d38d25102bd387f8
    
    docker ps
    CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS              PORTS                    NAMES
    98cfc8bdbc2d        twingao/springboot-docker:1.0.0   "/bin/sh -c 'exec ja…"   20 seconds ago      Up 18 seconds       0.0.0.0:8080->8080/tcp   springboot-docker
    
    curl http://localhost:8080/demo
    Hello World

打开浏览器，访问[http://192.168.1.35:8080/demo](http://192.168.1.35:8080/demo)，也可以访问。

可以进一步将构建的Docker镜像push到hub.docker.com上。当然首先应该有一个docker hub账号。

先修改maven的配置文件,增加docker hub服务配置。

    cat /usr/local/maven-3.6.2/conf/settings.xml
    ...
        <server>
          <id>docker-hub</id>
          <username>docker-hub账号</username>
          <password>docker-hub密码</password>
          <configuration>
            <email>docker-hub邮件地址</email>
          </configuration>
        </server>
      </servers>
    ...

docker-maven-plugin插件增加docker hub配置。

    cat pom.xml
    ...
                <plugin>
                    <groupId>com.spotify</groupId>
                    <artifactId>docker-maven-plugin</artifactId>
                    <version>0.4.13</version>
                    <configuration>
                        <imageName>twingao/${project.name}:${project.version}</imageName>
                        <dockerDirectory>src/main/docker</dockerDirectory>
                        <resources>
                            <resource>
                                <targetPath>/</targetPath>
                                <directory>${project.build.directory}</directory>
                                <include>${project.build.finalName}.jar</include>
                            </resource>
                        </resources>
                        <serverId>docker-hub</serverId>
                        <registryUrl>https://index.docker.io/v1/</registryUrl>
                    </configuration>
                </plugin>
    ...

编译、构建Docker镜像并push到docker hub中。

    mvn clean package docker:build -DpushImage
    [INFO] Scanning for projects...
    [INFO]
    [INFO] -------------------< com.twingao:springboot-docker >--------------------
    [INFO] Building springboot-docker 1.0.0
    [INFO] --------------------------------[ jar ]---------------------------------
    [INFO]
    [INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ springboot-docker ---
    [INFO] Deleting /root/code/springboot-docker/target
    [INFO]
    [INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ springboot-docker ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] Copying 1 resource
    [INFO] Copying 0 resource
    [INFO]
    [INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ springboot-docker ---
    [INFO] Changes detected - recompiling the module!
    [INFO] Compiling 2 source files to /root/code/springboot-docker/target/classes
    [INFO]
    [INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ springboot-docker ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] skip non existing resourceDirectory /root/code/springboot-docker/src/test/resources
    [INFO]
    [INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ springboot-docker ---
    [INFO] No sources to compile
    [INFO]
    [INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ springboot-docker ---
    [INFO] No tests to run.
    [INFO]
    [INFO] --- maven-jar-plugin:3.1.2:jar (default-jar) @ springboot-docker ---
    [INFO] Building jar: /root/code/springboot-docker/target/springboot-docker-1.0.0.jar
    [INFO]
    [INFO] --- spring-boot-maven-plugin:2.2.4.RELEASE:repackage (repackage) @ springboot-docker ---
    [INFO] Replacing main artifact with repackaged archive
    [INFO]
    [INFO] --- docker-maven-plugin:0.4.13:build (default-cli) @ springboot-docker ---
    [INFO] Copying /root/code/springboot-docker/target/springboot-docker-1.0.0.jar -> /root/code/springboot-docker/target/docker/springboot-docker-1.0.0.jar
    [INFO] Copying src/main/docker/Dockerfile -> /root/code/springboot-docker/target/docker/Dockerfile
    [INFO] Building image twingao/springboot-docker:1.0.0
    Step 1/5 : FROM openjdk:8-jdk-alpine
    
     ---> a3562aa0b991
    Step 2/5 : VOLUME /opt/tmp
    
     ---> Using cache
     ---> ce2415a919e7
    Step 3/5 : ADD springboot-docker-1.0.0.jar springboot-docker.jar
    
     ---> 2535eff7e177
    Step 4/5 : ENV JAVA_OPTS=""
    
     ---> Running in 7c7041df4f51
    Removing intermediate container 7c7041df4f51
     ---> d87e0602b118
    Step 5/5 : ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /springboot-docker.jar
    
     ---> Running in 1ea77b69a6cd
    Removing intermediate container 1ea77b69a6cd
     ---> 9ccd40c50a80
    ProgressMessage{id=null, status=null, stream=null, error=null, progress=null, progressDetail=null}
    Successfully built 9ccd40c50a80
    Successfully tagged twingao/springboot-docker:1.0.0
    [INFO] Built twingao/springboot-docker:1.0.0
    [INFO] Pushing twingao/springboot-docker:1.0.0
    The push refers to repository [docker.io/twingao/springboot-docker]
    fa517bd9ccf3: Pushed
    ceaf9e1ebef5: Layer already exists
    9b9b7f3d56a0: Layer already exists
    f1b5933fe4b5: Layer already exists
    1.0.0: digest: sha256:147e1d4475fc74a083bcf50e2a96abd13e8f796fc56b1d5445291fecd7ed8880 size: 1159
    null: null
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  52.408 s
    [INFO] Finished at: 2020-02-08T21:07:21+08:00
    [INFO] ------------------------------------------------------------------------