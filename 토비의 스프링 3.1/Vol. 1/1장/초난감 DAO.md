> *DAO*[^Data Access Object]
> *DAO*는 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트이다.
##### 1.1.1 User
사용자 정보를 저장하는 오브젝트를 만들어보자. 다음의 코드는 *자바빈(JavaBean)*[^1] 규약을 따른다.
```java
public class User {
	String id;
	String name;
	String password;

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
}
```
그리고 다음의 쿼리를 통해 생성되는 DB 테이블이 존재한다고 가정하자.
```SQL
create table users (
	id varchar(10) primary key,
	name varchar(20) not null,
	password varchar(10) not null
)
```
##### 1.1.2 UserDao
사용자 정보를 DB에 넣고 관리할 수 있는 *DAO* 클래스를 만들어보자. 일단 새로운 사용자를 생성하고(`add`), 아이디를 가지고 사용자의 정보를 읽어오는(`get`) 메서드를 만들어보자.
*JDBC*를 이용하는 작업의 일반적인 순서는 다음과 같다.
- DB 연결을 위한 `Connection`을 가져온다.
- SQL을 담은 `Statement`(또는 `PreparedStatement`)를 만들고, 실행한다.
- 조회의 경우 SQL 쿼리의 실행 결과를 `ResultSet`으로 받아서 정보를 저장할 오브젝트(`User`)에 옮겨준다.
- 작업 중 생성된 `Connection`, `Statement`, `ResultSet` 같은 리소스는 작업을 마친 후 반드시 닫아준다.
- *JDBC* API가 만들어내는 예외를 잡아서 직접 처리하거나, 메서드에 `throws`를 선언해서 예외가 발생하면 메서드 밖으로 던지게 한다.
```java
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
			"jdbc:mysql://localhost/springbook", "spring", book");

		PreparedStatement ps = c.prepareStatement(
			"insert int users(id, name, password) values(?, ?, ?)");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());

		ps.executeUdate();

		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
			"jdbc:mysql://localhost/springbook", "spring", "book");
		PreparedStatement ps = c.prepareStatement(
			"select * from users where id ?");
		ps.setString(1, id);

		ResultSet rs = ps.executeQuery();
		rs.next();
		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));

		rs.close();
		ps.close();
		c.close();

		return user;
	}
}
```
##### 1.1.3 main()을 이용한 DAO 테스트 코드
이제 `main` 메서드를 만들고 그 안에서 `UserDao`의 오브젝트를 생성해서 `add()`와 `get()` 메서드를 검증해보자.
```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
	UserDao dao = new UserDao();

	User user = new User();
	user.setId("whiteship");
	user.setName("백기선");
	user.setPassword("married");

	dao.add(user);

	System.out.println(user.getId() + " 등록 성공");

	User user2 = dao.get(user.getId());
	System.out.println(user2.getName());
	System.out.println(user2.getPassword());

	System.out.println(user2.getId() + " 조회 성공");
}
```
위의 `main()` 메서드를 실행하면 다음과 같은 테스트 성공 메시지를 확인할 수 있다.
```text
whiteship 등록 성공
백기선
married
whiteship 조회 성공
```
이렇게 만들어진 `UserDao`가 어떤 부분에서 부족한지, 어떻게 개선할 수 있는지 등을 알아보면서 **Spring**에 대해 알아볼 것이다.

#TobySpring #Spring 

[^1]: 자바빈(JavaBean)은 원래 비주얼 툴에서 조작 가능한 컴포넌트를 말한다. 이제는 자바빈이라고 말하면 비주얼 컴포넌트라기보다는 기본 생성자(매개변수가 없는 생성자)와 프로퍼티를 가지는 오브젝트를 가리킨다. 기본 생성자는 리플렉션을 이용해 오브젝트를 생성하기 때문에 필요하고, 프로퍼티는 *setter*와 *getter*를 이용해 수정 또는 조회할 수 있다. 간단히 *빈(Bean)* 이라고 부르기도 한다.