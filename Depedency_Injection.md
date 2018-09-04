## 3.1 의존성 관리
### 3.1.1 스프링의 의존성 관리 방법
- 스프링 프레임워크의 가장 중요한 특징은 객체의 생ㅇ성과 의존관계를 컨테이너가 자동으로 관리한다는 점이다.
- Dependency Lookup
  - 컨테이너가 애플리케이션 운용에 필요한 객체를 생성
  - 클라이언트는 컨테이너가 생성한 객체를 검색하여 사용

- Dependency Injection
  - 객체 사이의 의존관계를 스프링 설정 파일에 등록된 정보를 바탕으로 컨테이너가 자동으로 처리해주는 방식
  - 의존성 설정을 바꾸고 싶을 때 프로그램 코드를 수정하지 않고 스프링 설정 파일 수정만으로 변견사항 적용
  - 유지보수가 향상된다.
  - 컨테이너가 직접 객체들 사이에 의존관계를 처리한다.
  - 세터 인젝션과 생성자 인젝션으로 나뉜다.

### 3.1.2 의존성 관계
- 의존성 관계? 객체와 객체 사이의 결합관계

SonySpeaker.java
```java
package polymorphism;

public class SonySpeaker{
  public SonySpeaker(){
    System.out.println("===> SonySpeaker 객체 생성");
  }
  public void volumeUp(){
    System.out.println("SonySpeaker---소리 올린다.");
  }
  public void volumeDown(){
    System.out.println("SonySpeaker---소리 내린다.")
  }
}
```

SamsungTV클래스의 볼륨 조절 기능을 SonySpeaker가 이용하도록 수정한다.

SamsungTV.java
```java
package polymorphism;

public class SamsungTV implements TV{
  private SonySpeaker speaker;

  public SamsungTV(){
    System.out.println("===> SamsungTV 객체 생성");
  }
  public void powerOn(){
    System.out.println("SamsungTV --- 전원 켠다.");
  }
  public void powerOff(){
    System.out.println("SamsungTV --- 전원 끈다.");
  }
  public void volumeUp(){
    speaker = new SonySpeaker();
    speaker.volumeUp();
  }
  public void volumeDown(){
    speaker = new SonySpeaker();
    speaker.volumeDown();
  }
}
```

이 프로그램의 문제
1. SonySpeaker 객체가 쓸데 없이 두개나 생성된다.
2. 운영 과정에서 SonySpeaker의 성능이 떨어져 다른 Speaker로 변경하고자 할 때 메소드를 모두 수정해주어야 한다.

스프링은 이 문제를 `의존성 주입(Dependency Injection)`을 이용하여 해결한다.

## 3.2 생성자 인젝션 이용하기
- 스프링 컨테이너는 XML 설정 파일에 등록된 클래스를 찾아서 객체 생성할 때 기본적으로 매개변수가 없는 기본 생성자를 호출한다.
- 생성자 인젝션을 사용하면 생성자의 매개변수로 의존관계에 있는 객체의 주소 정보를 전달할 수 있다.

SamsungTV.java
```java
package polymorphism;

public class SamsungTV implements TV{
  private SonySpeaker speaker;

  public SamsungTV(){
    System.out.println("===> SamsungTV(1) 객체 생성");
  }
  //생성자 추가
  public SamsungTV(SonySpeaker speaker){
    System.out.println("===> SamsungTV(2) 객체 생성");
    this.speaker = speaker;
  }

  public void powerOn(){
    System.out.println("SamsungTV --- 전원 켠다.");
  }
  public void powerOff(){
    System.out.println("SamsungTV --- 전원 끈다.");
  }
  public void volumeUp(){
    speaker.volumeUp();
  }
  public void volumeDown(){
    speaker.volumeDown();
  }
}
```
추가된 두 번째 생성자는 SonySpeaker객체를 매개변수로 받아서 멤버변수로 선언된 speaker를 초기화한다.

applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.Springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
       xsi:schemaLocation="http://www.Springframework.org/schema/beans/Spring-beans.xsd">

       <bean id="tv" class="polymorphism.SamsungTV">
          <constructor-arg ref="sony"></constructor-arg>
       </bean>

       <bean id="sony" class="polymorphism.SonySpeaker"/>
</beans>
```
- 생성자 인젝션을 위해서 constructor-arg 엘리먼트를 추가한다.
- 생정자 인자로 전달할 객체의 아이디를 constructor-arg 엘리먼트에 ref 속성으로 참조한다.
- SamsungTV 클래스의 객체가 생성될 때, 두번째 생성자가 사용된다.
- 스프링 설정 파일에 SonySpeaker가 SamsungTV 클래스 및에 등록되었는데도 먼저 생성되고 있다.
- 스프링 컨테이너는 기본적으로 등록된 순서대로 객체를 생성한다.
- 모든 객체는 기본 생성자 호출을 원칙으로 한다.
- 생성자 인젝션으로 의존성이 주입될 경우 변할 수 있다.

### 3.2.1 다중 변수 매핑
생성자 인젝션에서 초기화해야 할 멤버변수가 여러 개이면, 여러 개의 값을 한꺼번에 전달해야 한다.

SamsungTV.java
```java
package polymorphism;

public class SamsungTV implements TV{
  private SonySpeaker speaker;
  //추가 된 부분
  private int price;

  public SamsungTV(){
    System.out.println("===> SamsungTV(1) 객체 생성");
  }

  public SamsungTV(SonySpeaker speaker){
    System.out.println("===> SamsungTV(2) 객체 생성");
    this.speaker = speaker;
  }

//추가 된 부분
  public SamsungTV(SonySpeaker speaker,int price){
    System.out.println("===>SamsungTV(3) 객체 생성");
    this.price = price;
  }

  public void powerOn(){
    System.out.println("SamsungTV --- 전원 켠다.(가격:"+price")");
  }
  public void powerOff(){
    System.out.println("SamsungTV --- 전원 끈다.");
  }
  public void volumeUp(){
    speaker.volumeUp();
  }
  public void volumeDown(){
    speaker.volumeDown();
  }
}
```

스프링 설정 파일에 `constructor-arg` 엘리먼트를 매개변수의 개수만큼 추가해야한다.
applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.Springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
       xsi:schemaLocation="http://www.Springframework.org/schema/beans/Spring-beans.xsd">

       <bean id="tv" class="polymorphism.SamsungTV">
          <constructor-arg ref="sony"></constructor-arg>
          <constructor-arg value="2700000"></constructor-arg>
       </bean>

       <bean id="sony" class="polymorphism.SonySpeaker"/>
</beans>
```
- `<constructor-arg>` 엘리먼트는 ref와 value 속성을 사용하여 생성자 매개변수로 전달할 값을 지정할 수 있다.
- 생성자가 여러 개 오버로딩 되어있다면 index속성을 지원해, 어떤 값이 몇 번째 매개변수로 매핑되는지 지정할 수 있다.
- index는 0부터 시작한다.

applicationContext.xml
```xml
<constructor-arg value="2700000" index="1"></constructor-arg>
```

### 3.2.2 의존관계 변경
- 다른 스피커로 교체하는 상황을 본다.

speaker.java
```java
package polymorphism;

public interface Speaker{
  void volumeUp();
  void volumeDown();
}
```
AppleSpeaker.java
```java
package polymorphism;

public class AppleSpeaker implements Speaker{
  public AppleSpeaker(){
    System.out.println("===> AppleSpeaker 객체 생성");
  }
  public void volumeUp(){
    System.out.println("AppleSpeaker---소리 올린다.");
  }
  public void volumeDown(){
    System.out.println("AppleSpeaker---소리 내린다.");
  }
}
```
SamsungTV.java
```java
package polymorphism;

public class SamsungTV implements TV{
  private Speaker speaker; //수정
  private int price;

  public SamsungTV(){
    System.out.println("===> SamsungTV(1) 객체 생성");
  }

  public SamsungTV(Speaker speaker){
    System.out.println("===> SamsungTV(2) 객체 생성");
    this.speaker = speaker;
  }


  public SamsungTV(Speaker speaker,int price){
    System.out.println("===>SamsungTV(3) 객체 생성");
    this.price = price;
  }

  public void powerOn(){
    System.out.println("SamsungTV --- 전원 켠다.(가격:"+price")");
  }
  public void powerOff(){
    System.out.println("SamsungTV --- 전원 끈다.");
  }
  public void volumeUp(){
    speaker.volumeUp();
  }
  public void volumeDown(){
    speaker.volumeDown();
  }
}
```
AppleSpeaker도 스프링 설정 파일에 bean등록한고 속성값을 apple로 지정하면 SamsungTV가 AppleSpeaker를 이용하여 볼륨을 조절한다.

applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.Springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
       xsi:schemaLocation="http://www.Springframework.org/schema/beans/Spring-beans.xsd">

       <bean id="tv" class="polymorphism.SamsungTV">
          <constructor-arg ref="apple"></constructor-arg>
          <constructor-arg value="2700000"></constructor-arg>
       </bean>

       <bean id="sony" class="polymorphism.SonySpeaker"/>

       <bean id="apple" class="polymorphism.AppleSpeaker"/>
</beans>
```
스피커가 AppleSpeaker로 변경되어 실행된다.
- 스프링 설정 파일 관리만으로 자바코드를 변경하지 않고 가능하다.

## 3.3 Setter 인젝션 이용하기
- 생성자 인젝션은 생성자를 이용하여 의존성을 처리한다.
- 대부분 Setter인젝션을 사용한다.

### 3.3.1 Setter 인젝션 기본

Setter 메소드를 추가한다.

SamsungTV.java
```java
package polymorphism;

public class SamsungTV implements TV{
  private SonySpeaker speaker;

  private int price;

  public SamsungTV(){
    System.out.println("===> SamsungTV(1) 객체 생성");
  }

//setter메소드
  public void setSpeaker(Speaker speaker){
    System.out.println("===> setSpeaker() 호출");
    this.speaker = speaker;
  }

  public void setPrice(int price){
    System.out.println("===> setPrice() 호출");
    this.price = price;
  }

  public void powerOn(){
    System.out.println("SamsungTV --- 전원 켠다.(가격:"+price")");
  }
  public void powerOff(){
    System.out.println("SamsungTV --- 전원 끈다.");
  }
  public void volumeUp(){
    speaker.volumeUp();
  }
  public void volumeDown(){
    speaker.volumeDown();
  }
}
```

- Setter메소드는 스프링 컨테이너가 자동으로 호출한다.
- 호출하는 시점은 bean 객체 생성 직후이다.
- 기본 생성자도 반드시 필요하다.

setter 인젝션을 이용하려면 스프링 설정파일에 `<property>`엘리먼트를 사용해야 한다.

applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.Springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
       xsi:schemaLocation="http://www.Springframework.org/schema/beans/Spring-beans.xsd">

       <bean id="tv" class="polymorphism.SamsungTV">
          <property ref="apple"></property>
          <property value="2700000"></property>
       </bean>

       <bean id="sony" class="polymorphism.SonySpeaker"/>
       <bean id="apple" class="polymorphism.AppleSpeaker"/>
</beans>
```
- name 속성 값이 호출 하고자 하는 메소드 이름이다.
- 예를 들어 name=speaker 이면 호출되는 메소드 이름은 setSpeaker()이다.
- 빈 객체를 인자로 넘기려면 `ref`속성이고, 기본형 데이터를 넘기려면 `value`속성을 사용한다.


### 3.3.2 p 네임스페이스 사용하기
Setter 인젝션을 설정할 때 `p 네임 스페이스`를 이용하면 좀 더 효율적으로 의존성 주입을 처리할 수 있다.

applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.Springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
       //요기 추가(sts기능으로도 추가 가능하다.)
       xmlns:p ="http://www.Springframework.org.schema/p"
       xsi:schemaLocation="http://www.Springframework.org/schema/beans/Spring-beans.xsd">

</beans>
```
- p 네임 스페이스를 선언 하고 참조할 객체를 할당할 수 있다.
- `p:변수명-ref="침조할 객체의 이름이나 아이디"`
- `p:변수명="설정할 값"`

applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.Springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLShema-instance"
       //요기 추가(sts기능으로도 추가 가능하다.)
       xmlns:p ="http://www.Springframework.org.schema/p"
       xsi:schemaLocation="http://www.Springframework.org/schema/beans/Spring-beans.xsd">

       <bean id="tv" class="polymorphism.SamsungTV" p:speaker-ref="sony" p:price="2700000">

       <bean id="sony" class="polymorphism.SonySpeaker"/>
       <bean id="apple" class="polymorphism.AppleSpeaker"/>
</beans>
```

## 3.4 컬렉션 객체 설정
- List나 배열 같은 컬렉션 객체를 이용하여 데이터 집합을 사용해야 하는 경우가 있다.
- 컬렉션 객체를 의존성 주입한다.
- java.util.List,배열 => ` <list> `
- java.util.Set => `<set>`
- java.util.Map => `<map>`
- java.util.Properties => `<props>`

### 3.4.1 List 타입 맵핑

CollectionBean.java
```java
package com.springbook.ioc.injection;

import java.util.List;

public class CollectionBean {
  private List<String> addressList;

  public void setAddressList(List<String> addressList){
    this.addressList = addressList;
  }

  public List<String> getAddressList(){
    return addressList;
  }
}
```

applicationContext.xml
```xml
<bean id ="collectionBean"
  class="com.springbook.ioc.injection.CollectionBean">
  <property name="addressList">
    <list>
      <value> 서울시 </value>
      <value> 경기도 </value>
    </list>
  </property>
<bean>
```
- 두 개의 문자열 주소가 저장된 List 객체를 CollectionBean 객체의 setAddressList()메소드를 호출할 때, 인자로 전달하여 getAddressList 멤버변수를 초기화하는 설정이다.

CollectionBeanClient.java
```java
package com.springbook.ioc.injection;

import.java.util.List;

public class CollectionBeanClient{
  public static void main(String[] args){
    AbstractApplicationContext factory = new GenericXmlApplicationContext("applicationContext.xml");

    CollectionBean bean = (CollectionBean) factory.getBean("collectionBean");
    List<String> addressList = bean.getAddressList();
    for(String address : addressList){
      System.out.println(address.toString());
    }
    factory.close();
  }
}
```

### 3.4.2 Set 타입 매핑
- 중복 값을 허용하지 않는 집합 객체를 사용할 때는 `java.util.Set`컬렉션을 사용한다.

```java
public class CollectionBean{
  private Set<String> addressList;
  public void setAddressList(Set<String> addressList){
    this.addressList = addressList;
}
```
```xml
<bean id ="collectionBean"
  class="com.springbook.ioc.injection.CollectionBean">
  <property name="addressList">
    <set value-type="java.lang.String">
      <value> 서울시 </value>
      <value> 경기도 </value>
      <value> 경기도 </value>
    </set>
  </property>
```
- 경기도가 중복 값인데 실행해보면 하나만 저장된다.

### 3.4.3 Map 타입 매핑
- 특정 key로 데이터를 등록하고 사용할 때는 `java.util.Map` 컬렉션을 사용한다.

```java
public class CollectionBean{
  private Map<String, Controller> addressList;

  public void setAddressList(Map<String, Conntroller> mapping){
    this.mappings = mappings;
  }
}
```

```xml
<property name="addressList">
  <map>
    <entry>
      <key><value>고길동</value></key>
      <value>서울시 강남구 역담동</value>
    </entry>
    <entry>
      <key<value>마이콜</value></key>
      <value>서울시 강서구 화곡동</value>
    </entry>
  </map>
  </property>
```

### 3.4.4 Properties 타입 매핑
key = value 형태의 데이터를 등록하고 사용할 때는 `java.util.Properties`라는 컬렉션을 사용하며 props 엘리먼트를 사용하여 설정한다.

```java
public class CollectionBean{
  private Properties addressList;

  public void setAddressList(Properties mappings){
    this.mappings = mappings;
  }
}
```
```xml
<bean id="collectionBean" class="com.springbook.ioc.injection.CollectionBean">
<property name="addressList">
  <props>
    <prop key="고길동">서울시 강남구 </prop>
    <prop key="마이콜">서울시 화곡동</prop>
  </props>
</Property>
</bean>
```
