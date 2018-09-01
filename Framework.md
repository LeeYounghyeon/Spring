**내가 이해하기 위해!! 참고자료: 스프링 퀵 스타트**

##1.1 프레임워크 개념
###1.1.1프레임워크의 등장 배경
* 프레임워크의 사전적 의미 
	- 뼈대, 틀
    
* 소프트웨어 관점
	- 아키텍처에 해당하는 골격코드

###1.1.2 프레임워크의 장점
* 프레임워크를 사용하면 애플리케이션에 대한 분석, 설계, 구현 모두에서 ==재사용성 이 증가==한다.
1. 빠른 구현 시간
	- 프레임워크를 사용하면 아키텍처에 ==해당하는 골격 코드를 프레임워크에서 제공==
    - 개발자는== 비즈니스 로직==만 구현한다.
    - 제한된 시간에 많은 기능을 구현할 수 있다.
    
2. 쉬운 관리
	- 같은 프레임워크가 적용된 애플리케이션은== 아키텍쳐가 같으므로 관리하기가 쉽다==.
    - 유지보수에 들어가는 ==인력과 시간==도 줄일 수 있다.

3. 개발자들의 역량 획일화
	- 경험이 적은 개발자도, 경험이 많은 개발자도 생성한 코드가 비슷해진다.
    - 효율적으로 구성이 가능하다.
    
4. 검증된 아키텍쳐의 재사용과 일관성 유지
	- 프레임워크에서 제공하는 아키텍쳐를 이용하므로 ==별다른 검증 없이== 소프트웨어 개발이 가능하다.
    - 왜곡이나 변형이 없다.

###1.1.3 자바 기반의 프레임워크
-  자바 기반의 프레임워크로 ==오픈소스 형태==로 제공한다.


##1.2 스프링 프레임워크
###1.2.1 스프링 탄생 배경
- 로드 존슨이 2004년에 만든 오픈소스 프레임워크
- POJO(Plain Old Java Object)를 사용하면서도 EJB일도 가능하도록 지원한다.

###1.2.2 스프링 프레임워크의 특징
- IoC와 AOP를 지원하는 경량의 컨테이너 프레임워크
1. 경량(Lightweight)
	- 여러 개의 모듈로 구성되어 있고, 하나 이상의 JAR파일로 구성되어있다.
	- 가볍고 빠르다.
	- POJO형태의 객체를 관리한다.
	
2. 제어의 역행(Inversion of Control)
	- 낮은 결합도를 유지한다.
	- 객체를 변경할 때 자바코드를 수정하지 않고, ==컨테이너==가 대신 처리함으로 의존 관계가 떨어져 유지보수가 편리하다.

3. 관점지향 프로그래밍(Aspect Oriented Programming, AOP)
	- 비즈니스 메소드를 개발할 때, 핵심 비즈니스 로직과 각 비즈니스 메소드마다 반복해서 등장하는 ==공통 로직을 분리==함으로써 응집도가 높게 개발 지원
	- 공통기능 -> 외부의 독립된 클래스로 분리/ 선언적으로 처리

4. 컨테이너
	- 특정 객체의 생성과 관리 담당
	- 객체 운용에 필요한 다양한 기능 제공
	- 서버안에 포함되어 배포 및 구동
	- 스프링도 일종의 컨테이너다.

##1.3 IoC컨테이너 
### 1.3.1 구동 순서

```java
public class HelloServlet extends HttpSerblet{
	public HelloServlet(){
    	System.out.println("===> HelloServlet 객체 생성");
    }
    protected void doGet(HttpServletRequest request,
    HttpServletResponse response) throws ServletException, IOException{
    	System.out.println("doGet() 메소드 호출");
    }
}
```

- Servlet클래스는 web.xml에 자동으로 등록된다.

```xml
<web-app>
	<servlet>
    	<servlet-name>hello</servlet>
        <servlet-class>hello.HelloServlet</servlet-class> <!-- hello.HelloServlet클래스를 찾아 객체를 생성 후 실행 -->
    </servlet>
    <servlet-mapping>
    	<servlet-name>hello</servlet-name>
        <url-pattern>/hello.do</url-pattern> <!--hello.do url요청 -->
    </servlet-mapping>
</web-app>
```

실행결과
`===> HelloServlet 객체 생성`
`doGet() 메소드 호출`

1. WEB-INF/web.xml 파일을 로딩하여 구동
2. 브라우저로부터 /hello.do 요청 수신
3. hello.HelloServlet 클래스를 찾아 객체를 생성하고 doGet()메소드 호출
4. doGet() 메소드 실행 결과를 클라이언트 브라우저로 전송

- 컨테이너는 자신이 관리할 클래스들이 등록된 XML 설정 파일은 로딩하여 구동
- 클라이언트의 요청이 들어오는 순간 XML 설정파일을 참조 //객체 생성, 객체 생명주기 관리
- 개발자가 어떤 객체를 생성할지 판단하고 객체 간의 의존관계도를 소스코드로 표현
- 제어의 역행은 소스코드가 아닌 ==컨테이너==로 처리

###1.3.2 결합도가 높은 프로그램

SamsungTV.java
```java
package polymorphism;

public class SamsungTV{
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

LgTV.java
```java
package polymorphism;

public class LgTV{
	public void turnOn(){
    	System.out.println("LgTV ---전원 켠다.");
    }
    
    public void turnOff(){
    	System.out.println("LgTV ---전원 끈다.");
    }
    
    public void soundUp(){
    	System.out.println("LgTV ---소리 올린다.");
    }
    
    public void soundDown(){
    	System.our.println("LgTV ---소리 내린다.");
    }
}
```
두개의 클래스는 같은 기능을 수행하는 메소드가 있지만 메소드 이름이 다르다.

TVUser.java
```java
package polymorphism;

public class TVUser{
	public static void main(String[] args){
    	SamsungTV tv = new SamsungTV();
        tv.powerOn();
        tv.volumeUp();
        tv.volumeDown();
        tv.powerOff();
    	}
	}
}
```
유저가 자바 코드 대부분을 수정해야한다. 그러므로 TV 교체를 결정하기가 쉽지 않을 것이다.

###2.3.2 다형성 이용하기
결합도를 낮추기 위해서 다형성을 이용한다. 
TV 클래스들의 최상위 부모로 사용할 TV인터페이스를 추가하고 공통으로 가져야 할 메소드를 추상 메소드로 선언한다.

TV.java
```java
package polymorphism;

public interface TV{
	public void powerOn();
    public void powerOff();
    public void volumeUp();
    public void volumeDown();
}
```
TV 인터페이스들을 구현하도록 수정한다.

SamsungTV.java
```java
package polymorphism;

public class SamsungTV implements TV{
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

LgTV.java
```java
package polymorphism;

public class LgTV implements TV{
	public void powerOn(){
    	System.out.println("LgTV ---전원 켠다.");
    }
    
    public void powerOff(){
    	System.out.println("LgTV ---전원 끈다.");
    }
    
    public void volumeUp(){
    	System.out.println("LgTV ---소리 올린다.");
    }
    
    public void volumeDown(){
    	System.our.println("LgTV ---소리 내린다.");
    }
}
```

TVUser.java
```java
package polymorphism;

public class TVUser{
	public static void main(String[] args){
    	SamsungTV tv = new SamsungTV();
        tv.powerOn();
        tv.volumeUp();
        tv.volumeDown();
        tv.powerOff();
    	}
	}
}
```

이제 참조하는 객체만 변경하면 된다. 객체를 쉽게 변경 가능하다.
다형성을 이용하면 최소한의 수정으로 객체를 교체 가능하다.
따라서 유지보수는 좀 더 편해졌다고 볼 수 있다.

###2.3.2 디자인 패턴 이용하기
결합도를 낮추기 위한 또 다른 방법으로 디자인 패턴을 이용하는 것이다.
TV를 교체할 때 클라이언트 소스를 수정하지 않고 TV만 교체할 수 있으면 유지보수는 더 편리해질 것이다.
이를 위헤 ==factory== 패턴을 적용해야한다.
- factory패턴? 클라이언트에서 사용할 객체 생성을 캡슐화하여 TVUser와 TV사이를 느슨한 결합 상태로 만들어준다.

BeanFactory.java
```java
package polymorphism;

public class BeanFactory{
	public Object getBean(String beanName){
    	if(beanName.equals("samsung)){
        	return new SamsungTV():
        }else if(beanName.equals("lg")){
        	return new LgTV();
        }
        return null;
    }
}
```

TVUser.java
```java
package polymorphism;

public class TVUser{
	public static void main(String[] args){
    	BeanFactory factory = new BeanFactory();
    	TV tv = (TV)factory.getBean(args[0]);
        tv.powerOn();
        tv.volumeUp();
        tv.volumeDown();
        tv.powerOff();
    	}
	}
}
```

프로그램을 실행하고 Lg 나 Samsung을 넘겨주면 된다.
	Run As - Run Configurations - Arguments탭 선택 - 입력 순이다.
유저는 더욱 간단하게 TV객체를 변경할 수 있을 것이다.

