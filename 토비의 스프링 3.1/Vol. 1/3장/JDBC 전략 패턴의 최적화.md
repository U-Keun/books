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
public void add(final User user) throws SQLException {
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

		StatementStrategy st = new AddStatement();
		jdbcContextWithStatementStrategy(st);
	}
}
```

이렇게 하면 클래스 파일도 하나 줄고, 자신이 선언된 곳의 정보에 직접 접근할 수 있기 때문에 `AddStatement` 클래스를 생성할 때 생성자를 통해 `User` 객체를 전달해줄 필요가 없다. `add()` 메서드에 매개변수로 주어지는 `user` 객체를 로컬 변수로 사용할 수 있기 때문이다. 추가로, `user` 객체가 메서드 내부에서 변경될 일이 없으므로 `final`로 선언해 두었다.
###### 익명 내부 클래스
내부 클래스를 작성하는 대신 익명 내부 클래스를 사용해서 위의 코드를 수정해보자.
```java
public void add(final User user) throws SQLException {
	jdbcContextWithStatementStrategy(
		StatementStrategy st = new StatementStrategy() {
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
				ps.setString(1, user.getId());
				ps.setString(2, user.getName());
				ps.setString(3, user.getPassword());
	
				return ps;
			}
		}
	);
}
```

`DeleteAllStatement`도 `deleteAll()` 메서드로 가져와서 익명 내부 클래스로 다음과 같이 처리할 수 있다.
```java
public void deleteAll() throws SQLException {
	jdbcContextWithStatementStrategy(new StatementStrategy) {
		public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
			return c.prepareStatement("delete from users");
		}
	}
}
```

#TobySpring #Spring 