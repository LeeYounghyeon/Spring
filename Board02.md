## 실습02
### Step1

- DB생성(Mysql사용)


```sql
CREATE TABLE `Board` (
  `SEQ` int(5) NOT NULL,
  `TITLE` varchar(200) DEFAULT NULL,
  `WRITER` varchar(20) DEFAULT NULL,
  `CONTENT` varchar(2000) DEFAULT NULL,
  `REGDATE` datetime DEFAULT CURRENT_TIMESTAMP,
  `CNT` int(5) DEFAULT '0',
  PRIMARY KEY (`SEQ`)
)
```
- pom.xml에 DB와 JDBC, Mybatis를 사용하기 위한 내용을 추가해준다.

- pom.xml에 추가할 내용

```xml
<!-- Spring-jdbc -->
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${org.springframework-version}</version>
    </dependency>

    <!-- MySQL -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.27</version>
    </dependency>

    <!-- MyBatis 3.4.1 -->
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.1</version>
    </dependency>


    <!-- MyBatis-Spring -->
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.0</version>
    </dependency>

    <!-- Servlet -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>
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
```

- dispatcher-servlet.xml에 내용 추가


```xml
<bean id="datasource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://localhost:3306/board"/>
      <property name="username" value="root"/>
      <property name="password" value="dudgus94"/>
  </bean>

  <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
      <property name="dataSource" ref="datasource"/>
      <property name="mapperLocations" value="classpath:mybatis/mapper/**/*.xml"/> // Mapper xml파일 위치
  </bean>

  <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
      <property name="basePackage" value="com.example.repository" /> // repository 인터페이스 위치
      <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
  </bean>

```

- com.example - repository - BoardRepository.java 인터페이스 생성
- BoardRepository.java


```java
package com.example.repository;

import com.example.domain.Board;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface BoardRepository {
    List<Board> findAll();
}
```
- resources - mybatis - mapper - BoardMapper.xml 생성
- BoardMapper.xml

```xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.repository.BoardRepository">
    <select id="findAll" resultType="com.example.domain.Board">
        SELECT * from Board;
    </select>
</mapper>
```
- com.example - domain - Board.java 생성
- Board.java


```java
package com.example.domain;

import java.time.LocalDateTime;

public class Board {

    private int seq;
    private String title;
    private String writer;
    private String content;
    private LocalDateTime regDate;
    private int cnt;

    public Board(){}

    public Board(int seq, String title, String writer, String content, LocalDateTime regDate, int cnt) {
        this.seq = seq;
        this.title = title;
        this.writer = writer;
        this.content = content;
        this.regDate = regDate;
        this.cnt = cnt;
    }

    public int getSeq() {
        return seq;
    }

    public void setSeq(int seq) {
        this.seq = seq;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getWriter() {
        return writer;
    }

    public void setWriter(String writer) {
        this.writer = writer;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public LocalDateTime getRegDate() {
        return regDate;
    }

    public void setRegDate(LocalDateTime regDate) {
        this.regDate = regDate;
    }

    public int getCnt() {
        return cnt;
    }

    public void setCnt(int cnt) {
        this.cnt = cnt;
    }
}

```

- TestController.java에 내용추가

```java
package com.example.controller;

import com.example.domain.Board;
import com.example.repository.BoardRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.List;

@Controller
public class TestController {
    @Autowired
    private BoardRepository boardRepository;

    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }

    @RequestMapping("/board")
    public String boards(Model model){
        model.addAttribute("list",boardRepository.findAll());
        return "board";
    }
}
```

- views 안에 board.jsp를 생성 해 결과가 잘 넘어오는지 확인하기
- localhost:8080/board를 써주면 된다.


```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <c:forEach items="${list}" var="board">
       ${board.seq}
    </c:forEach>
</body>
</html>

```
여기서 ${board.seq}값이 변수값이 아닌 "${board.seq}"그대로 출력되는 오류가 났었다.
web.xml에 내용을 추가해줘서 해결!
```xml
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
```

### 내용 정리
- `pom.xml`의 dependency는 https://mvnrepository.com/ 여기서 검색해서 추가해준다.


- `dispatcher-servlet`의 내용은 spring mybatis 연결 검색


- BoardRepository 인터페이스 - 컨트롤러에서는 리소스를 호출하기 힘들다.  컨트롤러 -> BoardRepository - (mybatis 가 연결) - BoardMaaper.xml


- `<bean:component-scan base-package="com.example"/>` : 어노테이션과 bean을 매핑시켜준다.


- `@Controller`는 클라이언트로부터 전달되어진 데이터를 가공하기 위한 Controller임을 명시하며 `@RequestMapping`을 통해 경로를 설정하게 된다.


- `@Repository`는 해당 클래스가 데이터 베이스에 접근하는 클래스임을 명시한다.


- `@Autowired`는 원하는 bean을 찾아서 주입해준다.


- `addAttribute("이름", "넘길값")` -> `model.addAttribute("user", user);`  // user 이름으로 user 값을 넣어준다.


- `<mapper namespace="com.example.repository.BoardRepository"> -> 매칭될 Repository 인터페이스`


- `<select id="findAll" resultType="com.example.domain.Board"> -> id는 repository 인터페이스 메소드와 매칭 , resultType는 조회 결과 타입`

db 까지 연결!
