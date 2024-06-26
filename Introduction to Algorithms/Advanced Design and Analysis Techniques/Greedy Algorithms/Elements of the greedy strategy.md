그리디 알고리즘은 일련의 선택을 통해 문제에 대한 최적해를 구합니다. 각각의 선택에서, 알고리즘은 그 순간에 최선으로 보이는 선택을 합니다. 이러한 전략이 항상 최적해를 찾는다고 할 수는 없지만, 활동 선택 문제에서 보았듯이, 가끔은 최적해를 찾을 수 있습니다. 이번 절에서는 그리디 알고리즘의 일반적인 특징에 대해서 알아볼 것입니다.

16.1 절에서 살펴본 그리디 알고리즘을 만드는 과정은 일반적인 경우보다 조금 더 복잡했습니다. 그 과정은 다음과 같은 단계들을 거쳤습니다:
1. 문제에 대한 최적의 하위 구조를 결정하고,
2. 재귀적으로 표현되는 해를 만들고,
3. '탐욕적인' 선택을 하면 하나의 하위 문제만 남는 다는 것을 보이고,
4. '탐욕적인' 선택을 하는 것이 항상 안전함을 증명하고,
5. '탐욕적인' 전략을 재귀 함수로 구현하고,
6. 재귀 함수를 반복문을 통해 다시 구현했습니다.

위의 단계를 거치면서, 그리디 알고리즘의 기초가 되는 [[Dynamic Programming|동적 프로그래밍]]을 자세히 살펴보았습니다. 예를 들어, 활동 선택 문제에서 $i$와 $j$가 모두 변하는 하위 문제인 $S_{ij}$를 정의했습니다. 그리고 '탐욕적인' 선택을 하면 항상 하위 문제를 $S_k$ 형식으로 제한할 수 있었습니다.

대안으로, '탐욕적인' 선택을 이용해서 그 선택이 해결해야 할 하나의 하위 문제만 남기도록 최적의 하위 구조를 구성할 수 있습니다. 활동 선택 문제에서 $S_{ij}$가 아닌 $S_k$를 이용해서 하위 문제를 정의한 것이 바로 그것입니다. 그리고 다음의 '탐욕적인' 선택과 남아 있는 $S_m$에 대한 최적의 해를 결합해서 $S_k$에 대한 최적해를 찾을 수 있다는 것을 증명할 수 있었습니다. 더 일반적으로는, 다음과 같은 단계에 따라 그리디 알고리즘을 설계합니다:
1. 주어진 최적화 문제를 하나의 선택을 하고 나면 해결해야 할 하나의 부분 문제가 남는 형태로 변환합니다.
2. '탐욕적인' 선택이 항상 안전하고, 그 선택으로 기존의 문제에 대한 최적해를 찾을 수 있음을 증명합니다.
3. '탐욕적인' 선택을 한 후 남은 부분 문제가 최적의 하위 구조를 가진다는 것을 증명합니다. 다시 말하면, 하위 문제에 대한 최적의 해와 '탐욕적인' 선택을 결합하면 원래 문제에 대한 최적해에 도달함을 보여줍니다.

다음 절에서 이 과정에 대해 좀 더 알아보겠지만, 거의 모든 그리디 알고리즘 아래에는 더 복잡한 동적 프로그래밍 해법이 존재합니다.

어떤 최적화 문제를 그리디 알고리즘을 이용해 해결할 수 있는지를 판단하는 데에는 *탐욕 선택 속성*과 *최적 하위 구조*라는 두 가지 주요 요소를 고려해야 합니다. 주어진 문제에 이 요소가 있음을 증명할 수 있다면, 그리디 알고리즘을 사용하여 그 문제를 해결할 가능성이 높습니다.

###### 탐욕 선택 속성
첫 번째 요소인 탐욕 선택 속성은, 부분적으로 최적인 선택(탐욕 선택)을 통해 전체적으로 최적인 해결책을 구성하는 성질을 말합니다. 다시 말하면, 어떤 선택을 할 지 고려할 때, 부분 문제의 결과를 고려하지 않고 현재 문제에서 가장 좋아 보이는 선택을 합니다.

여기에서 그리디 알고리즘과 동적 프로그래밍의 차이가 발생합니다. 동적 프로그래밍에서는 각 단계에서 선택을 하지만, 그 선택은 일반적으로 부분 문제들의 해에 따라 결정됩니다. 결과적으로 동적 프로그래밍 문제는 작은 문제부터 해결한 뒤에 큰 문제를 해결하는 방식으로 진행됩니다.(코드에서 하향식으로 문제를 해결하는 것과는 개념이 조금 다릅니다. 코드에서 하향식으로 작동하더라도, 선택을 하기 전에 부분 문제를 먼저 해결해야 한다는 점은 변하지 않습니다.) 그리디 알고리즘에서는 현재 순간에 가장 좋아 보이는 선택을 하고, 남은 부분 문제를 해결합니다. 그리디 알고리즘이 내리는 선택은 지금까지의 선택에 의존할 수 있지만, 미래의 선택이나 부분 문제의 해에 의존할 수 없습니다. 동적 프로그래밍에서는 첫 번째 선택을 하기 전에 부분 문제를 해결하는 반면, 그리디 알고리즘은 어떤 부분 문제도 해결하기 전에 첫 번째 선택을 합니다. 동적 프로그래밍과는 다르게 그리디 알고리즘은 보통 하향으로 진행되고, 하나의 탐욕 선택을 하고 나면 주어진 문제를 더 작은 문제로 만듭니다.

물론, 각 단계에서 탐욕 선택이 전체적으로 최적의 해결책을 제공한다는 것을 증명합니다. 일반적으로 증명은, 어떤 부분 문제에 대한 전역적인 최적해를 고려합니다. 그리고 다른 선택 대신 탐욕 선택을 적용해서, 비슷하면서 더 작은 하위 문제를 해결하도록 합니다.

탐욕 선택은 선택에 고려되는 대상이 많을 때 더 효율적일 수 있습니다. 활동 선택 문제에서는 선택에 고려되는 대상이 많았지만, 활동들을 종료 시간이 증가하는 순서로 정렬해 둔 것은 각 활동을 한 번만 검토할 수 있게 했습니다. 이처럼 입력을 전처리하거나 적절한 자료 구조(우선순위 큐 같은)를 사용하여 탐욕 선택을 빠르게 할 수 있고, 이것으로 효율적인 알고리즘을 만들 수 있습니다.

###### 최적 하위 구조
문제가 최적 하위 구조를 나타내려면, 그 문제에 대한 최적해가 하위 문제들에 대한 최적해를 포함하고 있어야 합니다. 이것 동적 프로그래밍 뿐만 아니라 그리디 알고리즘을 적용할 수 있는지를 판단하는 데 필요한 요소입니다. 활동 선택 문제에서, 하위 문제 $S_{ij}$의 최적해가 활동 $a_k$를 포함하면 하위 문제 $S_{ik}$와 $S_{kj}$의 최적해도 그것을 포함해야 합니다. 주어진 최적 하위 구조를 바탕으로, 만약 수행할 활동을 $a_k$로 알고 있다면, $a_k$를 선택하고 하위 문제 $S_{ik}$와 $S_{kj}$의 최적해에 포함된 모든 활동을 선택해서 $S_{ij}$에 대한 최적해를 구성할 수 있었습니다. 그리고 이 최적 하위 구조에 대한 관찰을 기반으로, 최적해를 계산하는 데 필요한 점화식을 만들 수 있었습니다.

그리디 알고리즘에 최적 하위 구조를 적용할 때는 일반적으로 더 직접적인 접근 방식을 사용합니다. 위에서 언급한 것처럼, 기존의 문제에서 탐욕 선택을 한 후 하위 문제에 도달했다고 가정할 수 있었습니다. 거기서 실질적으로 해야 하는 것은 하위 문제에 대한 최적해와 그 전에 선택했던 것을 합치면 원래 문제에 대한 최적해가 된다는 것을 논증하는 것입니다. 이 방식은 암묵적으로는 하위 문제에 대한 귀납법을 사용하여, 각각의 단계에서 탐욕 선택을 하는 것이 최적해를 생성함을 증명합니다.

###### 그리디 알고리즘 vs 동적 프로그래밍
그리디 알고리즘과 동적 프로그래밍 모두 최적 하위 구조를 사용하기 때문에, 그리디 알고리즘을 사용하는 대신 동적 프로그래밍 방식으로 해를 구성하거나, 동적 프로그래밍으로만 해결할 수 있지만 그리디 알고리즘으로 해결하려는 경우도 있습니다. 두 방식의 미묘한 차이에 대해 살펴보기 위해, 전통적인 최적화 문제의 두 가지 변형에 대해 알아보겠습니다.

*0-1 배낭 문제*는 다음과 같습니다: 어떤 도둑이 가게에서 $n$개의 물건을 훔치려고 합니다. $i$번째 물건은 $v_i$ 만큼의 가치가 있고, 무게는 $w_i$입니다.(각 변수는 정수입니다.) 도둑은 최대한 가치가 크도록 물건을 훔치고 싶지만, 챙길 수 있는 최대 무게는 $W$입니다. 어떤 물건을 챙겨야 할까요?(도둑은 물건을 부분적으로 챙길 수 없음을 가정합니다.)

*분할 배낭 문제*에서는 위의 문제와 기본적인 설정은 동일하지만, 각각의 물건에 대해 이진(0-1) 선택을 해야 하는 대신 물건의 일부를 가져갈 수 있습니다.

두 배낭 문제 모두 최적 하위 구조 속성을 가지고 있습니다. 0-1 배낭 문제에서는, 최대 무게가 $W$인 가장 가치 있는 선택을 가정합니다. 그 선택에서 $j$ 번째 물건을 제거하면, 남은 선택은 도둑이 원래 $n - 1$개의 물건($j$ 번째 물건 제외)에서 가져갈 수 있는 최대 무게 $W - w_j$의 가장 가치 있는 선택이어야 합니다. 분할 배낭 문제에서는, 최적의 선택에서 $j$ 번째 물건의 무게 $w$ 만큼을 제거하면, 남은 것은 원래의 $n - 1$개의 물건과 $j$ 번째 물건의 $w_j - w$ 무게 만큼을 포함하여 가져갈 수 있는 최대 무게 $W-w$ 의 가장 가치 있는 선택이어야 합니다.

유사한 문제지만, 분할 배낭 문제는 그리디 알고리즘으로 해결할 수 있는 반면, 0-1 배낭 문제는 그리디 알고리즘으로 해결할 수 없습니다. 분할 배낭 문제의 경우, 각 물건에 대해 단위 무게 $1$ 당 가치인 $\frac{v_i}{w_i}$를 계산합니다. 도둑은 단위 무게당 가장 높은 가치를 가진 항목을 최대한 가져가는 것으로 시작합니다. 만약 그 물건을 모두 챙긴 후에도 더 많이 챙길 수 있다면, 단위 무게 당 두 번째로 높은 가치를 가진 항목을 최대한 가져갑니다. 이 방식으로 무게 제한 $W$에 도달할 때까지 진행합니다. 물건을 단위 무게당 가치에 따라 정렬하면, 그리디 알고리즘은 $O(n \log n)$의 시간 복잡도를 가지게 됩니다. 

위의 방식은 0-1 배낭 문제에는 적용할 수 없습니다. 예를 들어, 3개의 물건이 있고 훔칠 수 있는 물건의 최대 무게가 $50$이라고 하겠습니다. 각각의 물건은, 첫 번째 물건은 가치 $60$에 무게 $10$, 두 번째 물건은 가치 $100$에 무게 $20$, 세 번째 물건은 가치 $120$에 무게 $30$ 입니다. 각각의 단위 무게당 가치는 순서대로 $6$, $5$, $4$ 이므로, 위의 방식에 따르면 첫 번째 물건을 선택해야 합니다. 하지만 이 문제에서는 첫 번째 항목을 제외한 나머지 물건을 선택하는 것이 더 높은 가치를 가집니다. 즉, 최적해를 구하지 못합니다.




#Algorithm #GreedyAlgorithm