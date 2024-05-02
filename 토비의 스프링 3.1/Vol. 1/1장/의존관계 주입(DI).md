##### 1.7.1 제어의 역전(IoC)과 의존관계 주입
IoC가 너무 포괄적인 개념이라서 **Spring**을 IoC 컨테이너라고만 해서는 **Spring**이 제공하는 기능의 특징을 명확하게 설명하지 못하는 부분이 있다. 그래서 *의존관계 주입*[^Dependency Injection]이라는, 좀 더 의도가 명확히 드러나는 이름을 사용하기 시작했다. **Spring**에서  IoC 기능의 대표적인 동작원리는 주로 의존관계 주입이다. 
##### 1.7.2 런타임 의존관계 설정
###### 의존관계
두 개의 클래스 또는 모듈이 의존 관계에 있다고 말할 때는 항상 방향성을 부여해줘야 한다. UML 모델[^1]에서는 두 클래스의 *의존관계*[^dependency relationship]를 다음과 같이 점선으로 된 화살표로 표현한다. 아래의 그림은 A가 B에 의존하고 있음을 나타낸다. ![[스크린샷 2024-04-15 오전 2.05.42.png|center|400]]
'의존한다'는 것은 B가 변하면 그것이 A에 영향을 미친다는 뜻이다. B의 기능이 추가되거나 변경되거나, 형식이 바뀌거나 하면 그 영향이 A로 전달된다는 것이다. 대표적인 예는 A가 B를 사용하는 경우, 조금 더 구체적으로는 A에서 B에 정의된 메서드를 호출해서 사용하는 경우이다. 이럴 때는 '사용에 대한 의존관계'가 있다고 말할 수 있다. 만약 B에 새로운 메서드가 추가되거나 기존 메서드의 형식이 바뀌거나, 또는 기능이 내부적으로 변경되면 A도 그에 따라 수정되거나 추가돼야 할 것이다.
###### `UserDao`의 의존관계
`UserDao`는 `ConnectionMaker`에 의존하고 있는 형태였다. 그러므로 `ConnectionMaker` 인터페이스가 변한다면 그 영향을 `UserDao`가 직접적으로 받는다. 하지만 `ConnectionMaker` 인터페이스를 구현한 클래스인 `DConnectionMaker` 등이 다른 것으로 바뀌거나 그 내부에서 사용하는 메서드에 변화가 생겨도 `UserDao`가 직접적으로 의존하고 있지 않기 때문에 `UserDao`에 영향을 주지 않는다. 이렇게 인터페이스에 대해서만 의존관계를 만들어두면 인터페이스 구현 클래스와의 관계는 느슨해지면서 변화에 영향을 덜 받는 상태가 되는데, 이것을 '결합도가 낮다'고 표현한다. 
![[스크린샷 2024-04-15 오전 2.17.45.png|center|300]]

그런데 모델이나 코드에서 클래스와 인터페이스를 통해 드러나는 의존관계가 아닌, 런타임 시에 오브젝트 사이에서 만들어지는 의존관계도 있다. 이것을 *런타임 의존관계* 또는 *오브젝트 의존관계*라고 한다.

인터페이스를 통해 설계 시점에 느슨한 의존관계를 갖는 경우에는 `UserDao`의 오브젝트가 런타임시에 사용할 오브젝트가 어떤 클래스로 만든 것인지 미리 알 수 없다. 프로그램이 시작되고 `UserDao` 오브젝트가 만들어지고 나서 런타임 시에 의존관계를 맺는 대상, 즉 실제 사용대상인 오브젝트를 *의존 오브젝트*[^dependent object]라고 한다.

의존관계 주입은 이렇게 구체적인 의존 오브젝트와 그것을 사용할 클라이언트 오브젝트를 런타임시에 연결해주는 작업을 말한다.

정리해보면 의존관계 주입은 다음과 같은 세 가지 조건을 충족하는 작업이다.
- 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.
- 런타임 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.

의존관계 주입의 핵심은 설계 시점에는 알지 못했던 두 오브젝트의 관계를 맺도록 도와주는 제3의 존재가 있다는 것인데, 지금까지 봤던 `DaoFactory`, 어플리케이션 컨텍스트, 빈 팩토리, IoC 컨테이너 등이 모두 외부에서 오브젝트 사이의 런타임 의존관계를 맺어주는 책임을 지닌 제3의 존재라고 볼 수 있다. 
###### `UserDao`의 의존관계 주입
인터페이스를 사이에 두고 `UserDao`와 `ConnectionMaker` 구현 클래스 간에 의존관계를 느슨하게 만들긴 했지만, 런타임 의존관계가 코드에 미리 결정되어 있는 상황이 있었다.
```java
public UserDao() {
	connectionMaker = new DConnectionMaker();
}
```

그래서 IoC 방식을 사용하기 위해 `UserDao`로부터 런타임 의존관계를 드러내는 코드를 제거하고, `DaoFactory`에 런타임 의존관계 결정 권한을 위임했다.
```java
public class UserDao {
	private ConnectionMaker connectionMaker;

	public UserDao(ConnectionMaker connectionMaker) {
		this.connectionMaker = connectionMaker;
	}
}
```

`DaoFactory`는 런타임 시점에 `UserDao`가 사용할 `ConnectionMaker` 타입의 오브젝트를 결정하고 이를 생성한 후에 `UserDao`의 생성자 매개변수로 주입해서 `UserDao`가 `DConnectionMaker`의 오브젝트와 런타임 의존관계를 맺게 해준다. 따라서 의존관계 주입을 담당하는 컨테이너라고 볼 수 있고, *DI 컨테이너*라고 부르기도 한다.
```java
public class DaoFactory {
	public UserDao userDao() {
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao userDao = new UserDao(ConnectionMaker);
		return userDao;
	}
}
```
##### 1.7.3 의존관계 검색과 주입
코드에서는 구체적인 클래스에 의존하지 않고, 런타임 시에 의존관계를 결정한다는 점에서 의존관계 주입과 비슷하지만, 외부로부터의 주입이 아니라 스스로 필요로 하는 의존 오브젝트를 능동적으로 검색하는 *의존관계 검색*[^dependency lookup]도 **Spring**이 제공하는 기능 중 하나이다. 물론 자신이 어떤 클래스의 오브젝트를 이용할지 결정하는 것은 아니고, 런타임시 의존관계를 맺을 오브젝트를 결정하는 것과 오브젝트의 생성 작업은 외부 컨테이너에게 IoC로 맡기지만, 이를 가져올 메서드나 생성자를 통한 주입 대신 스스로 컨테이너에게 요청하는 방법을 사용한다.

예를 들어 `UserDao`의 생성자를 다음과 같이 만들었다고 가정해보자.
```java
public UserDao() {
	DaoFactory daoFactory = new DaoFactory();
	this.connectionMaker = daoFactory.connectionMaker();
}
```
위의 코드에서도 `UserDao`는 자신이 어떤 `ConnectionMaker` 오브젝트를 사용할지 미리 알지 못한다. 런타임 시에 `DaoFactory`가 만들어서 돌려주는 오브젝트와 런타임 의존관계를 맺는다. 하지만 적용 방법이 외부로부터의 주입이 아니라 스스로 IoC 컨테이너인 `DaoFactory`에게 요청하는 것이다. 이 작업을 일반화한 어플리케이션 컨텍스트는 미리 정해놓은 이름을 전달해서 그 이름에 해당하는 오브젝트를 검색한다. 

어플리케이션 컨텍스트의 `getBean()` 메서드가 의존관계 검색에 사용된다.
```java
public UserDao() {
	AnnotationCofigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
	this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```

의존관계 검색보다는 의존관계 주입 쪽이 코드는 더 단순하고 깔끔하다. 의존관계 검색을 사용하는 코드는 어플리케이션 컴포넌트가 컨테이너와 같이 성격이 다른 오브젝트에 의존하게 되는 것도 어색하다. 따라서 대부분의 경우에는 의존관계 주입 방식을 사용하는 편이 낫다. 

의존관계 검색이 필요한 경우도 있는데, 앞에서 만들었던 테스트 코드인 `UserDaoTest`가 그 예제이다. 테스트를 하기 위해서는 IoC와 DI 컨테이너를 적용했다고 하더라고 어플리케이션의 기동 시점에서 적어도 한 번은 의존관계 검색을 사용해 오브젝트를 가져와야 한다. 정적 메서드인 `main()` 메서드는 DI를 이용해 오브젝트를 주입받을 방법이 없기 때문이다. 서버에서도 사용자의 요청을 받을 때마다 서블릿[^2]에서 스프링 컨테이너에 담긴 오브젝트를 사용하려면 한 번은 의존관계 검색 방식을 사용해 오브젝트를 가져와야 한다.

의존관계 주입과 의존관계 검색의 중요한 차이점이 있다. 의존관계 검색 방식에서는 검색하는 오브젝트는 자신이 빈일 필요가 없지만, 의존관계 주입 방식에서는 DI가 적용되려면 주입받는 오브젝트 자신도 빈이어야 한다. 컨테이너가 오브젝트를 주입해주려면 각 대상에 대한 생성과 초기화 권한을 갖고 있어야 하기 때문이다.
##### 1.7.4 의존관계 주입의 응용
지금까지 살펴본 DI 기술의 장점으로는, 코드에는 런타임 클래스에 대한 의존관계가 나타나지 않고, 인터페이스를 통해 결합도가 낮은 코드를 만들기 때문에 다른 책임을 가진 사용 의존관계에 있는 대상이 바뀌거나 변경되더라도 자신은 영향을 받지 않으며, 변경을 통한 다양한 확장 방법에 자유롭다는 것들이 있었다. 이것을 활용할 몇 가지 응용 사례를 살펴보자.
###### 기능 구현의 교환
다음의 상황을 생각해보자. 실제 운영에 사용할 데이터베이스는 중요한 자원이기 때문에 개발 중에는 개발자 PC에 설치한 로컬 DB를 사용하고, 어느 시점이 되면 지금까지 개발한 것을 그대로 운영서버로 배치해서 사용할 것이다. 만약 DI 방식을 적용하지 않았다면, 개발 중에는 로컬 DB를 사용해야 하니 로컬 DB에 대한 연결 기능이 있는 `LocalDBConnectionMaker`라는 클래스를 만들고, 모든 DAO에서 이 클래스의 오브젝트를 매번 생성해서 사용하게 했을 것이다. 이것을 서버에 배치하는 시점에 운영서버에서 DB에서 연결할 때 필요한 `ProductionDBConnectionMaker`라는 클래스로 변경해줘야 한다. DAO가 여러 개 있다면 그 코드를 모두 수정해야 한다. 하지만 DI 방식을 적용해서 만들었다면 아래의 코드만 수정하면 된다.
```java
@Bean
public ConnectionMaker connectionMaker() {
	return new LocalDBConnectionMaker(); // 수정할 부분
	// return new ProductionDBConnectionMaker();
}
```

개발환경과 운영환경에서 DI의 설정정보에 해당하는 `DaoFactory`만 다르게 만들어두면 나머지 코드에는 전혀 손대지 않고 개발 시와 운영 시에 각각 다른 런타임 오브젝트에 의존관계를 갖게 해줘서 문제를 해결할 수 있다.

또한 테스트용으로 별도의 테스트 DB를 만들어서 개발한 코드를 테스트할 때에도, DAO 코드와 상관 없이 테스트 DB에 접속하는 방법을 가진 `ConnectionMaker` 구현 클래스를 만들고, 그것을 테스트에서 사용할 `DaoFactory` 설정에 넣어주면 된다. 테스트가 수행되는 시점에서 테스트용 DB에 연결해주는 오브젝트를 DI 컨테이너가 생성해서 모든 DAO가 사용할 수 있도록 DI 해준다.
###### 부가기능 추가
DAO가 DB를 얼마나 많이 연결해서 사용하는지 파악하고 싶은 경우를 생각해보자. 이것을 위해 모든 DAO의 `makeConnection()` 메서드를 호출하는 부분에 카운터를 증가시키는 코드를 넣는 것은 좋은 방법이 아니다. DB 연결횟수를 세는 일은 DAO의 관심사항이 아니므로 어떻게든 분리되어야 할 책임이기도 하다.

DI 컨테이너를 이용하면 간단하게 해결할 수 있다. DAO와 DB 커넥션을 만드는 오브젝트 사이에 연결횟수를 카운팅하는 오브젝트를 하나 더 추가하는 것이다. 
```java
public class CountingConnectionMaker implements ConnectionMaker {
	int counter = 0;
	private ConnectionMaker realConnectionMaker;

	public CountingConnectionMaker(ConnectionMaker realConnectionMaker) {
		this.realConnectionMaker = realConnectionMaker;
	}

	public Connection makeConnection() throws ClassNotFoundException, SQLException {
		this.counter++;
		return realConnectionMaker.makeConnection();
	}

	public int getCounter() {
		return this.counter;
	}
}
```
`CountingConnectionMaker` 클래스는 `ConnectionMaker` 인터페이스를 구현했지만 내부에서 직접 DB 커넥션을 만들지 않고, DAO가 DB 커넥션을 가져올 때마다 호출하는 `makeConnection()`에서 DB 연결횟수 카운터를 증가시킨다. `CountingConnectionMaker`는 자신의 관심사인 DB 연결횟수 카운팅 작업을 마치면 실제 DB 커넥션을 만들어주는 `realConnectionMaker`에 저장된 `ConnectionMaker` 타입 오브젝트의 `makeConnection()`을 호출해서 그 결과를 DAO에게 돌려준다. 그리고 `realConnectionMaker` 또한 *DI*를 받는 것이다.

`UserDao`는 `ConnectionMaker` 인터페이스에만 의존하고 있기 때문에, `CountingConnectionMaker` 오브젝트를 *DI*하는 것으로 설정을 바꾸면 `UserDao`가 DB 커넥션을 가져오려고 할 때마다 `CountingConnectionMaker`의 `makeConnection()` 메서드가 실행되고 카운터는 하나씩 증가할 것이다.
![[스크린샷 2024-04-20 오후 5.23.57.png|center|450]]

새로운 의존관계를 컨테이너가 사용할 설정정보를 이용해 만들어보자.
```java
@Configuration
public class CountingDaoFactory {
	@Bean
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}

	@Bean
	public ConnectionMaker connectionMaker() {
		return new CountingConnectionMaker(realConnectionMaker());
	}

	@Bean
	public ConnectionMaker realConnectionMaker() {
		return new DConnectionMaker();
	}
}
```

다음으로 실행 코드를 만든다.
```java
public class UserDaoConnectionCountingTest {
	public static void main(String[] args) thrwos ClassNotFoundException, SQLException {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(CountingDaoFactory.class);
		UserDao dao = context.getBean("userDao", UserDao.class);

		//
		// DAO 사용 코드
		//
		CountingConnectionMaker ccm = context.getBean("connectionMaker", CountingConnectionMaker.class);
		System.out.println("Connection counter : " + ccm.getCounter());
	}
}
```
주석 부분에서 DAO를 *DL* 방식으로 가져와 어떤 작업이든 여러 번 실행시킨 뒤에, DAO의 사용횟수를 출력되는 내용을 통해 확인할 수 있다.

DI의 장점은 관심사의 분리[^SoC]를 통해 얻어지는 높은 응집도에서 나온다. 모든 DAO가 직접 의존해서 사용할 `ConnectionMaker` 타입 오브젝트는 `connectionMaker()` 메서드에서 만든다. 따라서 `CountingConnectionMaker`의 의존관계를 추가하려면 `connectionMaker()` 메서드만 수정하면 된다. 또한 `CountingConnectionMaker`를 이용한 분석이 끝나면 다시 `CountingDaoFactory` 설정 클래스를 `DaoFactory`로 변경하거나 `connectionMaker()` 메서드를 수정하는 것만으로 DAO의 런타임 의존관계는 이전 상태로 복구된다.

##### 1.7.5 메서드를 이용한 의존관계 주입
의존관계를 주입할 때 반드시 생성자를 사용해야 하는 것은 아니다. 생성자가 아닌 일반 메서드를 사용할 수 있고, 생성자를 사용하는 방법보다 더 자주 사용된다.

생성자가 아닌 일반 메서드를 이용해 의존 오브젝트와의 관계를 주입하는 방법은 크게 두 가지가 있다.
- 수정자 메서드를 이용한 주입
  *수정자*[^setter] 메서드는 외부에서 오브젝트 내부의 멤버 변수를 변경하려는 용도로 주로 사용된다. 수정자 메서드는 매개변수로 전달된 값을 내부의 인스턴스 변수에 저장하는데, 부가적으로 입력 값에 대한 검증이나 그 밖의 작업을 수행할 수도 있다. 수정자 메서드는 외부로부터 제공받은 오브젝트 참조를 저장해뒀다가 내부의 메서드에서 사용하게 하는 DI 방식에서 활용할 수 있다.
- 일반 메서드를 이용한 주입
  생성자가 수정자 메서드보다 나은 점은 한 번에 여러 개의 매개변수를 받을 수 있다는 점이다. 하지만 매개변수의 개수가 많아지고 비슷한 타입이 여러 개라면 혼란스러울 수 있다. 임의의 초기화 메서드를 이용하는 DI는 적절한 개수의 매개변수를 가진 여러 개의 초기화 메서드를 만들 수도 있기 때문에 한 번에 필요한 모든 매개변수를 다 받아야 하는 생성자보다 유용할 수 있다.

`UserDao`에서 기존 생성자를 제거하고 수정자 메서드를 이용해 DI 하도록 만들어보자.
```java
public class UserDao {
	private ConnectionMaker connectionMaker;

	public void setConnectionMaker(ConnectionMaker connectionMaker) {
		this.connectionMaker = connectionMaker;
	}
	...
}
```

그리고 `DaoFactory`의 코드도 함께 수정해야 한다.
```java
@Bean
public UserDao userDao() {
	UserDao userDao = new UserDao();
	userDao.setConnectionMaker(connectionMaker());
	return userDao;
}
```

생성자, 수정자 메서드, 초기화 메서드 이외에도 다양한 의존관계 주입 방법을 지원하는데 이것은 나중에 다룰 것이다.

#TobySpring #Spring 

[^1]: Unified Modeling Language의 약자로, 소프트웨어 개발과정인 분석, 설계, 구현 단계의 각과정에서 필요한 모델을 명세화 할 수 있는 언어이다. 
[^2]: 언급된 서블릿은 Java를 사용하여 웹을 만들기 위한 기술로, **Spring**이 미리 만들어서 제공하기 때문에 직접 구현할 필요는 없다.
[^SoC]: Separation of Concerns