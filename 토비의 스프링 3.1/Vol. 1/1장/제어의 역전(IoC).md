##### 1.4.1 오브젝트 팩토리
기존에 작성했던 `UserDaoTest`는 기능이 잘 작동하는지 테스트하는 책임과 더불어, `UserDao`와 `ConnectionMaker` 구현 클래스의 오브젝트를 만드는 책임과 그렇게 만들어진 두 개의 오브젝트가 연결돼서 사용될 수 있도록 관계를 맺어주는 책임을 갖고 있었다. 이 책임들도 분리해보자.
###### 팩토리
이번에 만들 클래스의 역할은 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는 것이다. 이런 일을 하는 오브젝트를 흔히 *팩토리*[^factory]라고 부른다.
```java
public class DaoFactory {
	public UserDao userDao() {
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao userDao = new UserDao(ConnectionMaker);
		return userDao;
	}
}
```
`DaoFactory`를 사용하면 `UserDao`가 어떻게 만들어지는지, 어떻게 초기화되어 있는지에 신경 쓰지 않고 팩토리로부터 오브젝트를 받아서 사용할 수 있다.
```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		UserDao dao = new DaoFactory.userDao();
		...
	}
}
```
###### 설계도로서의 팩토리
`UserDao`와 `ConnectionMaker`는 각각 어플리케이션의 핵심적인 데이터 로직과 기술 로직을 담당하고 있고, `DaoFactory`는 이런 어플리케이션의 오브젝트들을 구성하고 그 관계를 정의하는 책임을 맡고 있다. 여기서 `DaoFactory`는 컴포넌트의 의존관계에 대한 설계도와 같은 역할을 한다. 
![[스크린샷 2024-04-08 오전 3.00.54.png|center|450]]
##### 1.4.2 오브젝트 팩토리의 활용
다른 DAO(`AccountDao`, `MessageDao`)가 추가되고, 다른 DAO의 생성 기능이 `DaoFactory`에 추가되는 상황을 생각해보자.
```java
public class DaoFactory {
	public UserDao userDao() {
		return new UserDao(new DConnectionMaker());
	}

	public AccountDao accountDao() {
		return new AccountDao(new DConnectionMaker());
	}

	public MessageDao messageDao() {
		return new MessageDao(new DConnectionMaker());
	}
}
```
이렇게 오브젝트 생성 코드가 중복된다면 분리를 통해 `ConnectionMaker`의 구현 클래스를 결정하고 오브젝트를 만드는 코드를 별도의 메서드로 뽑을 수 있다.
```java
public class DaoFactory {
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}

	public AccountDao accountDao() {
		return new AccountDao(connectionMaker());
	}

	public MessageDao messageDao() {
		return new MessageDao(connectionMaker());
	}

	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker();
	}
}
```
##### 1.4.3 제어권의 이전을 통한 제어관계 역전
일반적으로 프로그램의 흐름은 프로그램이 시작되는 지점에서 다음에 사용할 오브젝트를 결정하고, 결정한 오브젝트를 생성하고, 만들어진 오브젝트에 있는 메서드를 호출하고, 그 오브젝트 메서드 안에서 다음에 사용할 것을 결정하고 호출하는 식의 작업이 반복된다. 이런 프로그램 구조에서 각 오브젝트는 프로그램 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여한다.

*제어의 역전*[^IoC, Inversion of Control]이란 이런 제어 흐름의 개념을 거꾸로 뒤집는 것이다. 제어의 역전에서는 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않는다. 당연히 생성하지도 않고, 자신도 어떻게 만들어지고 어디서 사용되는지를 알 수 없다. 모든 제어 권한을 자신이 아닌 다른 대상에게 위임하기 때문이다.

제어의 역전 개념은 서블릿이나 프레임워크 등 폭넓게 사용되고 있다. 지금까지 만든 `UserDao`와 `DaoFactory`에도 제어의 역전이 적용되어 있다. `UserDao`는 자신이 어떤 `ConnectionMaker` 구현 클래스를 만들고 사용할지를 결정할 권한을 `DaoFactory`에 넘겼으니 수동적인 존재가 되었다. 심지어는 자신도 `DaoFactory`에 의해 수동적으로 만들어진다. 자연스럽게 관심을 분리하고 책임을 나누고 유연하게 확장 가능한 구조로 만들기 위해 `DaoFactory`를 도입했던 과정이 IoC를 적용하는 작업이었다고 볼 수 있다. 
> 프레임워크와 라이브러리
> 프레임워크는 미리 만들어둔 반제품이지만, 확장해서 사용할 수 있도록 준비된 추상 라이브러리의 집합이 아니다. 라이브러리를 사용하는 어플리케이션 코드는 어플리케이션 흐름을 직접 제어하고, 필요한 기능이 있을 때 라이브러리를 사용한다. 반면에 프레임워크는 어플리케이션 코드가 프레임워크에 의해 사용된다. 일반적으로는 프레임워크 위에 개발한 클래스를 등록해두고, 프레임워크가 흐름을 주도하는 중에 개발자가 만든 어플리케이션 코드를 사용하도록 만드는 방식이다. 프레임워크에는 분명한 제어의 역전 개념이 적용되어 있어야 하고, 어플리케이션 코드는 프레임워크가 짜놓은 틀에서 수동적으로 동작해야 한다.

제어의 역전에서는 프레임워크 또는 컨테이너와 같이 어플리케이션 컴포넌트의 생성과 관계설정, 사용, 생명주기 관리 등을 관장하는 존재가 필요하다. `DaoFactory`는 오브젝트 수준의 가장 단순한 IoC 컨테이너 내지는 IoC 프레임워크라고 불릴 수 있다. **Spring**은 IoC를 모든 기능의 기초가 되는 기반기술로 삼고 있으며, IoC를 극한까지 적용하고 있는 프레임워크다.

#TobySpring #Spring 