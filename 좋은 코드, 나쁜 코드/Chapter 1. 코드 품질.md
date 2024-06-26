좋은 소프트웨어를 만들기 위해서는 '고품질 코드'가 필수적이다. 고품질 코드는 일반적으로 좀 더 신뢰할 수 있고, 유지보수가 쉬우며, 버그가 적은 소프트웨어를 생산한다. 좀 더 구체적으로 고품질 코드는
- 최초 요구 사항을 완전히 충족한다.
- 요구 사항이 변화해도 많은 작업이 필요 없다.
- 오류가 발생하더라도 시스템이 복구되거나 부분적으로 작동한다.
- 명백하게 예상되지 않은 상황도 처리할 수 있다.
- 시스템이 공격을 받아도 안전한 상태이고 손상되지 않는다.

#### 1.1 코드는 어떻게 소프트웨어가 되는가?
코드가 소프트웨어가 되는 과정에 정해진 규칙은 없지만, 주요 단계는 일반적으로 다음과 같다.
1. 개발자가 코드베이스의 로컬 복사본을 가지고 작업하면서 코드를 변경한다.
2. 작업이 끝나면 코드 검토를 위해 변경된 코드를 가지고 병합 요청을 한다.
3. 다른 개발자가 코드를 검토하고 변경을 제안할 수 있다.
4. 작성자와 검토자가 모두 동의하면 코드가 코드베이스에 병합된다.
5. 배포는 코드베이스를 가지고 주기적으로 일어난다.
6. 테스트에 실패하거나 코드가 컴파일되지 않으면 코드베이스에 병합되는 것을 막거나 코드가 배포되는 것을 막는다.

#### 1.2 코드 품질의 목표
1. 작동해야 한다.
2. 작동이 멈춰서는 안 된다.
3. 변화하는 요구 사항에 적응해야 한다.
4. 이미 존재하는 기능을 또다시 구현해서는 안 된다.

#### 1.3 코드 품질의 핵심 요소
1. 코드는 읽기 쉬워야 한다.
	- 올바른 추상화 계층을 정의하는 것(2장)
	- 코드의 가독성을 높이는 구체적인 기법(5장)
2. 코드는 예측 가능해야 한다.
3. 코드를 오용하기 어렵게 만들라.
4. 코드를 모듈화하라.
5. 코드를 재사용 가능하고 일반화할 수 있게 작성하라.
6. 테스트가 용이한 코드를 작성하고, 제대로 테스트하라.

#### 1.4 고품질 코드 작성은 일정을 지연시키는가?
고품질 코드 작성에는 시간이 오래 걸리는 것이 사실이다. 하지만 장기적으로 보았을 때 생산성이 좋기 때문에 지향해야한다. 

#GoodCodeBadCode 