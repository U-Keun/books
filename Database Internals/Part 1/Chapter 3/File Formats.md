B-Tree의 기본적인 개념에 대해서 알아보았으니, B-Tree와 다른 데이터 구조가 디스크에서 어떻게 구현되는지 알아보겠습니다. 메인 메모리에 접근하는 방식과 디스크에 접근하는 방식은 다릅니다. 어플리케이션 개발자의 관점에서, 메모리에 접근하는 것은 보통 투명하게 이루어집니다.[^1] 이것은 가상 메모리 때문입니다.

메모리 접근은 수동으로 오프셋을 관리할 필요가 없습니다. 디스크는 시스템의 호출로 접근할 수 있습니다. 

#### [[Motivation]]
#### [[Binary Encoding]]
#### [[General Principles]]
#### [[Page Structure]]
#### [[Slotted Pages]]
#### [[Cell Layout]]
#### [[Combining Cells into Slotted Pages]]
#### [[Managing Variable-Size Data]]
#### [[Versioning]]
#### [[Checksumming]]


#DataBaseInternals #DataBase 

[^1]: 메모리 접근이 투명하다는 것은, 프로그램이 메모리에 데이터를 읽고 쓰는 과정이 프로그램 코드에서 직접적으로 드러나지 않으며, 시스템에 의해 자동으로 관리된다는 의미입니다.