##### 3.1.1 예외처리 기능을 갖춘 DAO
DB 커넥션이라는 리소스를 공유해서 사용하는 서버에서는 반드시 예외처리를 해야 한다. 중간에 어떤 이유로든 예외가 발생하면 사용한 리소스를 반드시 반환하도록 만들어야 한다.
###### JDBC 수정 기능의 예외처리 코드
`UserDao` 클래스의 `deleteAll()` 메서드를 보자.
```java
public void deleteAll() throws SQLException {
	Connection c = dataSource.getConnection();

	PreparedStatement ps = c.prepareStatement("delete from users");
	ps.executeUpdate();
	
	ps.close();
	c.close();
}
```
이 메서드에서는 `Connection`과 `PrepareStatement`라는 두 개의 공유 리소스를 가져와서 사용하는데, 만약 `PreparedStatement`를 처리하는 도중에 예외가 발생하면 메서드는 실행을 끝마치지 못하고 바로 메서드를 빠져나간다. 그러면 `Connection`과 `PrepareStatement`의 `close()` 메서드가 실행되지 않아서 제대로 리소스가 반환되지 않는다.

일반적으로 서버에서는 제한된 개수의 DB 커넥션을 만들어서 재사용 가능한 풀로 관리한다. DB 풀은 매번 `getConnection()`으로 가져간 커넥션을 명시적으로 `close()`해서 돌려줘야 재사용 할 수 있다. 반환되지 못한 `Connection`이 계속 쌓이면 리소스가 모자란다는 오류를 내며 서버가 중단된다. 

이런 JDBC 코드에서는 어떤 상황에서도 가져온 리소스를 반환하도록 `try/catch/finally` 구문 사용을 권장한다.[^1] 

#TobySpring #Spring 

[^1]: 책 Effective Java에서는 `try-with-resources`를 권장한다. 다음의 글도 참고하자: [try-with-resources를 사용해야 하는 이유](https://mangkyu.tistory.com/217)