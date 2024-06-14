*Huffman 코드*는 데이터를 효과적으로 압축합니다. 데이터의 특성에 따라 다르지만, 일반적으로 20%에서 90%까지 절약됩니다. 여기서 데이터는 기본적으로 문자들의 시퀀스로 간주합니다. Huffman의 그리디 알고리즘은 각 문자가 얼마나 자주 나타나는지(즉, 빈도)를 나타내는 정보를 사용하여 각 문자를 이진 문자열로 나타내는 최적의 방법을 구축합니다.

10만 개의 문자로 이루어진 데이터 파일을 압축하여 저장하는 상황을 가정하겠습니다. 그리고 파일에 있는 문자들이 아래의 표에 주어진 빈도로 나타난다고 가정합니다. 가령, 주어진 파일에는 6개의 문자만 사용되었고, a가 45,000번 나타나=났음을 의미합니다.

|             | a   | b   | c   | d   | e    | f    |
| ----------- | --- | --- | --- | --- | ---- | ---- |
| 빈도(단위 1000) | 45  | 13  | 12  | 16  | 9    | 5    |
| 고정 길이 코드    | 000 | 001 | 010 | 011 | 100  | 101  |
| 가변 길이 코드    | 0   | 101 | 100 | 111 | 1101 | 1100 |
파일의 정보를 표현하는 방법은 여러 가지지만, 여기서는 이진 문자 코드를 설계하는 문제를 생각합니다. 이진 문자 코드를 *고정 길이 코드*로 사용할 수 있습니다. 위의 표의 내용처럼, 각각의 문자를 3 비트로 표현한다면 전체 파일의 크기는 300,000 비트가 될 것입니다.

반면, 같은 표의 가변 길이 코드를 사용해서, 상대적으로 자주 사용되는 문자에 더 적은 비트가 사용되도록 정하면 고정 길이 코드보다 파일 크기를 작게 만들 수 있습니다. 위의 표의 내용을 이용하면 주어진 가변 길이 코드로는 224,000 비트가 됩니다. 실제로 이 가변 길이 코드는 이 파일을 가장 작게 만들 수 있는 최적의 문자 코드입니다.
###### Prefix codes
여기서는 어떤 이진 코드도 다른 이진 코드의 접두사가 되지 않는 코드만 고려합니다. 이러한 코드를 *prefix code*라고 부릅니다. 여기서 증명하진 않겠지만, prefix code는 모든 문자 코드 중 가장 최적으로 데이터를 압축할 수 있으므로, prefix code만 고려해도 괜찮습니다.

임의의 이진 문자 코드에 대한 인코딩은 파일의 각 문제를 나타내는 코드를 단순히 연결하기만 하면 됩니다. 예를 들어, 위의 표를 이용해 `abc`를 나타내는 코드는 `0101100`이 됩니다.

prefix code는 디코딩도 단순합니다. 임의의 이진 코드는 다른 이진 코드의 접두사가 될 수 없으므로, 인코딩된 파일을 시작하는 코드는 명확합니다. 코드를 앞에서부터 식별하여 원래 문자로 변환하는 과정을 반복하면 됩니다. 예를 들어, 문자열 `001011101`은 고유하게 `0⋅0⋅101⋅1101`로 나뉘며, 이것은 `aabe`가 됩니다.

디코딩 과정에서는 접두사 코드를 편리하게 표현할 수 있는 방법이 필요합니다. 주어진 문자들이 리프 노드가 되는 이진 트리를 통해 코드가 어떻게 나뉘는지 쉽게 찾을 수 있습니다. 문자의 이진 코드는 트리의 루트에서 해당 문자까지의 경로로 해석되며, `0`은 '왼쪽 자식 노드로 이동', `1`은 '오른쪽 자식 노드로 이동'을 의미합니다. 아래의 그림은 처음 보았던 예제를 트리로 표현한 것입니다(이 트리들은 이진 탐색 트리가 아닙니다.).

<p align="center">
	<img width="400" src="../../../images/스크린샷 2024-06-13 오후 3.09.14.png">
</p>
![[스크린샷 2024-06-13 오후 3.09.14.png|center|400]]

파일에 대한 각각의 문자의 최적 코드는 *정 이진 트리*[^full binary tree]로 표현됩니다. 

#Algorithm #GreedyAlgorithm 