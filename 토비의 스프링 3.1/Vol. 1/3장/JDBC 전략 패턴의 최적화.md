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
지금까지 작성한 코드에서는, DAO 메서드마다 새로운 `StatementStrategy` 구현 클래스를 만들어야 한다. 다시 말하면, DAO 메서드가 추가될 때마다 새로운 클래스를 만들어야 한다. 그리고 DAO에서 `StatementStrategy`에 전달할 정보(또는 오브젝트)가 있으면, 그 오브젝트를 전달받는 생성자와 그것을 저장해둘 인스턴스 변수를 만들어야 한다. 이 오브젝트는 컨텍스트가 전략 오브젝트를 호출할 때 사용되므로 어딘가에 저장해주어야 한다.
###### 로컬 클래스
클래스 파일이 많아지는 것은 내부 클래스로 정의하는 것으로 방지할 수 있다. 기존에 만들었던 `DeleteAllStatement`, `AddStatement` 클래스는 `UserDao`에서만 사용된다. 그러면 다음과 같이 로컬 클래스로 만들어서 특정 메서드에서만 사용하도록 코드를 작성할 수 있다.
```java
public void add(User user) throws SQLException {
	class AddStatement implements StatementStrategy {
		// User user;

		//public AddStatement(User user) {
		//	this.user = user;
		//}

		public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
			c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
			ps.setString(1, user.getId());
			ps.setString(2, user.getName());
			ps.setString(3, user.getPassword());
			
			return ps;
		}

		StatementStrategy st = new AddStatement(user);
		jdbcContextWithStatementStrategy(st);
	}
}
```

`AddStatement` 클래스는 로컬 클래스로 정의되어 있다. 이것은 `AddStatement` 객체가 사용될 곳이 `add()` 메서드뿐이라면, 위와 같은 방식으로 사용하기 전에 바로 정의해서 쓸 수 있다. 그러면 클래스 파일도 하나 줄고, 자신이 선언된 곳의 정보에 직접 접근할 수 있다. 

#TobySpring #Spring 