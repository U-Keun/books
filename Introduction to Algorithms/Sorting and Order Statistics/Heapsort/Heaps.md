*(이진) 힙*[^(binary)heap] 자료 구조는 완전 이진 트리 형태로 볼 수 있는 배열 객체입니다. 트리의 각각의 노드는 배열의 원소에 대응됩니다. 트리는 가장 낮은 레벨을 제외하고는 모든 레벨이 완전히 채워져 있고, 가장 낮은 레벨은 왼쪽부터 채워집니다. 힙을 나타내는 배열 $A$에 대해 $A.length$는 배열에서 원소의 개수를 의미하고, $A.heap-size$는 힙에 저장된 원소의 개수를 의미합니다. 다시 말하면, 배열 $A[1\dots A.length]$의 모든 원소는 특정 값을 포함할 수 있지만, 유효한 정보는 $A[1\dots A.heap-size]$에 저장되어 있습니다. 트리의 루트 노드는 $A[1]$이고, 노드의 인덱스 $i$를 이용해서 그 노드의 부모 노드, 왼쪽 자식 노드, 오른쪽 자식 노드의 인덱스를 계산할 수 있습니다:
```pseudo
Parent(i)
return ⌊i / 2⌋

Left(i)
return 2 * i

Right(i)
return 2 * i + 1
```
위의 과정은 이진 표현을 오른쪽 또는 왼쪽으로 1비트 만큼 옮기는 것으로 간단하게 구현할 수 있습니다.(물론 1을 더하는 계산도 필요합니다.)

이진 힙에는 기본적으로 *최대 힙*[^max-heap]과 *최소 힙*[^min-heap]이 있습니다. 최대 힙에서는 모든 노드 $i$에 대해 다음의 조건을 만족합니다 : $$A[Parent(i)] \ge A[i],$$ 즉, 노드에 저장된 값은 부모 노드에 저장된 값보다 작다는 것을 의미합니다. 최대 힙에서 루트 노드에 저장된 값이 가장 크다는 것을 알 수 있고, 각각의 노드를 루트로 가지는 서브 트리에 저장된 값들은 루트 노드에 저장된 값보다 작습니다.

최소 힙은 최대 힙과 반대로 다음의 조건을 만족하도록 구성됩니다: $$A[Parent(i)]\le A[i].$$
최소 힙에서 루트 노드에는 가장 작은 값이 저장됩니다.

힙 정렬 알고리즘은 최대 힙이나 최소 힙에서 비슷하게 구현됩니다. 최소 힙은 보통 우선 순위 큐를 구현하는 데 사용되므로, 이번에는 최대 힙을 이용해서 힙 정렬 알고리즘을 알아볼 것입니다.




