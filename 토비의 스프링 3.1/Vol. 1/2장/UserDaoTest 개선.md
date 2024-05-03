##### 2.2.1 테스트 검증의 자동화
테스트 결과의 검증 부분을 코드로 구현해보자. 먼저 어떤 테스트 결과가 나올 수 있는지 생각해보자.
- 테스트의 실패
	- 테스트가 진행되는 동안 에러가 발생하는 경우 -> *테스트 에러*
	- 테스트 작업 중에 에러가 발생하지 않았지만 그 결과가 기대한 것과 다르게 나오는 경우 -> *테스트 실패*

먼저 테스트가 실패하는 경우를 고려하여 기존의 테스트 코드를 다음과 같이 수정해보자.
```java
if (!user.getName().equals(user2.getName())) {
	System.out.println("테스트 실패 (name)");
} else if (!user.getPassword().equals(user2.getPassword())) {
	System.out.println("테스트 실패 (password)");
} else {
	System.out.println("조회 테스트 성공");
}
```

테스트가 실패하는 경우에 대해서 `name`과 `password` 중 어떤 것 때문에 실패했는지도 확인할 수 있도록 출력 메시지를 작성했다. 성공 메시지가 나온다면 테스트 에러도 없고, 테스트가 실패하는 경우도 없는 것이다. 예외가 발생하거나 실패 메시지가 나오면 그 원인을 찾아서 코드를 수정하고 다시 테스트를 실행해야 한다.

개발 과저에서, 또는 유지보수를 하면서 기존 어플리케이션 코드에 수정을 할 때 문제가 없는지 확인할 수 있는 가장 좋은 방법은 빠르게 실행 가능하고 스스로 테스트 수행과 기대하는 결과에 대한 확인까지 해주는 코드로 된 자동화된 테스트를 만들어두는 것이다.
##### 2.2.2 테스트의 효율적인 수행과 결과 관리
`main()` 메서드로 만든 테스트는 테스트로서는 작동하겠지만, 어플리케이션 규모가 커지고 테스트 개수가 많아지면 테스트를 수행하는 일이 효율적으로 작동하지 않을 수 있다. Java에는 실용적인 테스트를 위한 도구가 여러 개 있는데, 그중 테스팅 프레임워크라고 불리는 *JUnit*을 이용해보자.
###### JUnit 테스트로 전환
프레임워크는 제어의 역전을 이용해 동작한다고 이야기했었다. JUnit도 테스트에 대한 제어 권한을 넘겨받아서 그것의 흐름을 제어한다. 다시 말하면, JUnit의 테스트는 `main()` 메서드도 필요하지 않고 오브젝트를 만들어서 실행시키는 코드를 만들지 않아도 된다.
###### 테스트 메서드 전환
JUnit 프레임워크는 테스트 메서드에 대해 필요한 조건이 두 가지 있다. 하나는 그 메서드가 `public`으로 선언되어 있어야 한다는 것이고, 다른 하나는 메서드에 `@Test`라는 어노테이션을 붙여야 한다는 것이다. 다음의 코드를 보자.
```java
import org.junit.Test;
...
public class UserDaoTest {
	@Test
	public void addAndGet() throws SQLException {
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

		UserDao dao = context.getBean("userDao", UserDao.class);
		...
	}
}
```
###### 검증 코드 전환
테스트의 결과를 검증하는 조건문 문장을 JUnit이 제공하는 방법을 이용해 전환해보자.
```java
import org.junit.Test;
import static org.hamcrest.CoreMatcher.is;
import static org.junit.Assert.assertThat;
...
public class UserDaoTest {
	@Test
	public void addAndGet() throws SQLException {
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

		UserDao dao = context.getBean("userDao", UserDao.class);
		User user = new User();
		user.setId("gyumee");
		user.setName("박성철");
		user.setPassword("springno1");

		dao.add(user);
		
		User user2 = dao.get(user.getId());
		
		assertThat(user2.getName(), is(user.getName()));
		assertThat(user2.getPassword(), is(user.getPassword()));
	}
}
```

#TobySpring #Spring 