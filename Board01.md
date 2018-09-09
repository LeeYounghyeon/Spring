## 실습 01
### 스프링 프로젝트 만들기
- [File] - [New] - [project]
- [maven]에서 Create from archetype를 체크 한 뒤 mave-webapp선택

### 디펜던시 추가
- pom.xml에 필요한 디펜던시를 추가해준다.


```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>example</groupId>
  <artifactId>example</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>example Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <org.springframework-version>4.3.18.RELEASE</org.springframework-version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

    <!-- 스프링 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${org.springframework-version}</version>
    </dependency>

    <!-- 스프링 web mvc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${org.springframework-version}</version>
    </dependency>

    <!-- Servlet -->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>


  </dependencies>

  <build>
    <finalName>example</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>

        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.7.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.20.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>8</source>
          <target>8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>

```

### 소스, 리소스 디렉토리 지정
- main에 java, resources 디렉토리를 추가한다.
- command + ; (단축키:intellij를 사용한다.)를 누르기
- Java는 Source , resources는 Resources를 선택해준다. (Mark as에 있다.)

### 각 폴더, 파일 생성
- java 밑에 com.example 패키지 생성
- com.example 안에 controller 패키지 생성 후 TestController.java를 생성해준다.
- TestController.java

```java
package com.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class TestController {

    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }
}

```

- resources에서 spring 패키지 생성 후 그 안에 dispatcher-servlet.xml, applicationContext.xml을 생성한다.(new- XmlConfigurationFile-springConfig으로 xml 생성)
- dispatcher-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:bean="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven/>
    <bean:component-scan base-package="com.example"/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

- webapp에서 WEB-INF안에 views 디렉토리 생성 후 안에 hello.jsp 를 생성해준다.
- WEB-INF 안 web.xml 작성하기

- web.xml


```xml
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

<display-name>Archetype Created Web Application</display-name>

  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/applicationContext.xml</param-value>
  </context-param>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring/dispatcher-servlet.xml</param-value>
    </init-param>
  </servlet>


  <servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```


### 톰캣 연결하기
- [Run] - [Edit configurations] 에서 왼쪽 상단에 있는 +를 누른다.
- Tomcat server - local 선택해주기
- Application Server에 Configure를 누르고 Tomcat이 있는 디렉토리를 지정해준다
- 오른쪽 아래의 fix누른 후 war 또는 war exploded artifact를 선택하면 완료

### 내용 정리
- Spring의 request처리 과정


1. 클라이언트의 요청을 Dispatcher servlet이 먼저 받는다.
2. Dispatcher가 요청을 받고 Handler Mapping을 통해서 Controller를 선별한다.
3. Dispatcher는 선별한 Controller에게 request를 보낸다.
4. controller은 비즈니스로직을 처리 후 Dispatcher에게 model과 view이름을 보낸다.
4. Dispatcher는 받은 View이름을 View Resolver에게 보내 맞는 View를 받는다.
5. Dispatcher는 View에게 Model을 보내고 View는 Model을 바인딩 한 후 html형태로 변환된 결과를 다시 보낸다.
6. Dispatcher는 Client에게 요청에 대한 결과를 보여준다.

기본 준비 끝!
