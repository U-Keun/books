JUnit은 Java의 표준 테스팅 프레임워크로 불릴 만큼 폭넓게 사용되고 있다. **Spring** 프레임워크 자체도 JUnit 프레임워크를 이용해 테스트를 만들어가며 개발됐다. 또한 *스프링 테스트 모듈*도 JUnit을 사용하기 때문에 JUnit의 사용법은 알아두는 것이 좋다.
##### 2.3.1 JUnit 테스트 실행 방법
`JUnitCore`를 이용해 테스트를 실행하는 방법 또한 테스트의 수가 많아지면 관리하기가 힘들어진다. 가장 좋은 JUnit 테스트 실행 방법은 Java IDE에 내장된 JUnit 테스트 지원 도구를 사용하는 것이다.
###### IDE
> 이 책에서는 이클립스 IDE에서 지원하는 JUnit 테스트를 사용하는 방법에 대한 묘사가 포함되어 있다. 현재 나는 IntelliJ IDEA를 사용하고 있는데, 여기서 묘사된 대부분의 기능들이 IntelliJ IDEA에 포함되어 있다.

`JUnitCore`를 이용해 `main()` 메서드를 만들지 않고도 간단하게 메뉴를 통해 테스트를 실행시킬 수 있다. 테스트가 시작되면 JUnit 테스트 정보를 표시해주는 화면이 나타나고, 테스트의 진행 상황을 확인할 수 있다: 테스트의 총 수행시간, 실행한 테스트의 수, 테스트 에러의 수, 테스트 실패의 수 등을 확인할 수 있다.

JUnit은 한 번에 여러 테스트를 동시에 실행할 수 있는데, 이클립스에서는 소스 트리에서 특정 패키지를 선택해서 해당 패키지 아래에 있는 모든 JUnit 테스트를 한 번에 실행시킬 수 있다.

> IntelliJ IDEA를 사용하면 테스트 코드를 위한 패키지가 따로 구성이 되는데, 위에서 이야기해놓은 내용을 만들어둔 것이 아닐까 싶다.
###### 빌드 툴
프로젝트의 빌드를 위해 *ANT*나 *메이븐*[^Maven] 같은 빌드 툴과 스크립트를 사용하고 있다면, 빌드 툴에서 제공하는 JUnit 플러그인이나 태스크를 이용해 JUnit 테스트를 실행할 수 있다. 테스트 실행 결과는 옵션에 따라 HTML이나 텍스트 파일의 형태로 확인할 수 있다.
##### 2.3.2 테스트 결과의 일관성
지금까지 테스트를 실행하면서, `UserDaoTest`를 실행하기 전에 DB의 USER 테이블 데이터를 모두 삭제해야 했다. 그렇지 않으면 테스트가 실패하는 경우가 생기는데, 이것은 테스트가 외부 상태에 따라 성공 여부가 결정되는 것이다. 코드에 변경사항이 없다면 테스트는 항상 동일한 결과를 내야 한다.

위에서 주목한 것을 해결하기 위해서는 `addAndGet()` 테스트를 마치고 나면 테스트가 등록한 사용자 정보를 삭제해서, 테스트를 수행하기 이전 상태로 만들어주어야 한다.
###### `deleteAll()`의 `getCount()` 추가
일관성 있는 결과를 보장하는 테스트를 만들기 위해 다음과 같은 메서드를 준비해보자.
- `deleteAll()` : USER 테이블의 모든 레코드를 삭제하는 메서드
	```java
	public void deleteAll() throws SQLException {
		Connectionc = dataSource.getConnection();
	
		PreparedStatement ps = c.prepareStatement("delete from users");
	
		ps.executeUpdate();
	
		ps.close();
		c.close();
	}
	```
- `getCount()` : USER 테이블의 레코드 개수를 반환하는 메서드
	```java
	public int getCount() throws SQLException {
		Connection c = dataSource.getConnection();
		
		PreparedStatement ps = c.prepareStatement("select count(*) from users");
		
		ResultSet rs = ps.executeQuery();
		rs.next();
		int count = rs.getInt(1);
		
		rs.close();
		ps.close();
		c.close();
		
		return count;
	}
	```
###### `deleteAll()`과 `getCount()`의 테스트
추가된 기능에 대한 테스트를 만들어보자. `deleteAll()` 메서드가 잘 작동하는지 확인하려면 메서드를 호출한 뒤 데이터베이스에 남아있는 것이 있는지 확인해야 한다. 그리고 `getCount()` 메서드는 레코드가 추가되거나 삭제되었을 때 레코드의 개수가 변화하는지 확인해야 한다. 

이 기능들은 독립적으로 테스트하기에는 애매한 부분이 있다. `deleteAll()` 메서드를 테스트하기 위해 `getCount()` 메서드를 통해 데이터베이스에 남아있는 데이터가 있는지 확인하는 방식을 선택할 수 있고, `getCount()` 메서드를 테스트하기 위해 `add()` 메서드나 `deleteAll()` 메서드를 사용하는 등 몇 가지 메서드 간의 상호 보완적인 테스트로 사용하는 것이 자연스럽다. 또한 `deleteAll()` 메서드는 기존의 `addAndGet()` 테스트에서 실행 전에 수동으로 USER 테이블의 내용을 모두 삭제해줘야 하는 것을 대신 해줄 수 있다.

위의 관찰을 적용한 테스트 코드를 보자.
```java
@Test
public void addAndGet() throws SQLException {
	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

	UserDao dao = context.getBean("userDao", UserDao.class);
	dao.deleteAll(); // USER 테이블 비우기
	assertThat(dao.getCount(), is(0)); // deleteAll(), getCount() 기능을 확인하는 부분

	User user = new User();
	user.setId("gyumee");
	user.setName("박성철");
	user.setPassword("springno1");

	dao.add(user);
	assertThat(dao.getCount(), is(1)); // getCount() 기능을 확인하는 부분
	
	User user2 = dao.get(user.getId());
	
	assertThat(user2.getName(), is(user.getName()));
	assertThat(user2.getPassword(), is(user.getPassword()));
}
```
더 고려해야 하는 부분이 많아 보이지만, 일단은 이 정도로 작성해두자.
###### 동일한 결과를 보장하는 테스트
이제 위의 코드는 반복해서 여러 번 실행해도 계속 성공한다. 그리고 이전에는 테스트하기 전에 매번 직접 DB에서 데이터를 삭제해야 했지만, 그 과정도 필요없어졌다.

물론 반복적인 테스트 실행에 동일한 결과를 얻을 수 있는 다른 방법도 있다. 예를 들면 `addAndGet()` 테스트를 마치기 전에 테스트가 변경하거나 추가한 데이터를 모두 원래 상태로 만들어주는 것이다. 하지만 이 방법에도 맹점은 있다. 테스트 실행 이전에 다른 이유로 USER 테이블에 데이터가 들어가 있다면 테스트가 항상 성공한다는 것을 보장할 수 없다. 상황에 따라 다르겠지만, 테스트 실행에 문제가 되지 않는 상태를 만들어주는 방향으로 테스트 코드를 작성하는 것이 더 낫다.

**Spring**은 DB를 사용하는 코드를 테스트하는 방법을 지원하지만, **Spring**의 기능을 좀 더 살펴보고 적용할 것이다. 일단은 위의 방식으로 사용하자.

단위 테스트는 항상 일관성 있는 결과가 보장돼야 한다는 점을 기억하자. DB에 남아 있는 데이터와 같은 외부 환경에 영향을 받지 않아야 하고, 테스트를 실행하는 순서를 바꿔도 동일한 결과가 보장되어야 한다.
##### 2.3.3 포괄적인 테스트
위에서 작성한 테스트 코드에서, `getCount()` 메서드에 대한 테스트는 테이블이 비어있는 경우와 테이블에 한 개의 레코드가 있는 경우만 확인한다. 이것으로 테스트가 충분하다고 생각할 수도 있지만, 미처 생각하지 못한 문제가 숨어 있을지도 모르니 더 꼼꼼한 테스트를 해보는 것이 좋은 자세다.
###### `getCount()` 테스트
이번에는 여러 개의 `User`를 등록해가면서 `getCount()` 메서드의 결과를 매번 확인해보는 코드를 작성해보자. 이번 테스트에서는 USER 테이블에 몇 개의 레코드가 있는지 세는 기능에만 집중할 것이다.

테스트 시나리오는 다음과 같다. 먼저 USER 테이블의 데이터를 모두 지우고 `getCount()` 메서드로 레코드 개수가 0임을 확인한다. 그리고 3개의 사용자 정보를 하나씩 추가하면서 매번 `getCount()`의 결과가 하나씩 증가하는지 확인할 것이다.

`User` 오브젝트를 여러 번 만들고 값을 넣어야 하니, `User` 클래스에 다음과 같은 생성자를 추가해둘 것이다.
```java
public User(String id, String name, String password) {
	this.id = id;
	this.name = name;
	this.password = password;
}
```

그리고 아래와 같이 `getCount()` 메서드에 대한 테스트 코드를 `addAndGet()` 테스트 아래에 작성하자.

```java
@Test
public void count() throws SQLException {
	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

	UserDao dao = context.getBean("userDao", UserDao.class);
	User user1 = new User("gyumee", "박성철", "springno1");
	User user2 = new User("leegw700", "이길원", "springno2");
	User user1 = new User("bumjin", "박범진", "springno3");
	
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));
	
	dao.add(user1);
	assertThat(dao.getCount(), is(1));

	dao.add(user2);
	assertThat(dao.getCount(), is(2));

	dao.add(user3);
	assertThat(dao.getCount(), is(3));
}
```

테스트를 실행하면 `addAndGet()` 테스트와 `count()` 테스트가 실행될 것이다. 주의할 점은 두 개의 테스트가 어떤 순서로 실행될지는 알 수 없다는 것이다. JUnit은 특정한 테스트 메서드의 실행 순서를 보장하지 않는데, 만약 테스트의 결과가 테스트의 실행 순서에 영향을 받는다면 그것은 올바른 테스트가 아니다.
###### `addAndGet()` 테스트 보완
기존의 `addAndGet()` 테스트에서는 DB의 데이터를 모두 삭제하고, 데이터를 하나 추가한 뒤 그 데이터가 제대로 조회되는지 확인하고 있었다. 여기서 DB에 저장되는 데이터는 하나뿐이기 때문에  `get()` 메서드가 사용자의 `id` 값을 이용해서 제대로 값을 조회한다는 것을 보장하지 않는다.[^1] 이 부분을 `User`를 하나 더 추가해서 두 개의 데이터가 있는 경우에 대한 로직을 구현하는 것으로 보완한다.

```java
@Test
public void addAndGet() throws SQLException {
	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

	UserDao dao = context.getBean("userDao", UserDao.class);
	dao.deleteAll();
	assertThat(dao.getCount(), is(0)); 

	User user1 = new User("gyumee", "박성철", "springno1");
	User user2 = new User("leegw700", "이길원", "springno2");

	dao.add(user1);
	dao.add(user2);
	assertThat(dao.getCount(), is(2));
	
	User userget1 = dao.get(user1.getId());
	assertThat(userget1.getName(), is(user.getName()));
	assertThat(userget1.getPassword(), is(user.getPassword()));

	User userget2 = dao.get(user1.getId());
	assertThat(userget2.getName(), is(user.getName()));
	assertThat(userget2.getPassword(), is(user.getPassword()));
}
```
###### `get()` 예외조건에 대한 테스트
`get()` 메서드에 전달된 `id` 값에 해당하는 사용자 정보가 없는 경우에 대한 처리는 구현되어있지 않다. 이 경우를 처리하는 방법으로 `null` 값을 반환하거나, `id`에 해당하는 정보를 찾을 수 없다는 것을 알리기 위한 예외를 던질 수 있다. 여기서는 예외를 던지는 방법을 사용해보자.

```java
@Test(expected = EmptyResultDataAccessException.class) // 테스트에서 던져질 것으로 기대하는 예외
public void getUserFailure() throws SQLException {
	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

	UserDao dao = context.getBean("userDao", UserDao.class);
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));

	dao.get("unknown_id"); // 예외가 발생하는 부분
}
```

이 테스트는 `dao.get("unknown_id")` 부분에서 `EmptyResultDataAccessException` 예외가 아닌 `SQLException` 예외를 던지기 때문에 실패한다. 이 테스트를 성공시키려면 `UserDao` 클래스의 `get()` 메서드를 수정해야 한다.
###### 테스트를 성공시키기 위한 코드의 수정
주어진 `id`에 해당하는 데이터가 없으면 `EmptyResultDataAccessException` 예외를 던지도록 `get()` 메서드를 수정해보자.
```java
public User get(String id) throws SQLException {
	...
	ResultSet rs = ps.executeQuery();

	User user = null;
	if (rs.next()) {
		user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));
	}

	rs.close();
	ps.close();
	c.close();
	
	if (user == null) throw new EmptyResultDataAccessException(1);
	
	return user;
}
```
###### 포괄적인 테스트
간단한 DAO라고 하더라도 포괄적인 테스트를 만들어두는 것이 훨씬 안전하고 유용하다. 단순하고 간단한 테스트가 치명적인 실수를 피할 수 있게 해줄 것이다.

개발자는 긍정적인 경우를 골라 성공할 만한 테스트만 골라서 만드는 경향이 있는데, 부정적인 경우에 대한 테스트를 먼저 만드는 습관을 들이는 것이 좋다. 위에서 작업한 것처럼 `get()` 메서드가 존재하지 않는 `id`가 주어졌을 때 어떻게 반응할 지를 결정하고, 이를 확인할 수 있는 테스트를 먼저 만들어서 예외적인 상황을 빠뜨리지 않는 꼼꼼한 개발에 도움이 될 것이다.
##### 2.3.4 테스트가 이끄는 개발
위에서 작업한 순서를 생각해보면 테스트를 먼저 만들어 테스트가 실패하는 것을 보고 나서 `UserDao`의 코드를 수정했다. 
###### 기능설계를 위한 테스트
좀 더 자세히 작업 순서를 돌이켜보면, 가장 먼저 '존재하지 않는 `id`로 `get()` 메서드를 실행하면 특정한 예외가 던져져야 한다'라고 만들어야 할 기능을 결정했다. 그리고 `UserDao` 코드를 수정하는 대신 `getUserFailure()` 테스트를 먼저 만들었다.

#TobySpring #Spring 

[^1]: 레코드를 조회하는 `get()` 메서드가 임의의 레코드를 조회하는 것일 수도 있다..!