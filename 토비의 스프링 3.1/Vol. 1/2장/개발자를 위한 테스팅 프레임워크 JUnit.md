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
- `deleteAll()`
	```java
	public void deleteAll() throws SQLException {
		Connectionc = dataSource.getConnection();
	
		PreparedStatement ps = c.prepareStatement("delete from users");
	
		ps.executeUpdate();
	
		ps.close();
		c.close();
	}
	```
- `getCount()`
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
추가된 기능에 대한 테스트를 만들어보자. 조금 애매한 부분이 있다면, 

#TobySpring #Spring 