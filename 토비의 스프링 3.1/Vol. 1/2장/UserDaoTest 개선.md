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


#TobySpring #Spring 