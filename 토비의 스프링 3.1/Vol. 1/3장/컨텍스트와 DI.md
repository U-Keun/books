##### 3.4.1 `JdbcContext`의 분리
전략 패턴의 관점에서 `UserDao`의 메서드는 클라이언트이고, 익명 내부 클래스로 만들어진 것이 개별적인 전략이고, `jdbcContextWithStatementStrategy()` 메서드는 컨텍스트다. 여기서 `jdbcContextWithStatementStrategy()`는 JDBC의 일반적인 작업 흐름을 담고 있기 때문에 다른 DAO에서도 사용하도록 클래스 밖으로 독립시킬 수 있다.
###### 클래스 분리
`JdbcContext` 클래스를 만들어서 `UserDao`에 있던 컨텍스트 메서드를 분리해보자. 여기서 `UserDao`는 더이상 `DataSource`를 필요로 하지 않는다. 이것도 함께 옮겨서 코드를 작성해보자.
```java
public class JdbcContext {
	private DataSource dataSource;

	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
		Connection c = null;
		PreparedStatement ps = null;

		try {
			c = this.dataSource.getConnection();
			
			ps = stmt.makePreapredStatement(c);
			
			ps.executeUpdate();
		} catch (SQLException e) {
			throw e;
		} finally {
			if (ps != null) {try { ps.close(); } catch (SQLException e) { } }
		if (c != null) {try { c.close(); } catch (SQLException e) { } }
		}
	}
}
```

`UserDao`에서는 `JdbcContext`를 DI 받아서 사용할 수 있도록 수정한다.
```java
public class UserDao {
	...
	private JdbcContext jdbcContext;

	public void setJdbcContext(JdbcContext jdbcContext) {
		this.jdbcContext = jdbcContext;
	}

	public void add(final User user) throws SQLException {
		this.jdbcContext.workWithStatementStrategy(new StatementStrategy() { ... }
		);
	}
	
	public void deleteAll() throws SQLException {
		this.jdbcContext.workWithStatementStrategy(new StatementStrategy() { ... }
		);
	}
}
```
###### 빈 의존관계 변경


#TobySpring #Spring 