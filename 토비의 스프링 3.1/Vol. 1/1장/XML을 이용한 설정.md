**Spring**은 `DaoFactory`와 같은 Java 클래스가 아닌 *XML*을 통해 의존관계 설정정보를 만들 수 있다.

XML은 단순한 텍스트 파일이기 때문에 다루기 쉽고, 컴파일 같은 별도의 빌드 작업이 없다. 그리고 환경이 달라져서 오브젝트의 관계가 바뀌는 경우에도 빠르게 변경사항을 반영할 수 있다.
##### 1.8.1 XML 설정
어플리케이션 컨텍스트는 XML에 담긴 DI 정보를 활용할 수 있다. DI 정보가 담긴 XML 파일은 `<beans>`를 루트 엘리먼트로 사용한다. `<beans>` 안에는 여러 개의 `<bean>`을 정의할 수 있다. Java 클래스에서 `@Configuration`을 `<beans>`, `@Bean`을 `<bean>`에 대응시켜서 생각하면 된다.

하나의 `@Bean` 메서드를 통해 얻을 수 있는 빈의 DI 정보는 다음과 같다.
- 빈의 이름 : `@Bean` 메서드 이름이 빈의 이름이고, `getBean()` 메서드에서 사용된다.
- 빈의 클래스 : 빈 오브젝트를 어떤 클래스를 이용해서 만들지 정의한다.
- 빈의 의존 오브젝트 : 빈의 생성자나 수정자 메서드를 통해 의존 오브젝트를 주입해준다. 의존 오브젝트도 하나의 빈이므로 이름이 있을 것이고, 그 이름에 해당하는 메서드를 호출해서 의존 오브젝트를 가져온다.

XML에서 `<bean>`을 사용해도 위의 정보를 정의할 수 있다. 의존하고 있는 오브젝트가 없는 경우에는 세 번째 정보는 생략할 수 있다. XML은 Java 코드처럼 유연하게 정의될 수 있는 것이 아니므로 태그와 어트리뷰트가 무엇인지 알아야 한다.
###### `connectionMaker()` 전환
`connectionMaker()` 메서드에 해당하는 빈에 대응되는 XML의 설정정보는 다음과 같이 `<bean>` 태그의 `id`와 `class` 어트리뷰트를 이용해 정의한다.
- `@Configuration` : `<beans>`
- `@Bean methodName()` : `<bean id="methodName"`
- `return new BeanClass();` : `class="a.b.c... BeanClass">`

`DaoFactory`의 `@Bean` 메서드에 담긴 정보를 1:1로 XML의 태그와 어트리뷰트로 전환해주면 되는데, `<bean>` 태그의 `class` 어트리뷰트에 지정하는 것은 Java 메서드에서, 메서드의 반환 타입이 아닌, 오브젝트를 만들 때 사용하는 클래스 이름이다. 그리고 클래스 이름에 패키지까지 모두 포함해야 한다.
```xml
<bean id="connectionMaker" class="springbook...DConnectionMaker" />
```
###### `userDao()` 전환
`userDao()`에서 수정자 메서드를 사용해 의존관계를 주입해주는 부분은 자바빈의 관례에 따라 프로퍼티가 된다. XML에서는 `<property>` 태그를 사용해 의존 오브젝트와의 관계를 정의한다. 태그의 `name` 어트리뷰트는 수정자 메서드 `setConnectionMaker()`에 대응되고, `ref` 어트리뷰트는 수정자 메서드를 통해 주입해줄 오브젝트의 빈 이름이다.
```xml
<bean id="userDao" class="springbook.dao.UserDao">
	<property name="connectionMaker" ref="connectionMaker" />
</bean>
```
###### XML의 의존관계 주입 정보
`<bean>` 태그를 이용해 `@Bean` 메서드를 모두 XML로 변환했다면 `<beans>`로 전환한 두 개의 `<bean>` 태그를 감싸주는 것으로 XML로의 전환 작업이 끝난다.
```xml
<beans>
	<bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />
	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
	</bean>
</beans>
```
프로퍼티의 이름이나 빈의 이름은 인터페이스 이름과 다르게 정해도 상관없지만, 그 이름을 참조하는 빈이 있다면 `ref` 어트리뷰트의 값도 함께 변경해줘야 한다.

같은 인터페이스를 구현한 의존 오브젝트를 여러 개 정의해두고 그 중에서 원하는 것을 골라서 주입하는 경우에도 각 빈의 이름을 독립적으로 만들어두고 `ref` 어트리뷰트를 이용해 주입받을 *빈*을 지정해주면 된다.
```xml
<beans>
	<bean id="localDBConnectionMaker" class="...LocalDBConnectionMaker" />
	<bean id="testDBConnectionMaker" class="...TestDBConnectionMaker" />
	<bean id="productionDBConnectionMaker" class="...ProductionDBConnectionMaker" />

	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="connectionMaker" ref="localDBConnectionMaker" />
	</bean>
</beans>
```

> *DTD*와 *스키마*
> XML 문서의 구조를 정의하는 방법으로 DTD와 스키마가 있다. **Spring**의 XML 설정파일은 두 가지 방식을 모두 지원한다.
> DTD를 사용할 경우에는 `<beans>` 엘리먼트 앞에 다음과 같은 DTD 선언을 넣어준다.
```xml
<!DOCTYPE beans PUBLIC "-//SPRING/DTD BEAN 2.0/EN" "http://www.springframeword.org/dtd/spring-beans-2.0.dtd">
```
> **Spring**은 DI를 위한 기본 태그인 `<beans>`와 `<bean>` 외에도 특별한 목적을 위해 별도의 태그를 사용할 수 있는 방법을 제공하는데, 이 태그들은 각각 별개의 스키마 파일에 정의되어 있고 독립적인 네임스페이스를 사용해야 한다. 따라서 이런 태그를 사용하려면 DTD 대신 네임스페이스가 지원되는 스키마를 사용해야 한다. `<beans>` 태그를 기본 네임스페이스로 하는 스키마 선언은 다음과 같다.
> 
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans 
	   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
```
##### 1.8.2 XML을 이용하는 어플리케이션 컨텍스트
어플리케이션 컨텍스트가 `DaoFactory` 대신 XML 설정정보를 활용하도록 만들어보자. 어플리케이션 컨텍스트가 사용하는 XML 설정파일은 관례를 따라 applicationContext.xml이라고 만들고 다음과 같이 작성해둔다.
```xml
<?xml version"1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans 
	   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
	<bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />

	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
	</bean>
</beans>
```

다음은 `UserDaoTest`의 어플리케이션 컨텍스트 생성 부분을 수정한다. 생성자에는 applicationContext.xml의 클래스패스를 넣는다. '/'는 생략해도 된다.
```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
```

`GenericXmlApplicationContext` 외에도 `ClassPathXmlApplicationContext`를 이용해 XML로부터 설정정보를 가져오는 어플리케이션 컨텍스트를 만들 수 있다. `GenericXmlApplicationContext`는 클래스패스를 포함한 다양한 소스로부터 설정파일을 읽어올 수 있는데, `ClassPathXmlApplicationContext`는 XML 파일을 클래스패스에서 가져올 때 사용할 수 있는 기능이 추가된 클래스이다.

예를 들면, `GenericXmlApplicationContext`가 특정 패키지 안에 있는 XML 설정파일을 사용하게 하려면 클래스패스 루트부터 파일의 위치를 지정해야 한다. `springbook.user.dao` 패키지에 daoContext.xml이라는 설정파일을 만들었다면 다음과 같은 코드를 작성해야 한다.
```java
new GenericApplicationContext("springbook/user/dao/daoContext.xml");
```

이것을 `ClassPathXmlApplicationContext`를 사용하면 XML 파일과 같은 클래스패스에 있는 클래스 오브젝트를 넘겨서 XML 파일의 위치를 `UserDao`의 위치를 통해 상대적으로 지정할 수 있다.
```java
new ClassPathXmlApplicationContext("daoContext.xml", UserDao.class);
```
##### 1.8.3 DataSource 인터페이스로 변환
###### DataSource 인터페이스 적용
`ConnectionMaker`는 DB 커넥션을 생성해주는 기능 하나만을 정의한 매우 단순한 인터페이스인데, 비슷한 용도로 사용할 수 있게 만들어진 `DataSource`라는 인터페이스가 이미 존재한다. 하지만 `DataSource`는 `getConnection()`이라는 DB 커넥션을 가져오는 기능 외에도 여러 개의 메서드를 가지고 있어서 인터페이스를 직접 구현하기는 부담스럽다.

일반적으로 `DataSource`를 구현해서 DB 커넥션을 제공하는 클래스를 만들 일은 거의 없다. 이미 다양한 방법으로 DB 연결과 풀링[^1] 기능을 갖춘 많은 `DataSource` 구현 클래스가 존재하고, 그것을 가져다 사용하면 충분하기 때문이다.

`DataSource` 인터페이스와 다양한 `DataSource` 구현 클래스를 사용할 수 있도록 `UserDao`를 리팩토링해보자.
```java
import javax.sql.DataSource;

public class UserDao {
	private DataSource dataSource;

	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}
	
	// DataSource의 getConnection() 메서드는 ClassNotFoundException을 던지지 않는다.
	public void add(User user) throws SQLException { 
		Connection c = dataSource.getConnection();
		...
	}
	...
}
```

`DataSource`의 구현 클래스로 `SimpleDriverDataSource`를 사용해보자. 이것은 DB 연결에 필요한 필수 정보를 제공받을 수 있도록  JDBC 드라이버 클래스, JDBC URL, 아이디, 비밀번호 등에 대한 수정자 메서드를 가지고 있다. 라이브러리는 `build.gradle` 파일에서 `dependencies` 섹션에 다음의 내용을 추가하면 된다.
```java
dependencies { 
	implementation 'org.springframework:spring-jdbc:3.0.7.RELEASE' 
}
```
###### Java 코드 설정 방식
`DaoFactory`에 있던 기존의 `connectionMaker()` 메서드를 `dataSource()` 메서드로 변경해보자.
```java
@Bean
public DataSource dataSource() {
	SimpleDriverDataSource dataSource = new SimpleDriverDataSource();

	dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
	dataSource.setUrl("jdbc:mysql://localhost/springbook");
	dataSource.setUsername("spring");
	dataSource.setPassword("book");

	return dataSource;
}
```

다음으로 `DaoFactory`의 `userDao()` 메서드를 `DataSource` 타입 객체를 주입받도록 변경해보자.
```java
@Bean
public UserDao userDao() {
	UserDao userDao = new UserDao();
	userDao.setDataSource(dataSource());
	return userDao;
}
```
###### XML 설정 방식
XML 파일에서는 `dataSource`라는 이름의 `<bean>`을 등록해야 한다.
```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource" />
```

Java 코드 설정 방식에서 JDBC 드라이버 클래스, JDBC URL, 아이디, 비밀번호를 수정자 메서드를 통해 설정했는데, XML에서는 `<property>` 태그로 설정해줄 수 있다. 이것은 다음 절에서 자세히 알아보자.
##### 1.8.4 프로퍼티 값의 주입
###### 값 주입
빈으로 등록될 클래스에 다른 빈 오브젝트의 레퍼런스(`ref`)가 아닌 단순 값을 주입해주는 것[^2]이기 때문에 `value` 어트리뷰트를 사용한다.

위의 Java 코드 설정 방식에서의 설정을 XML에서는 다음과 같이 작성해야 한다. 
```xml
<property name="driverClass" value="com.mysql.jdbc.Driver" />
<property name="url" value="jdbc:mysql://localhost/springbook" />
<property name="username" value="spring" />
<property name="password" value="book" />
```
###### value 값의 자동 변환
위의 설정에서 `driverClass`는 `java.lang.Class` 타입이고 나머지 설정값은 문자열로 구성되어 있다. 이것은 **Spring**에서 수정자 메서드의 매개변수 타입을 참고해서 적절한 타입으로 변환해준다. 가령, `setDriverClass()` 메서드의 매개변수 타입이 `Class`임을 확인하고 `"com.mysql.jdbc.Driver"`라는 문자열을 `com.mysql.jdbc.Driver.class` 오브젝트로 자동으로 변환해주는 것이다.

최종적으로는 다음과 같은 XML 설정 파일이 만들어진다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
   <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverSource">
	   <property name="driverClass" value="com.mysql.jdbc.Driver" />
	   <property name="url" value="jdbc:mysql://localhost/springbook" />
	   <property name="username" value="spring" />
	   <property name="password" value="book" />
   </bean>
   <bean id="userDao" class="springbook.user.dao.UserDao">
	   <property name="dataSource" ref="dataSource" />
   </bean>
</beans>
```

#Docs #TobySpring 

[^1]: 어플리케이션이 시작될 때 미리 정해진 수의 데이터베이스 연결을 생성하고 *풀*[^pool]에 저장하고, 그 연결을 사용할 때 풀에서 가져와 사용한 뒤 작업이 끝나면 그 연결을 반환해서 다른 요청에서 재사용할 수 있도록 한다.
[^2]: 텍스트나 단순 오브젝트 등을 수정자 메서드에 넣어주는 것을 '값을 주입한다'고 말한다. 성격은 조금 다르지만 일종의 DI라고 볼 수 있다.