기초 or 궁금했던 것을 정리하는 공간.
## pom.xml
- Project Object Model의 준말
- maven에서 라이브러리들이 XML태그 형태로 제공이 되는데 필요한 라이브러리의 태그를 pom.xml에 넣어주면 자동으로 dependencies에 *.jar 추가

## web.xml
- DispatherServlet의 설정은 웹 어플리케이션의 /WEB-INF/web.xml 파일에 추가하며, 서블릿과 매핑 정보를 추가하면 DispatherServlet 설정이 완료된다.

- `web.xml`서블릿 클래스는 JSP 페이지와 달리 설치뿐만 아니라 등록을 하는 과정을 필요로 한다. 여기서 서블릿 클래스를 등록하는 곳의 이름을 Web application deployment descriptor라고 하는데 (줄여서 DD-Deployment Descriptor) 이 역할을 하는 것이 바로 web.xml이다. web.xml 파일은 웹 애플리케이션 디렉터리마다 딱 하나씩만 존재할 수 있다.

- DD는 WAS 구동 시 /WEB-INF 디렉토리에 존재하는 web.xml을 읽어 웹 애플리케이션의 설정을 구성하기 위해 존재한다.

- web.xml을 작성하기 위해서 우선 루트 엘리먼트인 `<web-app></web-app>`을 만든다.

- 이 안에 웹 서버가 웹 브라우저로부터 URL을 받았을 때 서블릿 클래스를 찾아서 호출하기 위해 필요한 정보를 기록해야 한다.

- URL과 서블릿 클래스의 이름을 각각 `<servlet><servlet-class></servlet-class></servlet>`태그와`<servlet-mapping><url-pattern></url-pattern></servlet-mapping>`로 작성

```xml
<web-app>
	<servlet>
    	<servlet-name>
        	//서블릿의 이름
        </servlet-name>
    	<servlet-class>
    		//서블릿 클래스의 이름
        </servlet-class>
   	</servlet>
    <servlet-mapping>
    	<servlet-name>
        	//참조할 서블릿의 이름
        </servlet-name>
    	<url-pattern>
    		//mapping할 URL
        </url-pattern>
    </servlet-mapping>
</web-app>
```

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

- ContextLoaderListener 클래스는 스프링 설정 파일(디폴트에서 파일명 applicationContext.xml)을 로드하면 ServletContextListener 인터페이스를 구현하고 있기 때문에 ServletContext 인스턴스 생성 시(톰켓으로 어플리케이션이 로드된 때)에 호출된다. 즉, ContextLoaderListener 클래스는 DispatcherServlet 클래스의 로드보다 먼저 동작하여 비즈니스 로직층을 정의한 스프링 설정 파일을 로드한다.
- `<param-name>contextConfigLocation</param-name> ` : 빈 설정 파일을 하나 이상을 사용하거나, 파일 이름과 경로를 직접 지정해주고 싶다.

## dispatcher-servlet.xml
- ` <mvc:annotation-driven/>` : 어노테이션 사용을 도와줌
- ` <bean:component-scan base-package="com.example2"/>` : autowired할 패키지 경로
- DispatherServlet은 이 뷰 이름과 매칭되는 뷰 구현체를 찾기 위해 ViewResolver를 사용한다.
```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```


## 개발 순서


1. DB 테이블 생성
2. VO 클래스 설계 (domain)
3. DAO 인터페이스 작성 - 실행해야 하는 기능을 인터페이스로 정의
4. XML Mapper의 생성과 필요한 DTD 추가 및 SQL문 작성
5. MyBatis에서 작성된 XML Mapper를 MyBatis가 동작할 때 인식하도록 설정
6. DAO인터페이스 구현하는 DAO 클래스 작성 - SqlSessionTemplate 이용

## Mybatis란?


- 객체지향 언어인 자바의 관계형 데이터 베이스 프로그래밍을 좀더 쉽게 할수 있게 도와주는 개발 프레임워크

- 자바에선 데이터베이스 프로그래밍을 하기 위해 JDBC(자바에서 제공하는 데이터베이스 프로그래밍 API)를 제공

- JDBC는 관계형 데이터 베이스를 사용하기 위해 다양한 API를 제공

- 다양한 관계형 데이터베이스를 지원하기 위해 JDBC는 세부적인 작업이 가능하게 작업별로 각각의 메소드를 호출하게된다. 이러한 사항들은 다수의 메소드를 호출하고 관련된 객체를 해제해야하는 단점이 존재

-->Mybatis는 JDBC보다 좀더 편하게 사용하기 위해 개발되었음

## sqlSessionFactoryBean

- SqlSessionFactoryFactory 클래스 : SqlSession을 만드는 역할

- Dao는 Factory을 멤버로 유지하면서 필요할 때 SqlSession을 open해서 사용하고 다쓰면 sqlSession를 close

- SqlSession클래스 : sql문을 실제 호출해주는 역할(필요할때 open하고 close해줘야함)

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
      <property name="dataSource" ref="datasource"/>
      <property name="mapperLocations" value="classpath:mybatis/mapper/**/*.xml"/>
  </bean>
```

## BoardMapper.xml
- XML Mapper를 작성할 때는 namespace라는 것의 설정에 조심해야한다. 클래스의 패키지와 유사한 용도로, MyBatis 내에서 원하는 SQL 문을 찾아서 실행할 때 동작하는 것이다


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
## HttpServletRequest
- 하나의 요청에서 HttpServletRequest 객체가 소멸하기 까지 상태정보를 유지하고자 할 때, 한번의 요청으로 실행된 페이지끼리 정보를 공유하고자 할 때 사용되며, 디스패처에 의한 요청재지정을 하기 전 HttpServletRequest 객체의 setAttribute( ) 메소드로 데이터를 등록하고 요청 재지정으로 HttpServletRequest 객체가 전달된 페이지에서 getAttribute( ) 메소드로 추출할 수 있다.
