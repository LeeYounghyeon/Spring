제 5장
## 5.1 AOP 이해하기
- 응집도와 관련된 기능이다.
- 공통 코드를 효율적으로 관리하게 해준다.

### 5.1.1 AOP 라이브러리 추가
- AOP를 적용하기 위해서는 pom.xml 파일을 수정하여 AOP관련 라이브러리를 추가한다.

```xml
<!-- Aspect -->
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjrt</artifactId>
  <version>${org.aspectj-version}</version>
</dependency>
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjwaver</artifactId>
  <version>1.8.8</version>
</dependency>

```



### 5.1.2 네임스페이스 추가 및 AOP 설정
- AOP설정을 추가하려면 AOP에서 제공하는 엘리먼트들을 사용해야 한다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org.schema/aop"
  xsi:schemaLocation = "http://www.springframework.org/schema/AOP http://www.springframework.org/schema/aop/spring-aop-4.2.xsd">
```

## 5.2 AOP 용어 정리
### 5.2.1 조인포인트(jointpoint)
- 클라이언트가 호출하는 모든 비즈니스 메소드
- 클래스의 모든 메소드를 조인트포인트라고 한다.

### 5.2.2 포인트컷(pointcut)
- 필터링된 조인포인트
- 수많은 메소드 중에서 우리가 원하는 특정 메소드에서만 횡단 관심에 해당하는 공통 기능을 수행시키기 위해서 필요하다.


applicationContext.xml


```xml
<bean id="log" class="com.springbook.biz.common.LogAdvice"></bean>

<aop:config>
  <aop:pointcut id="allpointcut"
    expression = "execution(* com.springbook.biz..*Impl.*(..))"/>
  <aop:pointcut id="getPointcut"
    expression = "execution(* com.springbook.biz..*Impl.get*(..))"/>

  <aop:aspect ref="log">
    <aop:before pointcut-ref="allpointcut" method="printLog"/>
  </aop:aspect>
</aop:config>
```

- 포인트컷은 `<aop:pointcut>`엘리먼트로 선언하고, id속성으로 포인트컷을 식별하기 위한 유일한 문자열을 선언한다.
- expression 속성은 값을 어떻게 설정하느냐에 따라 필터링되는 메소드가 달라진다.
- `[*] [com.multicampus.biz..] [*impl].[*(..)]`
- `[리턴타입] [패키지 경로] [클래스명] . [메소드명 및 매개변수]`

### 5.2.3 어드바이스(Advice)
- 횡단 관심에 해당하는 공통 기능의 코드
- 독림된 클래스의 메소드로 작성된다.
- 어드바이스로 구현된 메소드가 언제 동작할지 스프링 설정 파일을 통해서 지정할 수 있다.
- before, after, after-returning, after-throwing, around -> 동작시점
```xml
<aop:aspect ref="log">
  <aop:after pointcut-ref="getPointcut" method="printLog"/>
</aop:aspect>
```


### 5.2.4 위빙(Weaving)
- 포인트컷으로 지정한 핵심 관심 메소드가 호출될 때,어드바이스에 해당하는 횡단 관심 메소드가 삽입되는 과정
- 위빙을 통해서 비즈니스 메소드를 수정하지 않고도 횡단 관심에 해당하는 기능을 추가하거나 변경할 수 있다.
- 런타임 방식!

### 5.2.5 애스팩트 또는 어드바이저
- AOP(Aspect Oriented Programming)의 핵심은 애스팩트이다.
- 포인트 컷과 어드바이스의 결합
- 어떤 포인트컷 메소드에 대해서 어떤 어드바이스 메소드를 실행할지 결정한다.


LogAdvice.java
```java
public class LogAdvice{
  public void printLog(){
    System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
  }
}
```

applicationContext.xml
```xml
<bean id="log" class="com.springbook.biz.common.LogAdvice"></bean>

<aop:config>
  <aop:pointcut id="allPointcut" expression = "execution(com.springbook.biz..*Impl.*(..)"/>
  <aop:pointcut id="getPointcut" expression = "execution(com.springbook.biz..*Impl.get*(..)"/>
  <aop:aspect ref="log">
    <aop:before pointcut-ref="getPointcut" method="pringLog"/>
  </aop:aspect>
</aop:config>
```
1. getPointcut으로 설정한 포인트컷 메소드가 호출될 떄
2. log라는 어드바이스 객체의
3. printLog() 메소드가 실행되고
4. 이때 printLog()메소드 동작시점이
5. `<aop:before>`라는 설정





- 트랜잭션 설정에서는 `<aop:advisor>`를 사용한다.

### 5.2.6 AOP용어 종합
1. 사용자가 조인트 포인트를 호출
2. 특정 포인트컷으로 지정한 메소드가 호출 되는 순간
3. 어드바이스객체의 어드바이스 메소드가 실행된다.
4. 어드바이스 메소드를 삽입하도록 하는 설정을 애스팩트라고 하는데 이 애스팩트의 설정에 따라
5. 위빙이 처리된다.

## 5.3 AOP 엘리먼트
### 5.3.1 `<aop:config> 엘리먼트`
- 루트 엘리먼트이다.
- 여러번 사용할 수 있다.
- 하위에는 `<aop:config>`와 `<aop:aspect>` 엘리먼트가 위치할 수 있다.

### 5.3.2 `<aop:pointcut> 엘리먼트`
- 포인트 컷을 지정하기 위해 사용한다.
- `<aop:config>`와 `<aop:aspect>`의 자식 엘리먼트로 사용할 수 있다.
- 그러나 `<aop:aspect>`하위에 설정된 포인트컷은 해당 `<aop:aspect>`에서만 사용할 수 있다.
- 여러개 정의할 수 있다.
- 유일한 아이디를 할당하여 에스팩트를 설정할 때 포인트컷을 참조하는 용도로 사용한다.

applicationContext.xml


```xml
<aop:config>
  <aop:pointcut id="allPointcut" expression = "execution(com.springbook.biz..*Impl.*(..)"/>

  <aop:aspect ref="log">
    <aop:before pointcut-ref="allPointcut" method="pringLog"/>
  </aop:aspect>
</aop:config>
```
- allPointcut이라는 포인트컷은 com.springbook.biz 패키지로 시작하는 클래스 중에서 이름이 Impl로 끝나는 클래스의 모든 메소드를 포인트컷으로 설정하고 있다.
- 애스팩트 설정에서 `<aop:before>`엘리먼트의 pointcut-ref 속성으로 포인트컷을 참조하고 있다.

### 5.3.3 `<aop:aspect>`엘리먼트
- 핵심 관심에 해당하는 포인트컷 메소드와 횡단 관심에 해당하는 어드바이스 메소드를 결합하기 위해 사용한다.



LogAdvice.java
```java
public class LogAdvice{
  public void printLog(){
    System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
  }
}
```

applicationContext.xml
```xml
<bean id="log" class="com.springbook.biz.common.LogAdvice"></bean>

<aop:config>
  <aop:pointcut id="allPointcut" expression = "execution(com.springbook.biz..*Impl.*(..)"/>
  <aop:pointcut id="getPointcut" expression = "execution(com.springbook.biz..*Impl.get*(..)"/>
  <aop:aspect ref="log">
    <aop:before pointcut-ref="getPointcut" method="pringLog"/>
  </aop:aspect>
</aop:config>
```

### 5.3.4 `<aop:advisor>`엘리먼트
- 포인트컷과 어드바이스를 결합한다.
- 하지만 몇몇 특수한 경우에서만 사용해야한다.
- AOP 설정에서 애스팩트를 사용하려면 어드바이스의 아이디와 메소드 이름을 알아야한다.
- 생성된 어드바이스의 아이디는 확인되지만 메소드 이름은 확인할 방법이 없을때 사용한다.

## 5.4 포인트컷 표현식
- execution 명시자를 이용한다.
- `execution(* com.multicampus.biz..*Impl.get*(..))`
- `리턴타입 패키지경로 클래스명 메소드명 매개변수`


- 리턴타입 지정
  - `*` : 모든 리턴타입 허용
  - void : 리턴타입이 void인 메소드 선택
  - !void : 리턴타입이 void가 아닌 메소드 선택


- 패키지 지정
  - `com.springbook.biz` : 정확하게 com.springbook.biz 패키지만 선택
  - `com.springbook.biz..` : com.springbook.biz 패키지로 시작하는 모든 패키지 선택
  - `com.springbook..impl` : com.springbook 패키지로 시작하면서 마지막 패키지 이름이 impl로 끝나는 패키지 선택


- 클래스 지정
  - `BoardServiceImpl` : 정확하게 BoardServiceImpl 클래스만 선택
  - `*Impl` : 클래스 이름이 Impl로 끝나는 클래스만 선택
  - `BoardService+`:클래스 이름 뒤에 '+' 가 붙으면 해당 클래스로부터 파생된 모든 자식클래스를 선택. 인터페이스 뒤에 '+'가 붙으면 해당 인터페이스를 구현한 모든 클래스 선택


- 메소드 지정
  - `*(..)` : 가장 기본 설정으로 모든 메소드 선택
  - `get*(..)` : 메소드 이름이 get으로 시작하는 모든 메소드 선택

  
- 매개변수 지정
  - `(..)` : 매개변수의 개수와 타입에 제약이 없다
  - `(*)` : 반드시 1개의 매개변수를 가지는 메소드만 선택
  - `(com.springbook.user.UserVO)`: 매개변수로 UserVO를 가지는 메소드만 선택, 이때 클래스의 패키지 경로가 반드시 포함되어야 함
  - `(!com.springbook.user.UserVO)` : 매개변수로 UserVO를 가지지 않는 메소드만 선택
  - `(Integer,..)` : 한개 이상의 매개변수를 가지되, 첫번째 매개변수 타입이 Integer인 메소드만 선택
  - `(Integer,*)` : 반드시 두개의 매개변수를 가지되, 첫 번째 매개변수의 타입이 Integer인 메소드만 선택
