제 7장
## 7.1 스프링JDBC
- JDBC는 가장 오랫동안 자바 개바랒들이 사용한 DB연동 기술이다.
- 정보 하나를 조회하기 위해 많은 자바 코드를 사용해야 하는데 이를 보완해준다.
- 메소드를 구현할 때 마다 아래처럼 반복해서 작성해야한다.
UserDAO.java
```java
public class UserDAO {
  private Connection conn;
  private PreparedStatement stmt;
  private ResultSet rs;

  private static final String USER_GET = "select * from users where id=?";

  //회원 상제조회
  public UserVO getUser(UserVO vo){
    UserVO user = null;
    try{
      Class.forName("orcle.jdbc.driver.OracleDriver"); //인터페이스 초기화, 클래스 오브젝트 리턴
      String url = "jdbc:oracle:thin:@localhost:1521:xe";
      conn = DriverManager.getConnection(url,"hr","hr"); //conn객체 생성 (url,id,pw)
      stmt = conn.PreparedStatement(USER_GET); // 재사용할수 있음
      stmt.setString(1,vo.getId());
      rs = stmt.executeQuery();
      //execute -> return 이 ResultSet일시 true,아닐시 false 리턴
      //executeQuery -> ResultSet 객체 값 반환
      //executeUpdate -> 레코드 건수를 반환 Insert/delete/update : 레코드 건수 create/drop : -1 반환
      if(rs.next()){
        user = new UserVO();
        user.setId(rs.getString("ID"));
        user.setPassword(rs.getString("PASSWORD"));
        user.setName(rs.getString("NAME"));
        user.setRole(rs.getString("ROLE"));
      }
    }catch (Exception e){
        e.printStackTrace();
      } finally {
        try {
          if(rs != null) rs.close();
        } catch (Exception e){
          e.printStackTrace();
        } finally {
          rs = null;
        }
        try {
          if (stmt != null) stmt.close();
        } catch (Exception e){
          e.printStackTrace();
        } finally {
          stmt = null;
        }
        try {
          if(conn != null) conn.close();
        } catch(Exception e){
          e.printStackTrace();
        } finally {
          conn =  null;
        }
      }
      return user;
    }
}
```

- JDBCUtil 클래스를 사용하여 구현한 BoardDAO 클래스이다.


BoardDAO.java

```java
private final String BOARD_INSERT = "insert into board(seq, title, writer, content) values((select nvl(max(seq,0 )+1 from board),?,?,?)";

private final String BOARD_UPDATE = "update board set title=?, content=?, where seq=?";

//글등록
public void insertBoard(BoardVO vo){
  try{
    conn = JDBCUtil.getConnection();
    stmt = conn.PreparedStatement(BOARD_INSERT);
    stmt.setString(1, vo.getTitle());
    stmt.setString(2, vo.getWriter());
    stmt.setString(3, vo.getContent());
    stmt.executeUpdate();
  } catch (Exception e){
    e.printStackTrace();
  } finally{
    JDBCUtil.close(stmt,conn);
  }
}

public void updateBoard(BoardVO vo){
  try{
    conn = JDBCUtil.getConnection();
    stmt = conn.PreparedStatement(BOARD_UPDATE);
    stmt.setString(1, vo.getTitle());
    stmt.setString(2, vo.getContent());
    stmt.setString(3, vo.getSeq());
    stmt.executeUpdate();
  } catch (Exception e){
    e.printStackTrace();
  } finally{
    JDBCUtil.close(stmt,conn);
  }
}
```

## 7.2 JDBCTemplate 클래스
- 메소드 패턴이 적용된 클래스이다.
- 복잡하고 반복되는 알고지름을 캡슐화해서 재사용하는 패턴으로 정의할 수 있다.
- 설정값만 신경쓰면 된다.
- 반복적인 코드를 제거하기 위해 제공하는 클래스다.

## 7.3 스프링 JDBC 설정
### 7.3.1 라이브러리 추가
- JDBC를 이용하려면 pom.xml 파일에 DBCP 관련 <dependency> 설정을 추가해야한다.

pom.xml

```xml
<!-- Spring -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
  <version>${org.springframework-version}</version>
</dependency>

<!-- DBCP -->
<dependency>
  <groupId>commons-dbcp</groupId>
  <artifactId>commons-dbcp</artifactId>
  <version>1.4</version>
</dependency>
```

### 7.3.2 DataSource 설정
- DB연동을 처리하려면 반드시 데이터베이스로부터 커넥션을 얻어야 한다.
- JDBCTemplate 객체가 사용할 DataSourc를 <bean> 등록하여 스프링 컨테이너가 생성하도록 해야한다.
- 매우 중요한 설정이다.
- 연결에 필요한 프로퍼티들을 setter 인젝션으로 설정해 주면된다.

applicationContext.xml
```xml
<!-- DataSource 설정 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
  <property name="driverClassName" value="org.h2.Driver"/>
  <property name="url" value="jdbc:h2:tcp://localhost/~/test"/>
  <property name="username" value="sa"/>
  <property name="password" value=""/>
</bean>
```

### 7.3.3 프로퍼티 파일을 활용한 DataSource 설정
- 외부의 프로퍼티 파일을 참조하여 DataSourc를 설정할 수 있다.
- 프로퍼티 파일을 사용하려면  `<context:property-placeholder>`엘리먼트로 프로퍼티 파일의 위치를 등록해야 한다.


database.properties
```
jdbc.driver = org.h2.Driver
jdbc.url = jdbc:h2:tcp://localhost/~/test
jdbc.username=sa
jdbc.password=
```

applicationContext.xml
```xml
<!-- DataSource 설정 -->
<context:property-placeholder location="classpath:config/database.properties"/>
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
  <property name="driverClassName" value="${jdbc.driver}"/>
  <property name="url" value="${jdbc.url}"/>
  <property name="username" value="${jdbc.username}"/>
  <property name="password" value="${jdbc.password}"/>
</bean>
```

## 7.4 JDBCTemplate 메소드
- JDBCTemplate 객체를 이용하여 DB 연동을 간단하게 처리할 수 있다.


### 7.4.1 Update() 메소드
- INSERT,UPDATE,DELETE 구문을 처리하려면 JDBCTemplate 클래스의 update() 메소드를 사용한다.
- `int update(String sql,Object... args)`
```java
public void updateBoard(BoardVO vo){
  String BOARD_UPDATE = "upadate board set title=?, content=?, where seq=?";
  int cnt = jdbcTemplate.update(BOARD_UPDATE,vo.getTitleO(),vo.getContent(),vo.getSeq());
  System.out.println(cnt+"건 데이터 수정");
}
```
- `int update(String sql, Object[] args)`
```java
public void updateBoard(BoardVO vo){
  String BOARD_UPDATE = "upadate board set title=?, content=?, where seq=?";
  Object[] args = {vo.getTitle(), vo.getContent(), vo.getSeq()};
  int cnt = jdbcTemplate.update(BOARD_UPDATE,args);
  System.out.println(cnt+"건 데이터 수정");
  ```

### 7.4.2 queryForInt() 메소드
- select 구문으로 검색된 정수값을 리턴받으려면 queryForInt()메소드를 사용한다.
- `int queryForInt(String sql)`

  `int queryForInt(String sql, Object ...args)`

  `int queryForInt(String sql,Object[] args)`

```java
public int getBoardTotalCount(BoardVO vo){
  String BOARD_TOT_COUNT = "select count(*) from board";
  int count = jdbcTemplate.queryForInt(BOARD_TOT_COUNT);
  System.out.println("전체 게시글 수 : "+count+건);
}
```

### 7.4.3 queryForObject() 메소드
- queryForObject() 메소드는 select 구문의 실행결과를 특정 자바 객체로 매핑하여 리턴받을 대 사용한다.
- 메소드는 검색 결과가 없거나 검색 결과가 두 개 이상히면 예외를 발생시킨다.
- 매핑할 RowMapper 객체를 반드시 지정해야 한다.
- `Object queryForObject(String sql)`

  `Object queryForObject(String sql,RowMapper<T> rowMapper)`

  `Object queryForObject(String sql,Object[] args,RowMapper<T> rowMapper)`

  ```java
  public BoardVO getBoard(BoardVO vo){
    String BOARD_GET = "select * from board where seq=?";
    Object[] args = {vo.getSeq()};
    return jdbcTemplate.queryForObject(BOARD_GET,args,new BoardRowMapper());
  }
  ```
- 검색 결과를 특정 VO 객체에 매핑하여 리턴하려면 RowMapper 인터페이스를 구현한 RowMapper 클래스가 필요하다.
- RowMapper 객체를 queryForObject() 메소드의 매개변수로 넘겨주면, 스프링 컨테이너는 SQL 구문을 수행한 후 자동으로 RowMapper 객체의 mapRow()메소드를 호출한다.

BoardRowMapper.java
```java
package com.springbook.biz.board.impl;

import java.sql,ResultSet;
import java.sql.SQLException;

import com.springbook.biz.board.BoardVO;
import org.springframework.jdbc.core.RowMapper;

private final String BOARD_INSERT = "insert into board(seq, title, writer, content) values((select nvl(max(seq,0 )+1 from board),?,?,?)";

private final String BOARD_UPDATE = "update board set title=?, content=?, where seq=?";

//글등록
public class BoardRowMapper impliments RowMapper<BoardVO>{
  public BoardVO mapROW(ResultSet rs, int rowNum) throws SQLException{
    BoardVO board = new BoardVO();
    board.setSeq(rs.getInt("SEQ"));
    board.setTitle(rs.getString("TITLE"));
    board.setWriter(rs.getString("WRITER"));
    board.setContent(rs.getString("CONTENT"));
    board.setRegDate(rs.getDate("REGDATE"));
    board.setCnt(rs.getInt("CNT"));

    return board;
  }
}
```
