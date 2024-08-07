만약 컴퓨터의 속도가 무한대이고, 메모리에도 제한이 없다고 하더라도, 당신의 프로그램이 옳은 답을 제공한다는 것을 증명하기 위해서는 알고리즘을 공부해야 합니다.

컴퓨터가 무한히 빠르다면, 문제를 해결하기 위한 옳은 답이라면 무엇이든 상관없을 것입니다. 또한, 프로그램을 잘 설계하고 문서화가 잘 되어 있도록 구현하겠지만, 대부분의 경우 구현하기 가장 쉬운 방법을 찾을 것입니다.

컴퓨터는 분명히 빨라지지만, 무한히 빨라지지는 않습니다. 메모리도 저렴해지겠지만, 제한이 없을 수는 없습니다. 그렇기 때문에 계산 시간은 제한된 리소스이고, 메모리 공간도 마찬가지입니다. 이 리소스를 현명하게 사용해야 하고, 알고리즘을 통해 시간 또는 공간 관점에서의 효율을 챙길 수 있습니다.
###### Efficiency
같은 문제를 해결하는 서로 다른 알고리즘은 종종 효율 관점에서 크게 차이가 날 수 있습니다. 이 차이는 하드웨어나 소프트웨어의 차이로 인한 효율보다 더 중요할 수 있습니다.

정렬 알고리즘을 예로 들어보겠습니다. *삽입 정렬*[^insertion_sort]은 $n$개의 항목을 정렬하는 데에 대략 $c_1 n^2$ 정도의 시간이 걸립니다. 즉, 실행 시간이 $n^2$에 비례합니다. *병합 정렬*[^merge_sort]의 경우, 대략 $c_2 n\log_2 n$ 정도의 시간이 걸립니다. 일반적으로는 $c_1 < c_2$이지만, 상수 요소보다 입력 크기인 $n$이 실행 시간에 더 큰 영향을 미칩니다. 작은 $n$에 대해서는 삽입 정렬이 더 빠를 수 있지만, 충분히 큰 $n$에 대해서 $nlog_2 n$과 $n^2$의 차이는 상수 $c_1$과 $c_2$의 차이를 무시할 수 있을 만큼 큽니다.

구체적인 예시로, 빠른 컴퓨터(컴퓨터 A)에서 삽입 정렬을 수행하는 것과 느린 컴퓨터(컴퓨터 B)에서 병합 정렬을 수행하는 것을 비교해 봅시다. 두 컴퓨터는 1000만 개의 숫자를 정렬해야 합니다. 컴퓨터 A는 초당 100억 개의 명령어를 실행한다고 가정합니다. 그리고 컴퓨터 B는 초당 1000만 개의 명령어를 실행한다고 가정하면, 컴퓨터 A는 계산 능력 면에서 컴퓨터 B보다 1000배 더 빠릅니다. 차이를 더 극적으로 만들기 위해, 컴퓨터 A의 삽입 정렬은 세계 최고의 프로그래머가 코딩해서, $n$개의 숫자를 정렬하는 데 $2n^2$개의 명령어가 필요하다고 가정하겠습니다. 반면, 평균 수준의 프로그래머가 비효율적인 컴파일러를 사용하여 고급 언어로 합병 정렬을 구현해서, $n$개의 숫자를 정렬하는 데 $50n \log_2 n$개의 명령어가 필요하다고 가정하겠습니다. 1000만 개의 숫자를 정렬하기 위해서 컴퓨터 A는 $$\frac{2\cdot (10^7)^2 \mbox{ instructions}}{10^{10} \mbox{ instructions/sec}}= 20,000 \mbox{ sec (5.5시간 이상)}$$
정도의 시간이 걸리고, 컴퓨터 B는 $$\frac{50\cdot 10^7 \log_2 10^7 \mbox{ instructions}}{10^7 \mbox{ instructions/sec}}\approx 1,163 \mbox{ sec (20분 미만)}$$
정도의 시간이 걸립니다. 두 컴퓨터에서 배열 정렬은 $n$의 크기가 크면 클 수록 차이가 더 벌어질 것입니다.
###### Algorithms and other technologies
위의 예제는 알고리즘을 컴퓨터 하드웨어 같은 기술로 봐야한다는 것을 보여줍니다. 전체 시스템의 성능은 빠른 하드웨어를 선택하는 것만큼 효율적인 알고리즘을 선택하는 것에도 의존합니다. 컴퓨터 기술이 빠르게 발전하는 것처럼, 알고리즘 또한 빠르게 발전하고 있습니다.

다음과 같은 다른 첨단 기술을 고려할 때, 현대의 컴퓨터에서 알고리즘이 정말로 중요한지 의문이 들 수 있습니다:
- 고급 컴퓨터 아키텍처와 제조 기술,
- 사용하기 쉽고 직관적인 GUI[^graphic_user_interfaces],
- 객체 지향 시스템,
- 통합된 웹 기술,
- 유선 및 무선의 빠른 네트워킹.

물론 필요합니다. 일부 어플리케이션은 어플케이션 수준에서 알고리즘적인 내용이 명시적으로 보이지 않을 수 있지만, 많은 어플리케이션은 알고리즘을 필요로 합니다. 예를 들어, 한 위치에서 다른 위치로 이동하는 경로를 결정하는 웹 기반 서비스는 빠른 하드웨어, GUI, 광역 네트워킹 및 객체 지향 시스템이 필요합니다. 하지만 경로를 찾을 때는 최단 경로 알고리즘을 사용할 수 있고, 지도 렌더링이나 주소 보간 같은 특정 작업을 위해서도 알고리즘이 필요합니다.

어플리케이션 뿐만 아니라, 하드웨어와 GUI, 네트워킹에도 알고리즘은 사용됩니다. 알고리즘은 현대 컴퓨터에서 사용되는 대부분의 기술의 핵심입니다.

그리고, 위에서 보았듯이, 이전보다 더 큰 문제를 해결할 때 효율적인 알고리즘을 만드는 것은 매우 중요합니다.

현대의 컴퓨터 기술로는 알고리즘에 대해 많이 알지 못해도 작업을 어느 정도 수행할 수 있지만, 알고리즘에 대한 탄탄한 배경 지식이 있다면 훨씬 더 많은 작업을 효율적으로 수행할 수 있습니다.

#Algorithm 