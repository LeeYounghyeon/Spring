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

<!-- After Returning -->
<bean id="afterReturning" class="com.springbook.biz.common.AfterReturningAdvice"/>

<aop:config>
  <aop:pointcut id="getpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="afterReturning">
      <aop:after-returning pointcut-ref="getPointcut"  method="afterLog"/>
    </aop:aspect>
</aop:config>

<!-- After  Throwing -->
<bean id="afterThrowing" class="com.springbook.biz.common.AfterThrowingAdvice"/>

<aop:config>
  <aop:pointcut id="allpointcut"
    expression="execution(* com.springbook.biz..*Impl.*(..))"/>

    <aop:aspect ref="afterThrowing">
      <aop:after-throwing pointcut-ref="allPointcut"  method="exceptionLog"/>
    </aop:aspect>
</aop:config>

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
