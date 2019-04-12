# web应用部署


## 1.应用程序编译

对于使用spring boot框架编写的web服务应用程序，使用下面的语句编译
```
gradlew build
```
将生成jar文件拷贝到服务器中


### 本地运行应用

因为spring boot 内置了tomcat框架，所以不需要安装tomcat，直接运行程序就行。

```
java -jar ****.jar
```


## 服务器端部署应用程序

服务器端一般使用docker容器化引擎进行部署

参考docker资料

### 方案1:部署jar包

编写Dockerfile
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

编译镜像时在参数中指定jar包位置
```
docker build --build-args=target/*.jar -t unicloud .
```
### 方案2:解压后部署

``` shell
$ mkdir target/dependency
$ (cd target/dependency; jar -xf ../*.jar)
$ docker build -t unicloud .
```
Dockerfile写法

``` Dockerfile
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","seu.lab601.UniCloud.UniCloudApplication"]
```




## 4.使用nginx实现负载均衡


## 5.使用docker-compose部署



编写docker-compose.yml配置脚本
``` docker-compose
version: '3.1'
services:
  db:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 6666
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    ports:
      - 3306:3306
    volumes:
      - ./data:/var/lib/mysql
  unicloud:
    restart: always
    image: test1
    container_name: unicloud
    ports:
      - 8080:8080
    environment:
      TZ: Asia/Shanghai
```
