제 6장
## 6.1 어드바이스 동작
- Before : 비즈니스 메소드 실행 전 동작
- After
  - After Returning : 비즈니스 메소드가 성공적으로 리턴되면 동작
  - After Throwing : 비즈니스 메소드 실행 중 예외가 발생하면 동작(catch)
  - After: 비즈니스 메소드가 실행된 후, 무조건 실행(finally)
- Around : 메소드 호출 자체를 가로채 비즈니스 메소드 실행 전후에 처리할 로직을 삽입할 수 있음

### 6.1.1 Before 어드바이스
- 포인트컷으로 지정된 메소드 호출 시, 메소드가 실행되기 전에 처리될 내용들을 기술
- `<aop:before>`엘리먼트를 사용한다.

```java
package com.springbook.biz.common;

public class BeforeAdvice{
  public void beforeLog(){
    System.out.println("[사진 처리] 비즈니스 로직 수행 전 동작");
    }
}
```
applicationContext.xml
```xml
<!-- Before -->
<bean id="before" class="com.springbook.biz.common.BeforeAdvice"/>

<aop:config>
  <aop:pointcut id="allpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="before">
      <aop:before pointcut-ref="allPointcut" method="beforeLog"/>
    </aop:aspect>
</aop:config>
```

### 6.1.2 After Returning 어드바이스
- 포인트컷으로 지정된 메소드가 정상적으로 실행되고 나서, 메소드 수행 결과로 생성된 데이터를 리턴하는 시점에 동작한다.
- 비즈니스 메소드 수행 결과로 얻은 결과 데이터를 이용하여 사후 처리 로직을 추가할 때 사용한다.
- `<aop:after-returning>`엘리먼트를 사용한다.


```java
package com.springbook.biz.common;

public class AfterReturningAdvice{
  public void AfterLog(){
    System.out.println("[사후 처리] 비즈니스 로직 수행 후 동작");
    }
}
```
applicationContext.xml
```xml
<!-- After Returning -->
<bean id="afterReturning" class="com.springbook.biz.common.AfterReturningAdvice"/>

<aop:config>
  <aop:pointcut id="getpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="afterReturning">
      <aop:after-returning pointcut-ref="getPointcut"  method="afterLog"/>
    </aop:aspect>
</aop:config>
```

### 6.1.3 After Throwing 어드바이스
- 포인트컷으로 지정한 메소드가 실행되다가 예외가 발생하는 시점에 동작한다.
- 예외 처리 어드바이스를 설정할 때 사용한다.

```java
package com.springbook.biz.common;

public class AfterThrowingAdvice{
  public void exceptionLog(){
    System.out.println("[예외 처리] 비즈니스 로직 수행 중 예외 발생");
    }
}
```
BoardServiceImpl.java
```java
@Service("boardService")
public class BoardServiceImpl impliments BoardService {
  @Autowired
  private BoardDAO boardDAO;

  public void insertBoard(BoardVO vo){
    if(vo.getSeq() == 0){
      thriw new IllegalArgumentException("0번 글은 등록할 수 없습니다.");
    }
    boardDAO.insertBoard(vo);
  }
}
```

applicationContext.xml
```xml
<!-- After  Throwing -->
<bean id="afterThrowing" class="com.springbook.biz.common.AfterThrowingAdvice"/>

<aop:config>
  <aop:pointcut id="allpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="afterThrowing">
      <aop:after-throwing pointcut-ref="allPointcut"  method="exceptionLog"/>
    </aop:aspect>
</aop:config>
```

### 6.1.4 After 어드바이스
- try-catch-finally구문에서 finally 블록처럼 무조건 수행되는 어드바이스
- finalltLog() 메소드 실행 후 exceptionLog()메소드가 실행된다.

```java
package com.springbook.biz.common;

public class AfterAdvice{
  public void finallyLog(){
    System.out.println("[사후 처리] 비즈니스 로직 수행 후 무조건 동작");
    }
}
```
applicationContext.xml
```xml
<!-- After -->
<bean id="afterThrowing" class="com.springbook.biz.common.AfterThrowingAdvice"/>
<bean id="after" class="com.springbook.biz.common.AfterAdvice"/>

<aop:config>
  <aop:pointcut id="allpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="afterThrowing">
      <aop:after-throwing pointcut-ref="allPointcut"  method="exceptionLog"/>
    </aop:aspect>

    <aop:aspect ref="after">
      <aop:after pointcut-ref = "allPointcut" method="finallyLog"/>
    </aop:aspect>
</aop:config>
```

### 6.1.5 Around 어드바이스
- 비즈니스 메소드 실행 전과 후에 모두 동작한다.
- 클라이언트의 메소드 호출을 가로챈다.
- 클라이언트가 호출한 비즈니스 메소드가 실행되기 전에 사전 처리 로직을 수행할 수 있으며, 비즈니스 메소드가 모두 실행되고 나서 사후 처리 로직을 수행할 수 있다.

```java
package com.springbook.biz.common;

import org.aspectj.lang.ProceedingJoinPoint;

public class AroundAdvice{
  public Object aroundLog(ProceedingJoinPoint pjp) throws Throwable{
    System.out.println("[BEFORE]: 비즈니스 메소드 수행 전에 처리할 내용...");
    Object returnObj = pjp.proceed();
    System.out.println("[AFTER]:비즈니스 메소드 수행 후에 처리할 내용");
    return returnObj;
    }
}
```
applicationContext.xml
```xml
<!-- Around -->
<bean id="afterReturning" class="com.springbook.biz.common.AroundAdvice"/>

<aop:config>
  <aop:pointcut id="allpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="around">
      <aop:around pointcut-ref="allPointcut"  method="aroundLog"/>
    </aop:aspect>
</aop:config>
```

## 6.2 JoinPoint와 바인드 변수
- 횡단 관심에 해당하는 어드바이스 메소드를 의미 있게 구현하려면 클라이언트가 호출한 비즈니스 메소드의 정보가 필요하다.
- 다양한 정보들을 JoinPoint인터페이스를 제공한다.

### 6.2.1 JoinPoint 메소드
- `Signature getSignature()` : 클라이언트가 호출한 메소드의 시그니처(리턴타입,이름,매개변수) 정보가 저장된 Signature 객체 리턴
- `Object getTarget()` : 클라이언트가 호출한 비즈니스 메소드를 포함하는 비즈니스 객체 리턴
- `Object[] getArgs()` : 클라이언트가 메소드 호출할 때 넘겨준 인자 목록을 Object 배열로 리턴
- Around 어드바이스에서는 ProceedingJoinPoint를 매개변수로 사용해야한다.
- Signature 제공 메소드
  - `String getName()` : 클라이언트가 호출한 메소드 이름 리턴
  - `String toLongString()` : 클라이언트가 호출한 메소드의 리턴 타입, 이름, 매개변수를 패키지 경로까지 포함하여 리턴
  - `String toShortString()` : 클라이언트가 호출한 메소드 시그니쳐를 축약한 문자열로 리턴



```java
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;

public class LogAdvice{
  public void printLog(JoinPoint jp){

    System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
  }
}
```

### 6.2.2 Before 어드바이스
- `getSignature()`메소드로 클라이언트가 호출한 메소드 이름을 출력한다.
- `getArgs()`메소드를 통해 인자 목록을 Object배열로 얻어낼 수 있어,메소드에 어떤 호출값들을 사용했는지 알 수있다.

```java
package com.springbook.biz.common;
import org.aspectj.lang.JoinPoint;

public class BeforeAdvice{
  public void beforeLog(JoinPoint jp){
    String method = jp.getSignature().getName();
    Object[] args = jp.getArgs();
    System.out.println("[사진 처리]" + method + "()메소드 ARGS 정보: " + args[0].toString());
    }
}
```

### 6.2.3 After Returning 어드바이스
- `afterLog()`메소드는 클라이언트가 호출한 비즈니스 메소드 정보를 알아내기 위해 JoinPoint객체를 첫 번째 매째 매개변수로 선언되어 있는데, 이를 '바인드 변수'라고 한다.
- 바인드 변수가 추가됐다면 반드시 바인드 변수에 대한 매핑 설정을 스프링 설정 파일에 `aop:after-returning`앨리먼트의 returning속성을 사용해서 추가해야한다.

```java
package com.springbook.biz.common;
import org.aspectj.lang.JoinPoint;
import com.springbook.biz.user.UserVO;

public class AfterReturningAdvice{
  public void AfterLog(JoinPoint jp,Object returnObj){
    String method = jp.getSignature().getName();
    if(returnObj instanceof UserVO){
      UserVO user = (UserVO) returnObj;
      if(user.getRole().equals("Admin")){
        System.out.println(user.getName()+"로그인(Admin)");
      }
    }
    System.out.println("[사후 처리]" + method+ " () 메소드 리턴값:" + returnObj.toString());
    }
}
```
applicationContext.xml
```xml
<!-- After Returning -->
<bean id="afterReturning" class="com.springbook.biz.common.AfterReturningAdvice"/>

<aop:config>
  <aop:pointcut id="getpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="afterReturning">
      <aop:after-returning pointcut-ref="getPointcut"  method="afterLog" returning="returnObj"/>
    </aop:aspect>
</aop:config>
```

### 6.2.4  After Throwing 어드바이스
- `<aop:after-throwing>`앨리먼트의 throwing속성을 사용한다.
- 비즈니스 메소드에서 발생한 예외 객체를 exceptObj라는 바인드 변수에 바인드 하라는 설정
- 예외 객체의 종류에 따라 다양하게 예외 처리를 할 수도 있다.


```java
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;

public class AfterThrowingAdvice{
  public void exceptionLog(JointPoint jp, Exception exceptObj){
    String method = jp.getSignature().getName();

    System.out.println("[예외 처리]" + method " ()메소드 수행 중 발생된 예외 메세지 :" + exceptObj.getMassage());
    }
}
```
BoardServiceImpl.java
```java
@Service("boardService")
public class BoardServiceImpl impliments BoardService {
  @Autowired
  private BoardDAO boardDAO;

  public void insertBoard(BoardVO vo){
    if(vo.getSeq() == 0){
      thriw new IllegalArgumentException("0번 글은 등록할 수 없습니다.");
    }
    boardDAO.insertBoard(vo);
  }
}
```

applicationContext.xml
```xml
<!-- After  Throwing -->
<bean id="afterThrowing" class="com.springbook.biz.common.AfterThrowingAdvice"/>

<aop:config>
  <aop:pointcut id="allpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="afterThrowing">
      <aop:after-throwing pointcut-ref="allPointcut"  method="exceptionLog" throwing="exceptObj"/>
    </aop:aspect>
</aop:config>
```


### 6.2.5 Around 어드바이스
- 다른 어드바이스와 다르게 반드시 ProceedingJoinPoint 객체를 매개변수로 받아야한다.

```java
package com.springbook.biz.common;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;

public class AroundAdvice{
  public Object aroundLog(ProceedingJoinPoint pjp) throws Throwable{
    String method = pjp.getSignature().getName();

    StopWatch stopwatch = new StopWatch();
    stopwatch.start();

    Object obj = pjp.proceed();

    stopwatch.stop();
    System.out.println(method + "() 메소드 수행에 걸린 시간: " + StopWatch.getTotalTimeMillis()+ "(ms)초");
    return obj;
    }
}
```

## 6.3 어노테이션 기반 AOP
### 6.3.1 어노테이션 사용을 위한 스프링 설정
- `aop:aspectj-autoproxy`만 선언하면 스프링 컨테이너는 AOP관련 어노테이션들을 인식하고 용도에 맞게 처리해준다.
- 어드바이스 클래스에 AOP관련 어노테이션들을 설정한다.
- 어드바이스 클래스는 반드시 스프링 설정 파일에 <bean>등록하거나 @Service어노테이션을 사용하여 컴포넌트가 검색될 수 있도록 해야한다.
- Annotation설정
```java
@service
public class LogAdvice{}
```
- XML 설정
```xml
<bean id="log" class="com.springbook.biz.common.LogAdvice"></bean>
```


applicationContext.xml
```xml
<context:component-scan base-package="com.springbook.biz"></context:component-scan>

<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

### 6.3.2 포인트컷 설정
- `<aop:config>`엘리먼트를 사용한다.
- 어노테이션 설정으로 선언할 때는 `@Pointcut`를 사용한다.
- 하나의 어드바이스 클래스 안에 참조 메소드를 이용해, 여러 개의 포인트컷을 선언할 수 있다.
- 참조 메소드는 구현 로직이 없는 메소드이므로 단순히 포인트컷을 식별하는 이름으로만 사용된다.
applicationContext.xml
```xml
<aop:config>
  <aop:pointcut id="allPointcut" expression="execution(* com.springbook.biz..*Impl.*(..))"/>
  <aop:pointcut id="getPointcut" expression="execution(* com.springbook.biz..*Impl.get*(..))"/>

  <aop:aspect ref="log">
    <aop:before pointcut-ref="allPointcut" method="printLog"/>
  </aop:aspect>
</aop:config>
```

LogAdvice.java
```java
import org.aspectj.lang.annotation.Pointcut;

@service
public class LogAdvice{
  @Pointcut("execution(* com.springbook.biz..*Impl.*(..))")
  public void allPointcut(){}

  @Pointcut("executionexecution(* com.springbook.biz..*Impl.get*(..))")
  public void getPointcut(){}
}
```

### 6.3.3 어드바이스 설정
- 어드바이스 메소드가 언제 동작할지 결정하여 관련된 어노테이션을 메소드 위에 설정하면 된다.
- 어노테이션 뒤에 괄호를 추가하고 포인트컷 참조 메소드를 지정하면 된다.
LogAdvice.java


```java
import org.aspectj.annotation.Before;
import org.aspectj.annotation.Pointcut;

@Service
public class LogAdvice{
  @Pointcut("execution(* com.springbook.biz.Impl.*(..))")
  public void allPointcut(){}

  @Before("allPointcut()")
  public void printLog(){
    System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
  }
}
```

### 6.3.4 애스팩트 설정
- `@Aspect`이용하여 설정한다.
- 애스팩트 : 포인트컷+어드바이스
- LogAdvice 클래스 위에 @Aspect가 설정되어있으므로 스프링 컨테이너는 LogAdvice 객체를 애스팩트 객체로 인식한다.



LogAdvice.java

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

@Service
@Aspect //Aspect = Pointcut + Advice
public class LogAdvice{
  @Pointcut("execution(* com.springbook.biz..*Impl.*(..))")
  public void allPointcut(){}

@Before("allPointcut()")
public void printLog(){
  System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
 }
}
```
## 6.4 어드바이스 동작 시점
### 6.4.1 Before 어드바이스
- 클래스 선언부에 `@Service` `@Aspect`를 추가하여 BeforeAdvice 클래스가 컴포넌트 스캔되어 애스팩트 객체로 인식되도록 했다.
- allPointcut() 참조 메소드를 추가하여 포인트컷을 선언했다.
- beforeLog() 메소드 위에 `@Before`를 추가하여 allPointcut()으로 지정한 메소드가 호출될 때, beforeLog() 메소드가 Before 형태로 동작하도록 설정했다.



```java
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;

@service
@Aspect
public class BeforeAdvice{
  @Pointcut("execution(* com.springbook.biz..*Impl.*(..))")
  public void allPointcut(){}

  @Before("allPointcut()")
  public void beforeLog(JoinPoint jp){
    String method = jp.getSignature().getName();
    Object[] args = jp.getArgs();

    System.out.println("[사전 처리]"+method+"()메소드 ARGS정보: "+ args[0].toString());
  }
}
```

### 6.4.2 After Returning 어드바이스
- 비즈니스 메소드가 리턴한 결과 데이터를 다른 용도로 처리할 때 사용
- AfterReturningAd 클래스에 관련된 어노테이션을 추가한다.
- afterLog() 메소드가 After Returning 형태로 동작하도록 메소드 위에 `@AfterReturning` 어노테이션을 추가한다.
- After Returning 어드바이스가 비즈니스 메소드 수행 결과를 받아내기 위해 바인드 변수를 지정해야 한다. -> 포인트컷을 참조하고 있다.

```java
package com.springbook.biz.common;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;

@service
@Aspect
public class AfterReturningAdvice{
  @Pointcut("execution(* com.springbook.biz..*Impl.get*(..))")
  public void getPointcut(){}

  @AfterReturning(pointcut="getPointcut()", returning="returnObj")
  public void AfterLog(JoinPoint jp,Object returnObj){
    String method = jp.getSignature().getName();

    if(returnObj instanceof UserVO){
      UserVO user = (UserVO) returnObj;
      if(user.getRole().equals("Admin")){
        System.out.println(user.getName()+"로그인(Admin)");
      }
    }
    System.out.println("[사후 처리]" + method+ " () 메소드 리턴값:" + returnObj.toString());
    }
}
```

applicationContext.xml
```xml
<aop:aspect ref="AfterReturning">
  <aop:after-returning pointcut-ref="getPointcut" method="afterLog" returning="returnObj"/>
</aop:aspect>
```

### 6.4.3 After Throwing 어드바이스
- 메소드 실행 도중 예외가 발생했을 때 공통적인 예외처리 로직을 제공할 목적으로 사용한다.
- AfterThrowingAdvice 클래스에 어노테이션을 추가한다.
- 바인드 변수를 지정해야 하기 때문에 포인트컷 사용
- throwing 속성을 이용하여 바인드 변수 지정

```java
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;

@Service
@Aspect
public class AfterThrowingAdvice{
  @Pointcut("exec(* com.springbo.biz..*Impl.*(..))")
  public void allPointcut(){}

  @AfterThrowing(pointcut="allPointcut()", throwing="exceptObj")
  public void exceptionLog(JointPoint jp, Exception exceptObj){
    String method = jp.getSignature().getName();
    System.out.println("[예외 처리]" + method " ()메소드 수행 중 발생된 예외 메세지 :" + exceptObj.getMassage());
    }
}
```

applicationContext.xml
```xml
<!-- After  Throwing -->
<bean id="afterThrowing" class="com.springbook.biz.common.AfterThrowingAdvice"/>

<aop:config>
  <aop:pointcut id="allpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="afterThrowing">
      <aop:after-throwing pointcut-ref="allPointcut"  method="exceptioLog" throwing="exceptObj"/>
    </aop:aspect>
</aop:config>
```

### 6.4.4 After 어드바이스
- 예외 발생 여부에 상관없이 무조건 수행
- `@After` 어노테이션을 사용하여 설정

```java
package com.springbook.biz.common;


import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;

@Service
@Aspect


public class AfterAdvice{
  @Pointcut("execution(* com.springbook.biz..*Impl.*(..))")
  public void allPointcut(){}

  @After("allPointcut()")
  public void finallyLog(){
    System.out.println("[사후 처리] 비즈니스 로직 수행 후 무조건 동작");
    }
}
```

### 6.4.5 Around 어드바이스 설정
- 하나의 어드바이스로 사전, 사후 처리를 모두 해결하고자 할때 사용
- `@Around` 어노테이션 추가
- 바인드 변수가 없으므로 포인트컷 메소드만 참조
- `ProceedJoinPoint`객체를 매개변수로 받는다.
- 바인드 변수가 없으므로 포인트컷 메소드만 참조하면 된다.

```java
package com.springbook.biz.common;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;
import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;

@Service
@Aspect

public class AroundAdvice{
  @Pointcut("execution(* com.springbook.biz..*Impl.*(..))")
  public void allPointcut(){}

  @Around("allPointcut()")
  public Object aroundLog(ProceedingJoinPoint pjp) throws Throwable{
    String method = pjp.getSignature().getName();

    StopWatch stopwatch = new StopWatch();
    stopwatch.start();

    Object obj = pjp.proceed();

    stopwatch.stop();
    System.out.println(method + "() 메소드 수행에 걸린 시간: " + StopWatch.getTotalTimeMillis()+ "(ms)초");
    return obj;
    }
}
```

### 6.4.6 외부 Pointcut 참조하기
- 어노테이션 설정은 어드바이스 클래스 마다 포인트컷 설정이 포함되면서, 비슷하거나 같은 포인트컷이 반복 선언되는 문제 발생
- 이를 해결하고자 외부에 독립된 클래스에 따로 설정한다.
- 시스템에서 사용할 모든 포인트컷을 PointcutCommon 클래스에 등록한다.

PointcutCommon.java
```java
package com.sprinbook.biz.common;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@aspectpublic class PointcutCommon {
  @Pointcut("execution(* com.springbook.biz..*Impl.*(..))")
  public void allPointcut(){}

  @Pointcut("execution(* com.springbook.biz..*Impl.get*(..))")
  public void getPointcut(){}
}
```

- 정의된 포인트컷을 참조하려면 클래스 이름과 참조 메소드 이름을 조합하여 지정해야 한다.


```java
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;

@service
@Aspect
public class BeforeAdvice{

  @Before("PointcutCommon.allPointcut()")
  public void beforeLog(JoinPoint jp){
    String method = jp.getSignature().getName();
    Object[] args = jp.getArgs();

    System.out.println("[사전 처리]"+method+"()메소드 ARGS정보: "+ args[0].toString());
  }
}
```


```java
package com.springbook.biz.common;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;

@service
@Aspect
public class AfterReturningAdvice{

  @AfterReturning(pointcut="PointcutCommon.getPointcut()", returning="returnObj")
  public void AfterLog(JoinPoint jp,Object returnObj){
    String method = jp.getSignature().getName();

    if(returnObj instanceof UserVO){
      UserVO user = (UserVO) returnObj;
      if(user.getRole().equals("Admin")){
        System.out.println(user.getName()+"로그인(Admin)");
      }
    }
    System.out.println("[사후 처리]" + method+ " () 메소드 리턴값:" + returnObj.toString());
    }
}
```
