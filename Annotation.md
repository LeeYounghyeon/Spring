## 4.1 어노테이션 설정 기초
### 4.1.1 Context 네임스페이스 추가
- 어노테이션 설정을 추가하려면 스프링 설정 파일의 루트 엘리먼트인 `<bean>`에 Context관련 네임스페이스와 스키마 문서의 위치를 등록해야 한다.
- [Namespace]-context 항목 체크

### 4.1.2 컴포넌트 스캔 설정
- 스프링 설정 파일에 애플리케이션에서 사용할 객체들을 `<bean>`등록하지 않고 자동으로 생성 하려면 `<context:component-scan>` 이라는 엘리먼트를 정의해야 한다.
- 이 설정을 추가하면 스프링 컨테이너는 클래스 패스에 있는 클래스들을 스캔하여 `@Component`가 설정된 클래스들을 자동으로 객체 생성한다.
```xml
<context:component-scan base-package="polymorphism"/>
```

### 4.1.3 @component
- `<context:component=scan>`을 설정했으면 이제 스프링 설정 파일에 클래스 들을 일일이 `<bean>` 엘리먼트로 등록할 필요가 없다.
```xml
<bean class="polymorphism.LgTV"/>
```
```java
@Component
public class LgTV impliments TV{
  publiv LgTV(){
    System.out.println("===> LgTV 객체 생성");
  }
}
```
- 스프링 컨테이너가 생성한 객체를 요청하려면, 요철할 때 사용할 아이디나 이름이 반드시 설정되어 있어야 한다.


```java
//1.spring 컨테이너를 구동한다.
AbstractApplicationContext factory = new GenericXmlApplicationContext("applicationContext.xml");

//2.Spring 컨테이너로부터 필요한 객체를 요청한다.
TV tv = (TV)factory.getBean("tv");
```
```xml
<bean id = "tv" class="polymorphism.LgTV"/>
```
```java
@Component("tv")
public class LgTV impliments TV{
  public LgTV(){
    System.out.println("===> LgTV 객체 생성");
  }
}
```

## 4.2 의존성 주입 설정
### 4.2.1 의존성 주입 어노테이션
- @Autowired
  - 주로 변수 위에 설정하여 해당 타입의 객체를 찾아 자동으로 할당한다.
- @Qualifier
  - 특정 객체의 이름을 이용하여 의존성 주입할 때 사용한다.
- @Inject
  - @Autowired와 동일한 기능을 제공한다.
- @Resource
  - @Autowired와 @Qualifier의 기능을 결합한 어노테이션이다.

### 4.2.2 @Autowired
- 생성자나 메소드, 멤버변수 위에 모두 사용할 수 있다.
- 스프링 컨체이너는 멤버 변수 위에 붙은 @Autowired를 확인하는 순간 해당 변수의 타입을 체크한다. 그리고 그 타입의 객체가 메모리에 존재하는 지를 확인 후 객체를 변수에 주입한다.
- @Autowired가 붙은 객체가 메모리에 없다면 컨테이너가 NoSuchBeanDefinitionException 을 발생시킨다.


LgTV.java
```java
package polymorphism;

@Component("tv")
public class LgTV impliments TV{
  @Autowired
  private Speaker speaker;
}
public LgTV(){
  System.out.println("===> LgTV 객체 생성");
}
public void powerOn(){
  System.out.println("LgTV---전원 켠다.");
}
public void powerOff(){
  System.out.println("LgTV--- 전원 끈다.");
}
public void volumeUp(){
  speaker.volumeUp();
}
public void volumeDown(){
  speaker.volumeDown();
}
```
- SonySpeaker 객체 생성
```xml
<bean id="sony" class="polymorphism.SonySpeaker"/>
```
```java
@Component("sony")
public class SonySpeaker implements Speaker{
  public SonySpeaker(){
    System.out.println("===> SonySpeaker 객체 생성");
  }
}
```

### 4.2.3 @Qualifier
- Speaker 타입의 객체가 두 개 이상일 때 컨테이너는 어떤 객체를 할당할지 스스로 판단할 수 없어서 에러가 발생한다.
- @Qualifier 어노테이션을 이요하면 의존성 주입될 객체의 아이디나 이름을 지정할 수 있다.

LgTV.java
```java
package polymorphism;

@Component("tv")
public class LgTV impliments TV{

  @Autowired
  @Qualifier("apple")
  private Speaker speaker;
}
public LgTV(){
  System.out.println("===> LgTV 객체 생성");
}
public void powerOn(){
  System.out.println("LgTV---전원 켠다.");
}
public void powerOff(){
  System.out.println("LgTV--- 전원 끈다.");
}
public void volumeUp(){
  speaker.volumeUp();
}
public void volumeDown(){
  speaker.volumeDown();
}
```

### 4.2.4 @Resource
- @Resource는 객체의 이름을 이용하여 의존성 주입을 처리한다.
- apple이라는 이름으로 메모리에 생성된 AppleSpeaker 객체를 speaker 변수에 할당하는 설정이다.

LgTV.java
```java
package polymorphism;

@Component("tv")
public class LgTV impliments TV{
  @Resource(name="apple")
  private Speaker speaker;
}
public LgTV(){
  System.out.println("===> LgTV 객체 생성");
}
public void powerOn(){
  System.out.println("LgTV---전원 켠다.");
}
public void powerOff(){
  System.out.println("LgTV--- 전원 끈다.");
}
public void volumeUp(){
  speaker.volumeUp();
}
public void volumeDown(){
  speaker.volumeDown();
}
```

### 4.2.5 어노테이션과 XML 설정 병행하여 사용하기
- XML설정과 어노테이션 설정은 장단점이 서로 상층한다.
- XML방식
  - 장점:자바 소스를 수정하지 않고 XML 파일의 설정만 변경하면 실행되는 Speaker을 교체 할 수 있어 유지보수가 편하다
  - 단점:XML소스를 해석 해야만 무슨 객체가 의존성 주입되는지를 확인할 수 있다.
- 어노테이션 기반 설정
  - 장점: XML설정에 대한 부담도 없고, 의존관계에 대한 정보가 자바 소스에 들어있어서 사용하기 편하다.
  - 단점: 의존성 주입할 객체의 이름이 자바 소스에 명시되어야 하므로 자바 소스를 수정하지 않고 Speaker을 교체할 수 없다.



  1. LgTV클래스의 speaker 변수를 원래대로 @Autowired 어노테이션만 설정한다.


  LgTV.java
  ```java
  package polymorphism;

  @Component("tv")
  public class LgTV impliments TV{
    @Autowired
    private Speaker speaker;
  }
  public LgTV(){
    System.out.println("===> LgTV 객체 생성");
  }
  public void powerOn(){
    System.out.println("LgTV---전원 켠다.");
  }
  public void powerOff(){
    System.out.println("LgTV--- 전원 끈다.");
  }
  public void volumeUp(){
    speaker.volumeUp();
  }
  public void volumeDown(){
    speaker.volumeDown();
  }
  ```
  2. Speaker 타입의 객체가 두개 이므로 에러가 나지만, @Component를 제거하여 객체가 자동으로 생성되는 것을 차단한다.

  ```java
  package polymorphism;
  public class SonySpeaker implements speaker{
    public SonySpeaker(){
      System.out.println("===> SonySpeaker 객체 생성됨");
      }
  }
  ```
  ``` xml
  <beans>
    <context:component-scan base-package="polymorphism"/>
    <bean class="polymorphism.SonySpeaker"/>
  </beans>
  ```
  ```java
  package polymorphism;
  public class AppleSpeaker implements speaker{
    public AppleSpeaker(){
      System.out.println("===> AppleSpeaker 객체 생성됨");
      }
  }
  ```

  ```xml
<beans>
  <context:component-scan base-package="polymorphism"/>
  <bean class="polymorphism.AppleSpeaker"/>
</beans>
```

-  LgTV에 설정한 @Autowired에 의해서 SonySpeaker 객체가 의존성 주입된다. 이후 AppleSpeaker로 교체할 때는 SonySpeaker를 AppleSpeaker로 수정하면 된다.


## 4.3 추가 어노테이션
- @Service
  - 비즈니스 로직을 처리하는 Service 클래스
- @Repository
  - 데이터베이스 연동을 처리하는 DAO 클래스
- @Controller
  - 사용자 요청을 제어하는 Controller 클래스
