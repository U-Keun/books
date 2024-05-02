이전에 언급했던 구현 세부 사항에 대한 테스트를 피하는 것은 퍼블릭 API만을 사용하여 테스트해야 한다는 것을 의미한다. 퍼블릭 API에 초점을 맞추면 세부 사항이 아닌 코드 사용자가 궁극적으로 신경 쓸 동작에 집중할 수밖에 없게 된다. 하지만 테스트할 코드가 복잡하다면 상황이 어려워질 수 있다.
##### 10.3.1 중요한 동작이 퍼블릭 API 외부에 있을 수 있다
테스트 대상 코드는 수많은 다른 코드에 의존하는 경우가 많다. 의존하는 코드로부터 외부 입력이 제공되거나 테스트 대상 코드가 의존하는 코드에 부수 효과를 일으킨다면 테스트의 의미가 미세하게 달라질 수 있다.

추상화 계층 관점에서는 한 코드가 다른 코드에 대해 알아야 할 모든 것이 퍼블릭 API로 제공되기 때문에 그 외의 모든 것은 구현 세부 사항이라고 볼 수 있다. 하지만 테스트에 관해서는 퍼블릭 API로 제공되지 않는 것 중에서도 테스트 코드가 알아야 할 다른 사항들이 있을 수 있다. 예제를 보자.
```java
class AddressBook {
	private final ServerEndPoint server;
	private final Map<Integer, String> emailAddressCache;
	...
	Optionta<String> lookupEmailAddress(Integer userId) { // 퍼블릭 API
		Optional<String> cachedEmail = emailAddressCache.get(userId);
		if (cachedEmail != null) {
			return cachedEmail;
		}
		return fetchAndCacheEmailAddress(userId);
	}

	private Optional<String> fetchAndCacheEmailAddress(Integer userId) {
		Optional<String> fetchedEmail = server.fetchEmailAddress(userId);
		if (fetchedEmail != null) {
			emailAddressCache.put(userId, fetchedEmail);
		}
		return fetchedEmail;
	}
}
```
이 코드에서 퍼블릭 API에 해당되는 것은 `lookupEmailAddress` 메서드 뿐이지만, `ServerEndPont`를 설정하지 않으면 테스트할 수 없다. 그리고 같은 `userId` 값으로 여러 번 호출이 되었을 때, 서버에 대한 호출이 일어나지 않는지도 테스트해봐야 한다.

가능하면 퍼블릭 API를 사용하여 코드의 동작을 테스트해야 하지만, 그것만으로 모든 동작이나 시나리오를 테스트할 수 없는 경우가 있다.
- 서버와 상호작용하는 코드
- 데이터베이스에 값을 저장하거나 읽는 코드

궁극적으로 중요한 것은 코드의 모든 중요한 동작을 제대로 테스트하는 것이고, 퍼블릭 API라고 생각하는 것만으로는 이것을 할 수 없는 경우가 있다.

#GoodCodeBadCode 