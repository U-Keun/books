##### 3.4.1 `JdbcContext`의 분리
전략 패턴의 관점에서 `UserDao`의 메서드는 클라이언트이고, 익명 내부 클래스로 만들어진 것이 개별적인 전략이고, `jdbcContextWithStatementStrategy()` 메서드는 컨텍스트다. 여기서 `jdbcContextWithStatementStrategy()`는 JDBC의 일반적인 작업 흐름을 담고 있기 때문에 다른 DAO에서도 사용하도록 클래스 밖으로 독립시킬 수 있다.
###### 클래스 분리


#TobySpring #Spring 