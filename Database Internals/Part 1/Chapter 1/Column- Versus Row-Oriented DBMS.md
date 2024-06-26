대부분의 데이터베이스 시스템은 *테이블*의 *열*[^column]과 *행*[^row]로 구성된 *데이터 레코드*의 집합을 저장합니다. *필드*[^field]는 특정 열과 행의 원소입니다:특정 타입의 단일 값입니다. 같은 열에 포함되어 있는 필드는 같은 데이터 타입을 가집니다. 예를 들면, 사용자 레코드를 가진 테이블을 정의하면, 모든 이름은 같은 타입일 것이므로 같은 열에 포함됩니다. 논리적으로 같은 레코드(일반적으로 특정 키로 식별되는)에 포함되는 값의 모음은 행을 구성합니다.

데이터베이스는 데이터가 디스크에 행으로 저장되는지 또는 열로 저장되는지로 분류할 수 있습니다. 테이블은 수평적으로(같은 행에 포함되는 값을 저장하는 방식) 또는 수직적으로(같은 열에 포함되는 값을 저장하는 방식) 나뉠 수 있습니다.

행 기반 DBMS의 예로는 MySQL, PostgreSQL과 대부분의 관계형 데이터베이스가 포함됩니다. 열 기반 DBMS의 예로는 MonetDB와 C-Store가 있습니다.
##### Row-Oriented Data Layout
행 기반 DBMS는 데이터를 레코드 또는 행으로 저장합니다. 데이터 레이아웃은 모든  행에 동일한 필드의 집합을 가지고 있는 표 형식의 데이터에 가깝습니다. 예를 들어, 행 기반 데이터베이스는 이름, 생일, 전화번호를 가진 사용자 목록을 효율적으로 저장할 수 있습니다:

| ID  | Name | Birth Date | Phone Number |
| --- | ---- | ---------- | ------------ |
| 10  | 김정현  | ~          | ~            |
| 20  | 송우근  | ~          | ~            |
| 30  | 송지호  | ~          | ~            |
이러한 접근 방식은 특정 키로 유일하게 식별되는 레코드가 몇 개의 필드로 구성되는 경우에 유용하다. 하나의 사용자를 표현하는 모든 필드는 종종 함께 읽기 작업이 수행된다. 레코드를 생성할 때도 모든 필드에 대해 쓰기 작업이 수행된다. 동시에, 각각의 필드는 각각 수정될 수 있다.

행 기반 저장소는 행으로 데이터에 접근해야 하는 시나리오에 가장 유용하기 때문에, 전체 행을 함께 저장하는 것은 공간적 지역성[^1]을 개선합니다.

디스크 같은 영구 저장 매체에 있는 데이터는 보통 *블록*[^2] 단위로 접근되기 때문에, 하나의 블록은 모든 열의 데이터를 포함할 것입니다. 이것은 하나의 사용자 레코드 전체에 접근하고 싶은 경우에는 좋지만, 모든 필드에 대한 데이터도 호출되기 때문에 여러 사용자 레코드의 개별 필드에 접근하는 쿼리는 비용이 높아집니다.
##### Column-Oriented Data Layout
열 기반 DBMS는 데이터를 행 대신 열로 나누어 저장합니다. 같은 열에 있는 값들은 디스크에 연속적으로 저장됩니다. 예를 들어 주식시장 시세를 저장한다면, 시세들이 함께 저장됩니다. 다른 열에 대한 값을 별도의 파일 또는 파일 세그먼트에 저장하면 열별로 효율적인 쿼리를 사용할 수 있습니다. 행 기반 데이터베이스에서는 전체 행을 소비해서 필요하지 않은 열 데이터를 버려야 하지만, 열 기반 데이터베이스에서는 한 번에 읽을 수 있기 때문입니다.

열 기반 저장소는 경향을 찾거나, 평균값을 계산하는 등 집계하는 분석적인 작업에 적합합니다. 논리적 레코드가 여러 필드를 가지고 있지만 그 중 일부(이 경우에는 가격 견적)는 중요도가 다르고 종종 함께 사용될 때, 복잡한 집계 처리가 사용될 수 있습니다.

논리적 관점에서는 주식 시장 시세 데이터도 테이블로 표현할 수 있습니다:

| ID  | Symbol | Date | Price |
| --- | ------ | ---- | ----- |
| 1   | DOW    | ~    | ~     |
| 2   | S&P    | ~    | ~     |
하지만 열 기반 데이터의 레이아웃은 물리적으로 완전히 다릅니다. 같은 행에 포함되는 값들은 서로 가깝게 저장됩니다:
```data
Symbol: 1:DOW; 2:S&P
Date: 1:~; 2:~
Price: 1:~; 2:~
```
조인, 필터링 및 다중 행 집계에 유용할 수 있는 데이터 튜플을 재구성하려면 해당 데이터가 다른 열에서 어떤 데이터 포인트와 연결되어 있는지 식별하기 위해 열 수준에서 일부 메타데이터를 보존해야 합니다. 이것을 명시적으로 한다면, 각각의 값은 키를 가지고 있어야 하는데, 이것은 중복을 유발하고 저장된 데이터의 양을 늘립니다. 어떤 열은 암시적 식별자(가상 ID)를 사용해 저장하고 값의 위치(*offset*)를 통해 관계된 값과 매핑하는 방식을 사용합니다.

지난 몇 년 동안, 커지는 데이터셋에 대한 복잡한 분석 쿼리 실행에의 수요가 증가함에 따라, Apache Parquet, Apache ORC, RCFile과 같은 새로운 열 지향 파일 형식과 Apache Kudu, ClickHouse 등과 같은 열 지향 저장소가 많이 등장했습니다.
##### Distinctions and Optimizations
행 기반 저장소와 열 기반 저장소는 데이터가 저장되는 방식만으로 구별할 수 없습니다. 데이터 레이아웃을 선택하는 것은 가능한 최적화 방법에서 하나의 단계일 뿐입니다.

한 번의 실행에서 같은 열의 여러 값을 읽는 것은 캐시 활용도과 계산 효율을 개선합니다. 현대의 CPU에서는 벡터화된 명령어를 사용하여 단일 CPU 명령어로 여러 데이터 포인트를 처리할 수 있습니다.

같은 데이터 타입의 값을 함께 저장하는 것은 더 나은 압축률을 제공합니다. 데이터 타입에 다라 다른 압축 알고리즘을 사용할 수 있고, 각각의 경우에 대해 가장 효율적인 압축 방법을 선택합니다.

행 기반 저장소나 열 기반 저장소 중 어떤 것을 선택할지는 *접근 패턴*에 따라 다릅니다. 읽을 데이터가 레코드에서 소비되고[^3] 작업 대부분이 점 쿼리 및 범위 검색이라면, 행 기반 저장소가 더 적합합니다. 만일 검색이 여러 행에 걸쳐 있거나, 열의 부분 집합에 대한 집계를 계산하는 것이라면, 열 기반 저장소가 적합합니다.
##### Wide Column Stores
열 기반 데이터베이스는 BigTable이나 HBase 같은 *wide column store*와 혼동되지 않아야 합니다. wide column store에서 데이터는 다차원 맵으로 표현되고 열은 열 패밀리(일반적으로 동일한 유형의 데이터 저장)로 그룹화되며 각 열 패밀리 내부에는 데이터가 행 단위로 저장됩니다. 이러한 레이아웃은 키 또는 키 시퀀스에 의해 조회되는 데이터를 저장하는 데 적합합니다.

BigTable 논문의 예제로 Webtable이 있습니다. Webtable은 웹 페이지의 내용, 속성, 특정 시점의 웹 페이지간의 관계를 스냅샷으로 저장합니다. 각 페이지는 역방향 URL[^4]로 식별되고, 모든 속성은 스냅샷이 찍힌 시점의 타임스탬프로 식별됩니다. 단순하게 다음과 같은 중첩된 맵 형식으로 표현될 수 있습니다.
```
{
	"com.cnn.www": {
		contents: {
			t6: html: "<html>..."
			t5: html: "<html>..."
			t3: html: "<html>..."	
		}
		anchor: {
			t9: cnnsi.com: "CNN"
			t8: my.look.ca: "CNN.com"
		}
	}
	"com.example.www": {
		contents: {
			t5: html: "<html>..."
		}
		anchor: {}
	}
}
```

데이터는 계층적 인덱스가 있는 다차원 정렬 맵에 저장됩니다: 특정 웹 페이지와 관련된 데이터를 역방향 URL과 타임스탬프에 의해 내용 또는 앵커로 찾을 수 있습니다. 각각의 행은 행의 키로 인덱싱 됩니다. 위의 예제에서 관련된 열들은  `contents`와 `anchor`라는 열 패밀리로 그룹화되었고 디스크에 따로 저장됩니다. 열 패밀리 안에 있는 각각의 열은 열의 키로 식별되는데, 이것은 열 패밀리의 이름과 `html`, `cnnsi.com`, `my.look.ca` 같은 키워드의(qualifier) 조합입니다. 열 패밀리는 타임스탬프에 따라 여러 버전의 데이터를 저장합니다. 이 레이아웃을 사용하면 상위 수준의 항목(이 경우 웹 페이지)과 해당 항목의 매개 변수(내용 버전 및 다른 페이지에 대한 링크)를 빠르게 찾을 수 있습니다.

위의 레이아웃은 wide column store의 개념을 이해하는 데 유용하지만, 물리적인 레이아웃은 조금 다릅니다. 아래와 같이 열 패밀리가 저장되는 레이아웃을 표현할 수 있습니다: 열 패밀리는 따로 저장되지만, 각각의 열 패밀리 안에서 같은 키를 포함하는 데이터는 함께 저장됩니다.

<p align="center">
	<img width="400" src="../../../images/스크린샷 2024-04-27 오후 3.22.39.png">
</p>

![[스크린샷 2024-04-27 오후 3.22.39.png|center|400]]

#DataBase #DataBaseInternals 

[^1]: 관련된 데이터 요소들이 메모리 상에서 서로 가까이 위치할 때 높은 공간적 지역성(spatial locality)을 가진다고 합니다.
[^2]: 디스크 상에서 데이터는 물리적으로 블록이라는 작은 단위로 구분되어 저장됩니다. 이 블록들은 일정한 크기(예: 512바이트, 4KB 등)를 가지며, 디스크에서 데이터를 읽거나 쓸 때는 이 블록 단위로 처리됩니다.
[^3]: 대부분 또는 모든 열이 요청되는 경우
[^4]: 링크 : [why we use reversed url identifier](https://stackoverflow.com/questions/11681430/why-we-use-reversed-url-identifier-on-xcode)