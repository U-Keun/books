코드를 보다 쉽게 읽을 수 있도록 하기 위한 일반적이고 효과적인 기법의 토대를 소개한다.
#### 5.1 서술형 명칭 사용
클래스, 함수, 변수와 같은 것들을 고유하게 식별하기 위해 이름이 필요하다. 이름을 붙이는 것에 그것이 스스로 설명되는 방식으로 언급함으로써 읽기 쉬운 코드를 작성하는 토대를 마련할 수 있다.
##### 5.1.1 서술적이지 않은 이름은 코드를 읽기 어렵게 만든다.
극단적으로 클래스의 이름을 `C` 라고 지으면, 그 클래스에 포함된 모든 코드를 읽어야 무슨 일을 하는지 파악할 수 있다. 다시 말하면, 어떤 일을 할 지 예상할 수 없다.
##### 5.1.2 주석문으로 서술적인 이름을 대체할 수 없다.
주석문과 문서를 추가하더라도, 코드를 모두 읽어야 하는 것은 변하지 않고, 코드 뿐만 아니라 주석문과 문서도 유지 보수해야 한다.
##### 5.1.3 해결책 : 서술적인 이름 짓기
```
class Team {
	Set<String> playerNames = new Set();
	Int score = 0;
	...
	Boolean containsPlayer(Strign playerName) {
		return playerNames.contains(playerName);
	}

	Int getScore() {
		return score;
	}
}

Int? getTeamScoreForPlayer(List<Team> teams, String playerName) {
	for (Team team in teams) {
		if (team.containsPlayer(playerName)) {
			return team.getScore();
		}
	}
	return null;
}
```
위의 코드 예제를 보면 변수, 함수 및 클래스를 별도로 설명할 필요가 없고, 메서드 호출이 무엇을 의미하는지, 무슨 값을 반환하는지 분명하게 예상할 수 있다.
#### 5.2 주석문의 적절한 사용
주석문이나 문서화는 코드가 무엇을 하는지, 왜 그 일을 하는지를 설명해야 하고, 그 외에도 사용 지침 같은 기타 정보를 제공해야 한다.

클래스와 같이 큰 단위의 코드가 무엇을 하는지 요약하는 높은 수준에서의 주석문은 유용하다. 하지만 하위 수준에서 한 줄 한 줄 코드가 무엇을 하는지 설명하는 주석문은 가독성을 해칠 수 있다.
##### 5.2.1 중복된 주석문은 유해할 수 있다
코드가 수행하는 작업이 코드 자체로 설명이 된다면, 그 코드에 주석문을 작성하는 것은 의미가 없다. 그리고 개발자는 주석문도 유지보수해야 하기 때문에 중복된 주석문은 코드를 지저분하게 만들 뿐더러 일을 더 만들 뿐이다.
##### 5.2.2 주석문으로 가독성 높은 코드를 대체할 수 없다
다음의 두 코드를 비교해보자.
``` 
// 예제 5.5 주석문이 있는 이해하기 어려운 코드
String generateId(String[] data) {
	// data[0]는 유저의 이름이고 data[1]은 성이다.
	// "{이름}.{성}"의 형태로 ID를 생성한다.
	return data[0] + "." + data[1];
}

// 예제 5.6 가독성이 더 좋아진 코드
String generateId(String[] data) {
	return firstName(data) + "." + lastName(data);
}
// 헬퍼 함수들
String firstName(String[] data) {
	return data[0];
}

String lastName(String[] data) {
	return data[1];
}
```
코드 자체로 설명이 되도록 코드를 작성하면 유지 및 관리의 과도한 비용을 줄이고, 주석문의 내용이 업데이트되지 않거나 잘못될 가능성을 없애주기 때문에 주석문을 사용하는 것보다 선호될 때가 많다.
##### 5.2.3 주석문은 코드의 이유를 설명하는 데 유용하다.
코드가 그 일을 **왜** 수행하는지는 코드 자체로는 설명하기 어렵다. 다른 개발자가 알 수 없는 배경 상황이나 지식과 관련있을 수 있기 때문이다. 그러므로 제품 또는 비즈니스 의사 결정이나 명확하지 않은 버그에 대한 해결책, 또는 의존하는 코드의 예상을 벗어나는 동작에 대처해야 하는 경우 등에는 주석문을 사용해 코드가 존재하는 이유를 설명하면 좋다.
##### 5.2.4 주석문은 유용한 상위 수준의 요약 정보를 제공할 수 있다.
```
// 예제 5.8 클래스에 대한 상위 수준의 문서화
/**
* 스트리밍 서비스의 유저에 대한 자세한 사항을 갖는다.
* 
* 이 클래스는 데이터 베이스에 직접 연결하지 않는다. 대신 메모리에 저장된 값으로 생성된다.
* 따라서 이 클래스가 생성된 이후에 데이터베이스에서 이뤄진 변경 사항을 반영하지 않을 수 있다.
*/
class User {
	...
}
```
주석과 문서화는 코드만으로는 전달할 수 없는 세부 사항을 설명하거나 코드가 큰 단위에서 하는 일을 요약하는 데 유용하다. 기억해야 할 것은, 이런 주석문과 문서도 유지 및 보수가 필요하고, 내용이 제때 업데이트되지 않으면 코드와 맞지 않게 되고 코드가 지저분해질 수 있다.
`javadoc`을 적절하게 사용하는 것을 연습하면 좋을 것 같다.
#### 5.3 코드 줄 수를 고정하지 말라
일반적으로 코드베이스의 코드 줄 수는 적을수록 좋지만, 반드시 지켜야 할 엄격한 규칙은 아니다. 우리가 정말로 신경 써야 하는 것은 코드가 이해하기 쉽고, 오해하기 어렵고, 실수로 작동이 안 되게 만들기 어렵도록 작성하는 것을 확실하게 하는 것이다.
##### 5.3.1 간결하지만 이해하기 어려운 코드는 피하라
```
Boolean isIdValid(UInt16 id) {
	return countSetBits(id & 0x7FFF) % 2 == ((id & 0x8000) >> 15);
}
```
이 코드는 패리티 비트를 검사하는데, 이는 데이터를 전송할 때 사용되는 오류 감지를 위한 값이다. 이 코드를 이해하기 위해서는 알아야 할 배경 지식이 많다. 코드가 간결하긴 하지만, 이 코드가 무엇을 하는지 이해하려고 많은 시간을 낭비할 수 있고, 이 코드가 전제하고 있는 명확하지 않고 문서화되지 않은 많은 가정으로 인해 코드 수정에 상당히 취약하고, 수정된 코드가 제대로 동작하지 않을 수 있다.
##### 5.3.2 해결책 : 더 많은 줄이 필요하더라도 가독성 높은 코드를 작성하라
위의 코드를 가독성이 높도록 수정해보자.
```
Boolean isIdValid(UInt16 id) {
	return extractEncodedParity(id) == calculateParity(getIdValue(id));
}

private const UInt16 PARITY_BIT_INDEX = 15;
private const UInt16 PARITY_BIT_MASK = (1 << PARITI_BIT_INDEX);
private const UInt16 VALUE_BIT_MASK = ~PARITY_BIT_MASK;

private UInt16 getIdValue(UInt16 id) {
	return id & VALUE_BIT_MASK;
}

private UInt16 extractEncodedParity(UInt16 id) {
	return (id & PARITY_BIT_MASK) >> PARITY_BIT_INDEX;
}

// 패리티 비트는 1인 비트의 수가 짝수이면 0이고, 홀수이면 1이다.
private UInt16 calculateParity(UInt16 value) {
	return countSetBits(value) % 2;
}
```
코드의 줄 수가 많다는 것은 기존 코드를 재사용하지 않거나 무언가를 필요 이상으로 복잡하게 만들고 있다는 경고 신호가 될 수 있지만, 더 중요한 것은 코드가 이해하기 쉬워야 하고, 어떤 상황에서도 잘 동작하고, 문제가 되는 동작을 할 가능성은 없는지 확인하는 것이다.
#### 5.4 일관된 코딩 스타일을 고수하라
##### 5.4.1 일관적이지 않은 코딩 스타일은 혼동을 일으킬 수 있다
코드 작성 시 클래스 이름은 일반적으로 첫 글자를 대문자로 시작하는 **파스칼 케이스**[^PascalCase] 로 작성되는 반면 변수 이름은 첫 글자를 소문자로 시작하는 **캐멀 케이스**[^camelCase] 로 작성된다. 이렇게 작성함으로서 전체 클래스 정의를 확인하지 않고도 어떤 것이 클래스인지 변수인지 확실하게 구분할 수 있는데, 이런 암묵적인 규칙을 지키지 않으면 다른 개발자가 그 코드를 봤을 때 오해할 수 있다.
##### 5.4.2 해결책 : 스타일 가이드를 채택하고 따르라
클래스 이름, 변수명 관련 코딩 스타일 이외에도 언어의 특정 기능을 사용하거나, 코드 들여쓰기, 패키지 및 디렉터리 구조화, 코드 문서화 방법 등 대부분의 조직과 팀들은 개발자들이 따라야 할 코딩 스타일 가이드를 이미 가지고 있다. 팀이 요구하는 스타일 가이드를 파악한 후 그대로 따르면 된다.
#### 5.5 깊이 중첩된 코드를 피하라
##### 5.5.1 깊이 중첩된 코드는 읽기 어려울 수 있다
여러 겹으로 중첩된 몇 개의 `if` 문을 포함하고 있는 코드는 상당히 읽기 어렵다. 즉 가독성이 떨어진다. 그러므로 중첩을 최소화하도록 코드를 구성하는 것이 바람직하다
##### 5.5.2 해결책 : 중첩을 최소화하기 위한 구조 변경
중첩된 모든 블록에 반환문(`return`)이 있을 때, 중첩을 피하기 위해 `else` 블록을 없애는 등의 논리를 재배치하는 것이 일반적인 방법이다. 그러나 중첩된 블록에 반환문이 있다면 그것은 대개 함수가 너무 많은 일을 하고 있다는 신호다.
##### 5.5.3 중첩은 너무 많은 일을 한 결과물이다
```
SentConfirmation? sendOwnerALetter(Vehicle vehicle, Letter letter) {
	Address? ownersAddress = null;
	if (vehicle.hasBeenScraped()) {
		ownersAddress = SCRAPYARD_ADDRESS;
	} else {
		Purchase? mostRecentPurchase = vehicle.getMostRecentPurchase();
		if (mostRecentPurchase == null) {
			ownersAddress = SHOWROOM_ADDRESS;
		} else {
			Buyer? buyer = mostRecentPurchase.getBuyer();
			if (buyer != null) {
				ownersAddress = buyer.getAddress();
			}
		}
	}
	if (ownersAddress == null) {
		return null;
	}
	return sendLetter(ownersAddress, letter);
}
```
위의 메서드는 주소를 찾기 위한 로직과 편지를 보내는 로직이 하나의 함수에 다 포함되어 있다. 이 함수를 더 작은 함수로 나누어보자.
##### 5.5.4 해결책 : 더 작은 함수로 분리
```
SentConfirmation? sendOwnerALetter(Vehicle vehicle, Letter letter) {
	Address? ownersAddress = getOwnersAddress(vehicle);
	if (ownersAddress == null) {
		return null;
	}
	return sendLetter(ownersAddress, letter);
}
Address? getOwnersAddress(Vehicle vehicle) {
	if (vehicle.hasBeenScraped()) {
		return SCRAPYARD_ADDRESS;
	}
	Purchase? mostRecentPurchase = vehicle.getMostRecentPurchase();
	if (mostRecentPurchase == null) {
		return SHOWROOM_ADDRESS;
	}
	Buyer? buyer = mostRecentPurchase.getBuyer();
	if (buyer == null) {
		return null;
	}
	return buyer.getAddress();
}
```
하나의 함수가 너무 많은 일을 하면 추상화 계층이 나빠진다는 점을 살펴봤었다. 따라서 중첩이 없더라도 많은 일을 한꺼번에 하는 함수를 더 작은 함수로 나누는 것은 여전히 바람직하다.
#### 5.6 함수 호출도 가독성이 있어야 한다
메서드의 이름이 잘 명명되었더라도 매개변수가 무엇을 위한 것이고, 무슨 역할을 하는지 명확하지 않다면 함수 호출 자체가 이해되지 않을 수 있다.
##### 5.6.1 매개변수는 이해하기 어려울 수 있다
```
sendMessage("hello", 1, true);
```
위와 같은 메서드를 호출할 때는 매개변수가 무엇을 의미하는지 알려면 함수 정의를 살펴봐야 한다.
##### 5.6.2 해결책 : 명명된 매개변수 사용
Java에서는 사용할 수 없는 방법인 것 같다.
##### 5.6.3 해결책 : 서술적 유형 사용
```
void sendMessage(String message, Int priority, Boolean allowRetry) {
	...
} // 정수와 불리언은 그 자체로는 서술적이지 않다.
```
클래스 또는 열거형 클래스를 사용해서 매개변수가 나타내는 바를 설명하도록 코드를 작성할 수 있다.
```
class MessagePriority {
	...
	MessagePriority(Int priority) {...}
	...
}
enum RetryPolicy {
	ALLOW_RETRY,
	DISALLOW_RETRY
}
void sendMessage(String message, MessagePriority, RetryPolicy retryPolicy) {
	...
}
```
위와 같이 정의하면 `sendMessage` 메서드를 더 이해하기 쉽게 작성할 수 있다.
```
sendMessage("hello", new MessagePriority(1), RetryPolicy.ALLOW_RETRY);
```
##### 5.6.4 때로는 훌륭한 해결책이 없다
하지만 무분별하게 클래스로 값을 표현하도록 모든 값에 대해 새로운 클래스를 만드는 것이 좋은 방법은 아니다. 주석문을 추가하거나 세터 함수 추가, 빌더 패턴을 사용하는 것이 대안이 될 수 있지만, 모두 단점이 있는 방법들이다.
##### 5.6.5 IDE는 어떤가?
킹텔리제이..쓰세요..
#### 5.7 설명되지 않은 값을 사용하지 말라
하드 코드로 작성된 값은 컴퓨터가 코드를 실행할 때 이 값을 알아야 하고, 개발자가 코드를 이해하기 위해 값의 의미를 알아야 한다.
##### 5.7.1 설명되지 않은 값은 혼란스러울 수 있다
코드에 설명되지 않은 값이 있으면 혼란을 초래하고 이로 인해 버그가 발생할 수 있다. 그 값이 무엇을 의미하는지를 다른 개발자들에게 명확하게 해주는 것이 중요하다.
##### 5.7.2 해결책 : 잘 명명된 상수를 사용하라
```// 예제 5.22
...
private const Double KILOGRAM_PER_US_TON = 907.1847;
private const Double METERS_PER_SECOND_PER_MPH = 0.44704;
...
```
##### 5.7.3 해결책 : 잘 명명된 함수를 사용하라
- 상수를 반환하는 공급자 함수[^provider function]
	```
	private static Double kilogramsPerUsTon() {
		return 907.1847;
	}
	```
- 변환을 수행하는 헬퍼 함수[^helper function]
	``` // 변환 계수는 함수를 호출하는 쪽에서는 몰라도 되는 구현 세부 사항이다.
	private static Double mphToMeterPerSecond(Double mph) {
		return mph * 0.44704;
	}
	```
#### 5.8 익명 함수를 적절하게 사용하라
**익명 함수**[^anonymous function]는 이름이 없는 함수이며, 일반적으로 코드 내의 필요한 지점에서 인라인으로 정의된다. 대부분의 주류 프로그래밍 언어는 어떤 형태로든 익명 함수를 지원한다. 간단하고 자명한 것에 익명 함수를 사용하면 코드의 가독성을 높여 주지만, 복잡하거나 자명하지 않은 것 혹은 재사용해야 하는 것에 사용하면 문제가 될 수 있다.
>함수형 프로그래밍
>익명 함수와 매개변수에 함수를 사용하는 기법은 함수형 프로그래밍, 특히 람다 표현과 관련되어 있다. 함수형 프로그래밍은 논리를 표현할 때 상태를 수정하는 명령문으로 하지 않고 함수에 대한 호출이나 참조로 표현하는 패러다임이다.
##### 5.8.1 익명 함수는 간단한 로직에 좋다
단순하고 자명한 논리에 익명 함수를 사용하는 것은 좋은 방법이지만, 코드 재사용성 관점에서 명명 함수를 만들어두는 것이 유용할 때가 있다. 적절하게 판단해서 사용하는 것이 가장 이롭다.
##### 5.8.2 익명 함수는 가독성이 떨어질 수 있다
함수의 이름은 그 함수가 무엇을 하는지 간결하게 요약해주는 역할을 갖고 있었는데 익명 함수는 이름이 없기 때문에 그 코드를 읽는 사람에게 어떠한 것도 제공하지 않는다.
``` // 예제 5.27
List<UInt16> getValidIds(List<UInt16> ids) {
	return ids.filter(id -> id != 0)
			.filter(id -> countSetBits(id & 0x7FFF) % 2 == ((id & 0x8000) >> 15));
}
```
##### 5.8.3 해결책 : 대신 명명 함수를 사용하라
``` // 예제 5.27
List<UInt16> getValidIds(List<UInt16> ids) {
	return ids.filter(id -> id != 0)
			.filter(isParityBitCorrect);
}
private Boolean isParityBitCorrect(UInt16 id) {
	...
}
```
적절한 조합은 가독성을 대단히 높일 수 있다!
##### 5.8.4 익명 함수가 길면 문제가 될 수 있다
함수형 스타일 코드를 작성할 때, 너무 많은 논리와 때로는 다른 익명 함수들을 중첩해서 가지고 있는 거대한 익명 함수를 생성한다. 이것을 여러 개의 명명 함수로 분리하면 코드의 가독성이 좋아진다. 익명 함수를 사용하더라도 하나의 함수가 너무 많은 일을 하지 않도록 더 작은 명명 함수로 나누어야 한다.
##### 5.8.5 해결책 : 긴 익명 함수를 여러 개의 명명 함수로 나누라
5.5 절의 내용과 매우 유사하다.
#### 5.9 프로그래밍 언어의 새로운 기능을 적절하게 사용하라
##### 5.9.1 새 기능은 코드를 개선할 수 있다
대표적으로 Java의 스트림은 반복문을 훨씬 간결하고 이해하기 쉬운 코드로 작성할 수 있게 했다. 무언가 직접 만드는 대신 언어에서 제공하는 기능을 사용하면 코드가 최적화되어 효율적이고 버그가 없을 가능성이 커진다.
##### 5.9.2 불분명한 기능은 혼동을 일으킬 수 있다
프로그래밍 언어에서 제공하는 기능이 확실한 이점을 가지고 있다 하더라도 해당 기능이 다른 개발자에게 얼마나 잘 알려져 있는지 고려할 필요가 있다. 가령, 규모가 크지 않은 자바 코드를 유지보수하고 있고, 다른 개발자가 자바 스트림에 익숙하지 않다면 스트림을 사용하지 않는 것이 좋을 수도 있다.
##### 5.9.3 작업에 가장 적합한 도구를 사용하라
프로그래밍 언어에서 새로운 기능은 다 이유가 있기에 추가된다. 이런 새로운 기능을 사용하면 큰 이점이 있을 수 있지만 코드 작성에서와 동일하게 해당 기능이 단지 새로워서가 아니라 작업에 적합한 도구이기 때문에 사용한다는 점을 분명히 해야 한다.

#GoodCodeBadCode