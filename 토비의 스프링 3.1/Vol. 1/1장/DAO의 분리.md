##### 1.2.1 관심사의 분리
사용자의 비즈니스 프로세스와 그에 따른 요구사항은 끊임없이 바뀌고 발전한다. 어플리케이션이 기반을 두고 있는 기술도 시간이 지남에 따라 바뀌고, 운영되는 환경도 변화한다. 그로 인해 오브젝트에 대한 설계와 이를 구현한 코드 또한 변한다.

개발자가 객체를 설계할 때 가장 염두에 둬야 할 사항은 미래의 변화를 어떻게 대비할 것인가이다. 변화는 먼 미래에만 일어나는 것이 아니다. 며칠 내에, 때로는 몇 시간 후에 변화에 대한 요구가 갑자기 발생할 수 있다. 객체지향 설계와 프로그래밍이 초기에 좀 더 많은, 번거로운 작업을 요구하는 이유는 변화에 효과적으로 대처할 수 있다는 기술적인 특징 때문이다. 좀 더 자세히 말하면, 객체지향 기술이 만들어내는 가상의 추상세계 자체를 효과적으로 구성하고, 이것을 자유롭고 편리하게 변경, 발전, 확장시킬 수 있다. 이를 통해 개발자는 미래의 변화에 대비할 수 있다.

미래의 변화에 대비하는 가장 좋은 대책은 변화의 폭을 최소한으로 줄여주는 것이다. 변경이 일어날 때 필요한 작업을 최소화하고, 그 변경이 다른 곳에 문제를 일으키지 않도록 *분리*와 *확장*을 고려한 설계가 그 방법이 될 수 있다.

먼저 *분리*에 대해 생각해보자.

모든 변경과 발전은 한 번에 한 가지 관심사항에 집중해서 일어난다. 하지만 그에 따른 작업은 한 곳에 집중되지 않는 경우가 많다. 그러므로 우리는 한 가지 관심이 한 군데에 집중되게 해야 한다. 관심이 같은 것끼리는 모으고, 관심이 다른 것은 따로 떨어져 있게 하는 것이다.

프로그래밍의 기초 개념 중에 *관심사의 분리*[^Separation of Concerns]라는 게 있다. 객체지향의 관점에서는, 관심이 같은 것끼리는 하나의 객체 안으로 또는 친한 객체로 모이게 하고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것이다.

##### 1.2.2 커넥션 만들기의 추출
`UserDao` 클래스에 구현된 메서드를 다시 살펴보자. `add()` 메서드 하나에서만 적어도 세 가지 관심사항을 발견할 수 있다.
- DB와 연결을 위한 커넥션을 가져오는 것
- 사용자 등록을 위해 DB에 보낼 SQL 문장을 담을 `Statement`를 만들고 실행하는 것
- 작업이 끝나면 사용한 리소스인 `Statement`와 `Connection` 오브젝트를 닫아서 공유 리소스를 시스템에 돌려주는 것

첫 번째 관심사인 DB 연결을 위한 `Connection` 오브젝트를 가져오는 코드는 `add()` 메서드 뿐만 아니라 `get()` 메서드에도 중복되어 있다. 이렇게 하나의 관심사가 중복되어 있고 여기저기 흩어져 있어서 다른 관심의 대상과 얽혀 있으면 변경이 일어날 때 수정할 코드가 많아진다.
###### 중복 코드의 메서드 추출
중복된 DB 연결 코드를 `getConnection()`이라는 이름의 독립적인 메서드로 만들고, `UserDao`의 메서드에서는 `getConnection()` 메서드를 호출해서 DB 커넥션을 가져오게 만든다.
```java
public void add(User user) throws ClassNotFoundException, SQLException {
	Connection c = getConnection();
	...
}

public void get(User user) throws ClassNotFoundException, SQLException {
	Connection c = getConnection();
	...
}

private Connection getConnection() throws ClassNotFoundException, SQLException {
	Class.forName("com.mysql.jdbc.Driver");
	Connection c = DriverManager.getConnection(
			"jdbc:mysql://localhost/springbook", "spring", "book");
	return c;
}
```
이렇게 하면 DB 연결과 관련된 부분에 변경이 일어났을 경우, 예를 들어 DB 종류와 접속 방법이 바뀌어서 드라이버 클래스의 URL이 바뀌었다거나, 로그인 정보가 변경돼도 `getConnection()` 메서드만 수정하면 된다.

##### 변경사항에 대한 검증 : 리팩토링과 테스트
코드를 수정한 후에는 기능에 문제가 없다는 것이 보장되지 않기 때문에 다시 검증해야 한다. 테이블의 기본키인 `id` 값이 중복되는 것을 피하기 위해 `User` 테이블의 사용자 정보를 모두 삭제한 뒤, 다시 `main()` 메서드를 실행해서 처음과 같은 결과가 화면에 출력되는지를 확인해보면 된다.

위의 코드 수정은 `UserDao`의 기능에는 아무런 변화를 주지 않았다. 여러 메서드에 중복돼서 등장하는 특정 관심사항이 담긴 코드를 별도의 메서드로 분리해낸 것이다. 이 작업은 기능에는 영향을 주지 않으면서 코드의 구조만 변경한다. 이런 작업을 *리팩토링*[^refactoring]이라고 한다. 그리고 공통의 기능을 담당하는 메서드로 중복된 코드를 뽑아내는 것을 리팩토링에서는 *메서드 추출*[^extract method] 기법이라고 부른다.

> 📒 리팩토링
> 기존의 코드를 외부의 동작방식에는 변화 없이 내부 구조를 변경해서 재구성하는 작업 또는 기술을 말한다. 리팩토링을 하면 코드 내부의 설계가 개선되어 코드를 이해하기가 더 편해지고, 변화에 효율적으로 대응할 수 있다. 이것을 통해서 생산성을 높이고, 코드의 품질을 높이고, 유지보수하기 용이해지며, 견고하면서도 유연한 제품을 개발할 수 있다.
##### 1.2.3 DB 커넥션 만들기의 독립
`UserDao`가 여러 종류의 DB를 사용할 수 있도록 하려면 어떻게 코드를 수정하면 좋을지 알아보자. 
###### 상속을 통한 확장
기존에 만들었던 `UserDao`에서 메서드의 구현 코드를 제거하고 `getConnection()` 메서드를 추상 메서드로 만들어 놓으면 `UserDao` 클래스를 상속해서 추상 메서드로 선언했던 `getConnection()` 메서드를 원하는 방식으로 구현할 수 있다.

기존에는 같은 클래스에 다른 메서드로 분리됐던 DB 커넥션 연결이라는 관심을 상속을 통해 서브 클래스로 분리하는 것이다. 수정한 코드를 보자.
```java
public abstract class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}
	
	public void get(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}

	public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}

public class NUserDao extends UserDao {
	public Connection getConnection() throws ClassNotFoundException, SQLException {
		// N 사 DB connection 생성 코드
	}
}

public class DUserDao extends UserDao {
	public Connection getConnection() throws ClassNotFoundException, SQLException {
		// D 사 DB connection 생성 코드
	}
}
```
코드를 보면, 현재 DAO의 핵심 기능인 어떻게 데이터를 등록하고 가져올 것인가라는 관심을 담당하는 `UserDao`와 DB 연결 방법은 어떻게 할 것인가라는 관심을 담고 있는 `NUserDao`, `DUserDao`가 클래스 레벨로 구분이 되고 있다. 이제는 `UserDao`의 코드는 한 줄도 수정할 필요 없이 DB 연결 기능을 새롭게 정의한 클래스를 만들 수 있다.

이렇게 추상 클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메서드나 오버라이딩이 가능한 `protected` 메서드 등으로 만든 뒤 서브 클래스에서 이런 메서드를 필요에 맞게 구현해서 사용하도록 하는 방법을 디자인 패턴에서 *템플릿 메서드 패턴*[^template method pattern]이라고 한다. 

그리고 `UserDao`의 서브 클래스의 `getConnection` 메서드는 어떤 `Connection` 클래스의 오브젝트를 어떻게 생성할 것인지를 결정하는 방법이라고 볼 수 있는데, 서브 클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 *팩토리 메서드 패턴*[^factory method pattern]이라고 한다.

<p align="center">
	<img width="460" src="../../../images/스크린샷 2024-04-01 오전 3.36.07.png">
</p>

![[스크린샷 2024-04-01 오전 3.36.07.png|center]]

이렇게 템플릿 메서드 패턴 또는 팩토리 메서드 패턴으로 관심사항이 다른 코드를 분리하고, 서로 독립적으로 변경 또는 확장할 수 있도록 만드는 것은 간단하면서도 효과적인 방법이다. 하지만 이 방법은 상속을 사용했다는 단점이 있다. 만약 이미 `UserDao`가 다른 목적을 위해 상속을 사용하고 있다면, Java는 클래스의 다중 상속을 허용하지 않기 때문에 위의 방법을 적용하기 어렵다. 

또한, 상속을 통한 상하위 클래스의 관계는 생각보다 밀접하다. 다시 말하면, 상속관계는 두 가지 다른 관심사에 대해 긴밀한 결합을 허용한다. 서브 클래스는 슈퍼 클래스의 기능을 직접 사용할 수 있기 때문에, 슈퍼 클래스 내부의 변경이 있을 때 모든 서브 클래스를 함께 수정하거나 다시 개발해야 할 수도 있다. 반대로 그런 변화에 따른 불편을 주지 않기 위해 슈퍼 클래스가 더 이상 변화하지 않도록 제약을 가해야 할 수도 있다.

확장된 기능인 DB 커넥션을 생성하는 코드를 다른 DAO 클래스에 적용할 수 없다는 것도 큰 단점이다. `UserDao` 외의 DAO 클래스가 생기고, 그 클래스에서도 DB 커넥션이 필요하다면 상속을 통해서 만들어진 `getConnection()`의 구현 코드가 DAO 클래스마다 중복돼서 나타날 것이기 때문이다.

------
#TobySpring #Spring 