관심사에 따라서 분리한 오브젝트들은 제각기 독특한 변화의 특징이 있다. 지금까지 예제로 보았던 코드는 데이터 액세스 로직을 만드는 방법과 DB 연결을 구현하는 방법에 대한 두 개의 관심을 상하위 클래스로 분리시켰다.

두 관심사는 변화의 이유와 시기, 주기 등이 다르다. `UserDao`는 JDBC API를 사용할지 DB 전용 API를 사용할지, 또는 어떤 테이블 이름과 필드 이름을 사용해 어떤 SQL을 만들 것인지 같은 관심을 가진 코드를 모아둔 것이다. 따라서 이런 관심사가 바뀌면 그때 변경이 일어난다. 하지만 이때도 DB 연결 방법이 그대로라면 DB 연결 확장 기능을 담은 `NUserDao`나 `DUserDao`의 코드는 변하지 않는다. 반대로 DB 연결 방식이나 DB 커넥션을 가져오는 방법이 바뀌면, 그때는 `UserDao` 코드는 그대로지만 서브 클래스의 코드는 바뀐다.
##### 1.3.1 클래스의 분리
이번에는 두 개의 관심사를 독립시키면서 동시에 손쉽게 확장할 수 있는 방법을 알아보자. 먼저 코드를 보자.
```java
public class UserDao {
	private SimpleConnectionMaker simpleConnectionMaker;

	public UserDao() {
		simpleConnectionMaker = new SimpleConnectionMaker();
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();
		...
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();
		...
	}
}
```
`SimpleConnectionMaker`라는 새로운 클래스를 만들고 DB 생성 기능을 그 안에 넣는다. 그리고 `UserDao`는 `SimpleConnectionMaker` 클래스 객체를 만들어두고, 이것을 `add()`, `get()` 메서드에서 사용한다. `UserDao`는 상속을 통한 방법을 쓰지 않으니 더 이상 `abstract`일 필요는 없다.

`SimpleConnectionMaker` 클래스는 다음과 같이 작성되어 있다.
```java
public class SimpleConnectionMaker {
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
		return c;
	}
}
```

하지만 `UserDao`의 코드가 `SimpleConnectionMaker`라는 특정 클래스에 종속되어 있기 때문에 DB 커넥션 기능을 확장해서 사용할 수 없다. 조금 더 구체적인 상황을 생각해보면, 첫 번째로는 `SimpleConnectionMaker` 클래스가 호출하는 메서드는 `makeNewConnection` 메서드인데, 다른 DB 커넥션을 사용하는 클래스를 사용하고자 한다면, 커넥션을 생성해서 사용하는 `add()`, `get()` 메서드 내부의 코드에도 수정이 필요하다. 두 번째는 DB 커넥션을 제공하는 클래스가 어떤 것인지를 `UserDao`가 구체적으로 알고 있어야 한다는 점이다. 즉, 다른 DB 커넥션을 사용하는 클래스도 `UserDao`에 포함될 것이다.

이런 문제의 근본적인 원인은 `UserDao`가 바뀔 수 있는 정보(DB 커넥션)를 가져오는 클래스에 대해 너무 많이 알고 있기 때문이다. 어떤 클래스가 쓰일지, 그 클래스에서 커넥션을 가져오는 메서드는 이름이 무엇인지까지 일일이 알고 있어야 한다. 위에서 작성한 코드는 `SimpleConnectionMaker`라는 특정 클래스와 그 코드에 종속적이기 때문에 DB 커넥션을 가져오는 방법을 자유롭게 확장하기가 힘들어졌다.
##### 1.3.2 인터페이스의 도입
위에서 발생한 문제를 해결하기 위해서 두 개의 클래스가 서로 긴밀하게 연결되어 있지 않도록 인터페이스를 사용해서 느슨한 연결고리를 만들어줄 것이다. 
```java
public interface ConnectionMaker {
	public Connection makeConnection() throws ClassNotFoundException, SQLException;
}

public class DConnectionMaker implements ConnectionMaker {
	...
	public Connection makeConnection() throws ClassNotFoundException ,SQLException {
		// D 사의 독자적인 방법으로 Connection을 생성하는 코드
	}
}
```
`UserDao`는 자신이 사용할 구체적인 클래스가 어떤 것인지 몰라도 `ConnectionMaker` 인터페이스를 통해 원하는 기능을 사용할 수 있다. 이렇게 하면 인터페이스의 메서드를 통해 알 수 있는 기능에만 관심을 가지면 되고, 그 기능을 어떻게 구현했는지에는 관심을 둘 필요가 없다. 이 인터페이스를 사용하는 `UserDao` 입장에서는 `makeConnection()` 메서드를 호출하면 `Connection` 타입의 오브젝트를 만들어서 돌려줄 것이라고 기대할 수 있다.

개선된 `UserDao`의 코드도 살펴보자.
```java
public class UserDao {
	private ConnectionMaker connectionMaker;

	public UserDao() {
		connectionMaker = new DConnectionMaker; // ..?
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeConnection();
		...
	}

	public void get(String id) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeConnection();
		...
	}
}
```
이제 `UserDao`의  `add()`, `get()` 메서드와 필드에는`ConnectionMaker` 인터페이스와 그것의 메서드인 `makeConnection()`만 사용하도록 했다. 그런데 생성자를 보면 `connectionMaker`에 `DConnectionMaker`를 할당하고 있다. 다시 말하면, 초기에 한 번 어떤 클래스의 오브젝트를 사용할지를 결정하는 생성자의 코드는 제거되지 않고 남아있는 것이다. 
##### 1.3.3 관계설정 책임의 분리
위에서 언급한, 생성자에 포함되어 있는 아래의 코드
```java
connectionMaker = new DConnectionMaker;
```
에는 분리되지 않은, 또 다른 관심사항이 존재함을 의미한다. 바로 `UserDao`가 어떤 `ConnectionMaker` 구현 클래스의 오브젝트를 이용하게 할지를 결정하는 것이다. 이 관심사 또한 분리해서 `UserDao`를 독립적으로 확장 가능한 클래스로 만들어야 한다.

제3의 관심사항인 `UserDao`와 `ConnectionMaker` 구현 클래스의 관계를 결정해주는 기능을 분리해서 두기에 적절한 곳은 `UserDao`의 클라이언트 오브젝트이다.[^1] 먼저 `UserDao`가 어떤 `ConnectionMaker`의 구현 클래스를 사용할지를 결정하도록 만들어보자. 즉, `UserDao` 오브젝트와 특정 클래스로부터 만들어진 `ConnectionMaker` 오브젝트 사이에 관계를 설정해주는 것이다.

오브젝트 사이의 관계는 런타임 시에 한쪽이 다른 오브젝트의 레퍼런스를 갖고 있는 방식으로 만들어진다. 예를 들면 위의 코드는 `DConnectionMaker`의 오브젝트의 레퍼런스를 `UserDao`의 `connectionMaker` 변수에 넣어서 사용하게 함으로써, 두 개의 오브젝트가 '사용'이라는 관계를 맺게 해준다.

위처럼 직접 생성자를 호출해서 직접 오브젝트를 만드는 방법도 있지만 외부에서 만들어준 것을 가져오는 방법도 있다. 외부에서 만든 오브젝트를 전달받으려면 메서드 매개변수나 생성자 매개변수를 이용하면 된다. 매개변수의 타입을 전달받을 오브젝트의 인터페이스로 선언해뒀다면, 매개변수로 전달되는 오브젝트의 클래스는 해당 인터페이스를 구현하기만 했다면 어떤 것이든지 상관없다.

`UserDao`의 생성자를 다음과 같이 수정해보자.
```java
public UserDao(ConnectionMaker connectionMaker) {
	this.connectionMaker = connectionMaker;
}
```
`UserDao` 오브젝트가 동작하려면 특정 클래스의 오브젝트와 관계를 맺어야 한다. 하지만 위의 코드를 통해 클래스 사이의 관계가 아닌, 오브젝트 사이에 동적인 관계가 만들어진다. 코드에서는 특정 클래스를 전혀 알지 못하더라도 해당 클래스가 구현한 인터페이스를 사용했다면, 그 클래스의 오브젝트를 인터페이스 타입으로 받아서 사용할 수 있다. 

그러면 `UserDao`의 클라이언트는 매개변수로 실제로 사용되는 오브젝트를 전달해주는 역할 또는 책임을 가진다. 클라이언트는 `UserDao`의 세부 전략이라고 볼 수 있는 `ConnectionMaker`의 구현 클래스를 선택하고, 선택한 클래스의 오브젝트를 생성해서 `UserDao`와 연결해줄 수 있다. 기존의 코드에서는 `UserDao`의 생성자에게 이 책임이 있었지만, 관심사가 함께 있었어서 확장성을 떨어뜨린 것이다.

`UserDao`의 클라이언트로 사용할 클래스를 하나 만들어보자.
```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ConnectionMaker connectionMaker = new DConnectionMaker();

		UserDao dao = new UserDao(connectionMaker);
		...
	}
}
```
`UserDaoTest`는 `UserDao`와 `ConnectionMaker` 구현 클래스와의 런타임 오브젝트 의존 관계를 설정하는 책임을 담당해야 한다. 그래서 특정 `ConnectionMaker` 구현 클래스의 오브젝트를 만들고, `UserDao` 생성자 매개변수에 넣어 두 개의 오브젝트를 연결해준다.

이제 `UserDao`의 코드는 전혀 변경하지 않고 DB 연결 기능을 확장해서 사용할 수 있게 되었다!
![[스크린샷 2024-04-04 오후 5.32.21.png|center|450]]
##### 1.3.4 원칙과 패턴
###### 개방-폐쇄 원칙
*개방-폐쇄 원칙*[^OCP, Open-Closed Principle]을 간단히 정의하면, '클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다'라고 할 수 있다. 위의 그림은 *개방-폐쇄 원칙*을 잘 보여준다. 인터페이스를 통해 확장에 대해서는 열려 있고, 인터페이스를 이용하는 클래스는 자신의 변화가 불필요하게 일어나지 않도록 폐쇄되어 있다.
> 객체지향 설계 원칙(SOLID)
> SOLID는 다음의 5가지 원칙을 의미한다.
> - *단일 책임 원칙*[^SRP, Single Responsibility Principle]
> - *개방-폐쇄 원칙*[^OCP, Open-Closed Principle]
> - *리스코프 치환 원칙*[^LSP, Liskov Substitution Principle]
> - *인터페이스 분리 원칙*[^ISP, Interface Segregation Principle]
> - *의존관계 역전 원칙*[^DIP, Dependency Inversion Principle]
###### 높은 응집도와 낮은 결합도
개방-폐쇄 원칙은 *높은 응집도*[^high coherence]와 *낮은 결합도*[^low coupling]라는 소프트웨어 개발의 고전적인 원리로도 설명이 가능하다.
- *높은 응집도*
  *응집도가 높다*는 것은 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 뜻이다. 불필요하거나 직접 관련이 없는 외부의 관심과 책임이 얽혀 있지 않으며, 하나의 공통 관심사는 한 클래스에 모여 있다. 높은 응집도는 클래스 수준뿐 아니라, 패키지, 컴포넌트, 모듈에 이르기까지 그 대상의 크기가 달라도 동일한 원리로 적용될 수 있다.
  변경이 일어날 때 모듈의 많은 부분이 함께 바뀐다면 응집도가 높다고 할 수 있다. 여러 관심사와 책임이 얽혀 있는 복잡한 코드에서는 변경이 필요한 부분을 찾아내는 것도 번거롭고, 변경한 것이 다른 기능에 영향을 줘서 오류를 발생시키지 않는지도 확인해야 한다.
- *낮은 결합도*
  *결합도가 낮다*는 것은 책임과 관심사다 다른 오브젝트 또는 모듈과 느슨하게 연결된 형태를 말한다. 느슨한 연결은 관계를 유지하는 데 꼭 필요한 최소한의 방법만 간접적인 형태로 제공하고, 나머지는 서로 독립적이고 알 필요도 없게 만들어주는 것이다.
  여기서 *결합도* 란 '하나의 오브젝트가 변경이 일어날 때에 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도'라고 설명할 수 있다. 그런 관점에서 결합도가 낮다는 것은 변경이 발생할 때 다른 모듈과 객체로 변경에 대한 요구가 전파되지 않는 상태를 말한다. 

지금까지 개선된 코드를 위의 관점으로 다시 보면, `UserDao` 클래스는 그 자체로 자신의 책임에 대한 응집도가 높다. 사용자의 데이터를 처리하는 기능이 클래스 안에 잘 모여 있다. `ConnectionMaker` 또한 자신의 기능에 충실하도록 독립돼서 자신의 책임만을 담당한다.

동시에 `UserDao`와 `ConnectionMaker`의 관계는 인터페이스를 통해 매우 느슨하게 연결되어 있다. 꼭 필요한 관계만 `ConnectionMaker`라는 인터페이스를 통해 낮은 결합도로 최소한으로 연결되어 있다.
###### 전략 패턴
*전략 패턴*[^Strategy Pattern]은 자신의 기능 *맥락*[^context]에서(`UserDao`), 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고(`ConnectionMaker`), 이를 구현한 구체적인 알고리즘 클래스를(`NConnectionMaker`, `DConnectionMaker`) 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴이다. 그리고 컨텍스트를 사용하는 클라이언트(`UserDaoTest`)는 컨텍스트가 사용할 전략을 컨텍스트의 생성자 등을 통해 제공해주는 것이 일반적이다.

#TobySpring #Spring 

[^1]: 두 개의 오브젝트가 있고 한 오브젝트가 다른 오브젝트의 기능을 사용한다면, 사용되는 오브젝트를 서비스, 사용하는 오브젝트를 클라이언트라고 부른다.