최대 힙 구조를 유지시키는 함수인 `Max-heapify` 함수를 정의합니다. 이 함수의 입력으로는 배열 $A$와 배열의 인덱스 $i$가 주어집니다. 이 함수는 기본적으로 `Left(i)`와 `Right(i)`를 루트 노드로 가지는 부분 이진 트리를 최대 힙이라고 가정합니다. 하지만 $A[i]$의 값이 자식 노드의 값보다 작아서 최대 힙의 정의에 부합하지 않을 수 있습니다. 이 때 $A[i]$의 값을 서브 트리의 루트 노드와 값을 교환합니다.
```pseudo
Max-heapify(A, i)
l = Left(i)
r = Right(i)
if l <= A.heap-size and A[l] > A[i]
	largest = l
else largest = i
if r <= A.heap-size and A[r] > A[largest]
	largest = r
if largest != i
	exchange A[i] with A[largest]
	Max-heapify(A, largest)
```

$i$는 현재 노드의 인덱스이고, 조건문을 통해 현재 노드와 두 개의 자식 노드에 저장된 값을 비교하여 가장 큰 값을 찾습니다. 자식 노드에 저장된 값이 더 큰 경우 `largest != i`가 참이 되는데, 그 경우 각 인덱스에 저장된 값을 교환하고, 그 인덱스에 해당되는 노드를 루트 노드로 가지는 서브 트리에 대해 `Max-heapify` 함수를 호출하여 재귀적으로 저장된 값을 변경해 줍니다.

