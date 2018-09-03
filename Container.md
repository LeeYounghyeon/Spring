## 2.1 스프링 IoC 시작하기
### 2.1.1 스프링 설정 파일 생성
* 프로젝트 생성 -> [new] -> [Other] -> [Spring Bean Configurations File] -> 파일 이름 입력 -> [Finish]

- 앞서 작성한 TV예제를 스프링 설정 파일에 등록하기


applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.Springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
       xsi:schemaLocation="http://www.Springframework.org/schema/beans/Spring-beans.xsd">

       <bean id="tv" class="polymorphism.SamsungTV"/>
</beans>
```

- <bean>엘리먼트에서 가장 중요한 것은 class 속성값 -> 패키지 경로가 포함된 전체 클래스 경로를 지정


### 2.1.2 스프링 컨테이너 구동 및 테스트
- TV 객체를 테스트 하는 클라이언트 작성

TVUser.java
```java
package polymorphism;

public class TVUser{
    public static void main(String[] args){
      //1. Spring 컨테이너 구동한다.
      AbstractApplicationContext factory = new GenericXmlApplicationContext("applicationContext.xml");
    }
}
```
위 소스는 환경설정 파일인 applicationContext.xml을 로딩하여 스프링 컨테이너 중 하나인 GenericXmlApplicationContext를 구동했다.

SamsungTV.java

```java
package polymorphism;

public class SamsungTV implements TV{

  //추가사항 : 객체가 언제 생성되어지는지 확인하기 위해서
  public SamsungTV(){
    System.our.println("===> SamsungTV 객체 생성 ");
  }
	public void powerOn(){
    	System.out.println("SamsungTV ---전원 켠다.");
    }

    public void powerOff(){
    	System.out.println("SamsungTV ---전원 끈다.");
    }

    public void volumeUp(){
    	System.out.println("SamsungTV ---소리 올린다.");
    }

    public void volumeDown(){
    	System.our.println("SamsungTV ---소리 내린다.");
    }
}
```

TVUser.java
```java
package polymorphism;

public class TVUser{
	public static void main(String[] args){

    //1. Spring 컨테이너 구동한다.
    AbstractApplicationContext factory = new GenericXmlApplicationContext("applicationContext.xml");

      //2.Spring 컨테이너로부터 필요한 객체를 요청(Lookup)한다.
      TV tv = (TV)factory.getBean("tv");

      tv.powerOn();
      tv.volumeUp();
      tv.volumeDown();
      tv.powerOff();

      //3.Spring 컨테이너를 종료한다.
      factory.close();
    	}
	}
}
```

1. TVUser 클라이언트가 스프링 설정 파일을 로딩하여 컨테이너 구동
2. 스프링 설정 파일에 <bean> 등록된 SamsungTV 객체 생성
3. getBean() 메소드로 이름이 'tv'인 객체를 요청
4. SamsungTV 객체 반환

- 이제 TV를 LgTV로 변경할 때, applicationContext.xml 파일만 수정하면 된다. -> 유지보수가 더 편해졌다.


### 2.1.3 스프링 컨테이너의 종류
스프링에서는 BeanFactory와 이를 상속한 ApplicationContext 두 가지 유형의 컨테이너를 제공한다.
- BeanFactory
    - 스프링 설정 파일에 등록된 <bean> 객체를 생성하고 관리하는 가장 기본적인 컨테이너 기능 제공
    - 컨테이너가 구동될 때 <bean> 객체를 생성하는 것이 아니라, 클라이언트의 요청에 의해서만 객체 생성

- ApplicationContext
  - <bean>객체 관리 기능 외에도 트렌잭션 관리나 메세지 기반의 다국어 처리 등 다양한 기능 지원
  - 컨테이너가 구동되는 시점에 <bean> 등록된 클래스들을 객체 생성하는 즉시 로딩 방식으로 동작
  - `GenericXmlApplicationContext`: 파일 시스템이나 클래스 경로에 있는 XML 설정 파일을 로딩하여 구동하는 컨테이너
  - `XmlWebApplicationContext`: 웹 기반의 스프링 애플리케이션을 개발할 때 사용하는 컨테이너


## 2.2 스프링 XML 설정
### 2.2.1 < beans > 루트 엘리먼트
  - `beans` 저장소에 해당하는 XML 설정 파일을 참조하여 `bean`의 생명주기를 관리하고 여러가지 서비스 제공한다.
  - 가장 중요한 역할을 담당한다.
  - 파일을 정확하게 작성하고 관리하는 것이 중요하다.
  - 루트 엘리먼트로 사용되어야한다.


### 2.2.2 < import > 엘리먼트
- 분리하여 작성한 설정 파일들을 하나로 통합할 때 사용한다.
- 여러 스프링 설정 파일을 포함함으로써 한 파일에 작성하는 것과 같은 효과를 낼 수 있다.
```xml
<import resource="context-datasource.xml">
```
### 2.2.3 < bean > 엘리먼트
- 스프링 설정 파일에 클래스를 등록할 때 사용된다.
- `id` 속성은 생략할 수 있지만 `class` 속성은 필수이다.
- 정확한 패키지 경로와 클래스 이름을 지정해야 한다.
- `id`속성은 유일해야한다.
- `name`속성은 특수기호가 포함된 아이디를 지정할 때 사용한다.
-
```xml
<bean class="ploymorphism.SamsungTV" name="http://www.naver.com"> </bean>
```

### 2.2.4 < bean > 엘리먼트 속성
- init-method 속성
  - 멤버변수를 초기화하기 위해 사용된다.
  - 객체를 생성한 후에 멤버변수 초기화 작업이 필요하다면, init() 메소드를 사용한다.
```xml
<bean id="tv" class="polymorphism.SamsungTV" init-method="initMethod"/>
```
SamsungTV.java
```java
//추가
public void initMethod() {
  System.out.println("객체 초기화 작업처리..");
}
```

- destroy-method 속성
  - 스프링 컨테이너가 객체를 삭제하기 직전에 호출될 임의의 메소드를 지정
```xml
<bean id="tv" class="polymorphism.SamsungTV" destroy-method="destroyMethod"/>
```
SamsungTV.java
  ```java
  public void destroyMethod(){
    System.out.println("객체 삭제 전에 삭제 될 로직 처리...");
  }
  ```

- lazy-init 속성
  - 자주 사용되지도 않으면서 메모리를 많이 차지하여 시스템에 부담을 주는 경우에 사용된다.
  - 해당 `<bean>`이 사용되는 시점에 객체를 생성한다.
  - `lazy-init="true"`로 설정하면 스프링 컨테이너는 해당 `<bean>`을 미리 생성하지 않고, 클라이언트가 요청하는 시점에 생성한다.
  ```xml
  <bean id="tv" class="polymorphism.SamsungTV" lazy-init="true"/>
  ```

- scope 속성
  - 컨테이너가 생성한 `bean`을 어느 범위에 사용할 수 있는지 지정하게 해준다.
  - 스프링 컨테이너에 의해 단 하나만 생성되어 운용되도록 한다.
  - 여러번 요청해도 메모리에 객체는 하나만 생성되어 유지된다.
  - `prototype` 으로 지정하면, `bean`이 요청될 때 마다 새로운 객체를 생성하여 반환한다.
  ```xml
  <bean id="tv" class="polymorphism.SamsungTV" scope="singleton"/>
  ```
