독립된 JDBC 작업 흐름이 담긴 `jdbcContextWithStatementStrategy()`는 DAO 메서드들이 공유할 수 있게 됐다. 이것이 컨텍스트가 되고, 전략은 `PreparedStatement` 객체를 생성하는 것이다.
##### 3.3.1 전략 클래스의 추가 정보
`deleteAll()`에 적용했던 전략 패턴을 `add()` 메서드에도 적용해보자.
```java
public class AddStatement implements StatementStrategy {
	User user;

	public AddStatement(User user) {
		this.user = user;
	}

	public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
		PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());
		
		return ps;
	}
}
```

`deleteAll()` 메서드와 다르게 `PreparedStatement`를 생성할 때 `user` 정보가 필요하기 때문에, 클라이언트로부터 `User` 객체를 주입받아서, 주입받은 객체로부터 정보를 제공받는다.

##### 3.3.2 전략과 클라이언트의 동거


#TobySpring #Spring 