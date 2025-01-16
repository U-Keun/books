`task_struct` 구조체는 프로세스의 속성 정보를 포함하고 있다. 

#### 프로세스 이름
```C
char comm[TASK_COMM_LEN];
```
`comm` 문자 배열은 프로세스 이름을 저장한다.

명령줄에 `ps -ely` 명령을 이용해 볼 수 있는 내용 중 `CMD` 칸에 보이는 이름들(`systemd`, `kthreadd` 등)이 위의 `comm` 변