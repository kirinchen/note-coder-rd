---
title: Spring Boot +Spring Data + MongoDB(docker base) 實作筆記
tags: [java, spring, mongodb, springboot]

---

# Spring Boot +Spring Data + MongoDB(docker base) 實作筆記

## 建置專案環境

* Spring Boot 2.3.0.RELEASE
* Docker
* Gradle
* Java 8

## 用Docker 啟動 mongodb 的開發環境

鍵入以下command
```shell=
 docker run \
  -d \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME="root" \
  -e MONGO_INITDB_ROOT_PASSWORD="root" \
  -e MONGO_INITDB_DATABASE="bzklog" \
  -it --memory="512m" \
  mongo:3.4.2
```

## Spring Boot 設定

### 專案設定

可以直接用 Spring boot started initializr 建置 : [連結](https://start.spring.io/#!type=gradle-project&language=java&platformVersion=2.6.0-M2&packaging=jar&jvmVersion=11&groupId=net.kirin&artifactId=mongodbdemo&name=mongodbdemo&description=Demo%20project%20for%20Spring%20Boot&packageName=net.kirin.mongodbdemo&dependencies=data-mongodb)

### Gradle 說明
如果是現有專案就要要加入以下套件
```groovy=
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
    
    compileOnly    'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
```

>> 如果是用上面Spring boot started initializr 就已經有包含了 可跳過
>> lombok 是用來簡化繁瑣的code


![](https://i.imgur.com/mOKja1t.png)

> 專案結構


## coding

### 設定application.properties
在 resources\application.properties
```javascript=
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.authentication-database=admin
spring.data.mongodb.username=root
spring.data.mongodb.password=root
spring.data.mongodb.database=bzklog
```


### 建立model

**在 net.kirin.mongodbdemo.model**

建立 Demo Entity Class
```java=
package net.kirin.mongodbdemo.model;

import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;

import java.util.Map;

@Data
@Document(collection = "demo")
public class Demo {
    @Id
    private long id;
    @Indexed(unique = true)
    private String name;
    private boolean eable;
    private Map var;
}
```

### 建立 Spring Data MongoRepository

**在 net.kirin.mongodbdemo.dao**

建立 Demo Dao Class

```java=
package net.kirin.mongodbdemo.dao;

import net.kirin.mongodbdemo.model.Demo;
import org.springframework.data.mongodb.repository.MongoRepository;

// No need implementation, just one interface, and you have CRUD, thanks Spring Data
public interface DemoRepository extends MongoRepository<Demo, Long> {
    Demo findFirstByName(String name);
}
```

### 建立測試範例 在 MongodbdemoApplication (Sprint boot 進入點)
```java=
package net.kirin.mongodbdemo;

import net.kirin.mongodbdemo.dao.DemoRepository;
import net.kirin.mongodbdemo.model.Demo;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableAutoConfiguration
@ComponentScan(basePackages = {"net.kirin"})
@SpringBootApplication
public class MongodbdemoApplication {

	public static void main(String[] args) {

		SpringApplication.run(MongodbdemoApplication.class, args);
	}

	@Bean
	CommandLineRunner init(DemoRepository domainRepository) {

		return args -> {
			Demo n = new Demo();
			n.setId(System.currentTimeMillis());
			n.setName("kirin");
			Map v = new HashMap();
			v.put("dd",123);
			n.setVar(v);
			domainRepository.save(n);

			Demo obj2 = domainRepository.findFirstByName("kirin");
			System.out.println(obj2);
		};

	}

}

```

run MongodbdemoApplication :

```shell=
Demo(id=1630746342799, name=kirin, eable=false, var={dd=123})
```

## 建立 dock-compose production 環境

### 建立與設定application-prod.properties
建立 resources\application-prod.properties
```javascript=
spring.data.mongodb.host=mongodb
```

### 建立 docker-compose.yml 
與 該專案同一層
```yaml=
version: '3'
services:

  mongodbdemo:
    build:
        context: ./mongodbdemo/
        dockerfile: Dockerfile
    ports: 
      - "8080:8080"
    depends_on:
      - "mongodb"      
  mongodb:
    image: mongo:3.4.2
    environment:
       - MONGO_INITDB_ROOT_USERNAME=root
       - MONGO_INITDB_ROOT_PASSWORD=root
       - MONGO_INITDB_DATABASE=bzklog       
    ports: 
      - "27017:27017"   
    deploy:
      resources:
        limits:
          cpus: '1.00'
          memory: 2048M         
```

### 建立DockerFile 
在該專案目錄下
```dockerfile=
FROM kirinddt/gradle-graalvm:latest AS build
 
COPY . /app
WORKDIR /app

RUN gradle :bootJar --refresh-dependencies 

FROM ghcr.io/graalvm/graalvm-ce:java11-20.3.1

# EXPOSE 8080
EXPOSE 8080 

RUN mkdir /app


COPY --from=build /app/build/libs/mongodbdemo-0.0.1-SNAPSHOT.jar /app/spring-boot-application.jar

ENTRYPOINT ["java","-Dspring.profiles.active=prod","-jar","/app/spring-boot-application.jar"]



```

### enjoy!!!
鍵入command
```shell=
docker-compose up --build
```

## source code (github)

**https://github.com/kirinchen/springboot-mongodb-dockerize**



###### tags: `java` `spring` `mongodb` `springboot`