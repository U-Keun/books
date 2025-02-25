### Issue #1
- Mac에서 리눅스 커널을 `git clone` 하면 파일을 제대로 받아오지 못하는 문제

Mac에서는 파일 이름을 대소문자로 구분하지 않는다. 그래서 아래와 같이 리눅스 커널을 설치하고자 했을 때 경고 메시지가 함께 떴다.

```bash
$ mkdir RPI_kernel
$ cd RPI_kernel
$ git clone --depth=1 https://github.com/raspberrypi/linux
```

```
warning: the following paths have collided (e.g. case-sensitive paths on a case-insensitive filesystem) and only one from the same colliding group is in the working tree: 'include/uapi/linux/netfilter/xt_CONNMARK.h' 'include/uapi/linux/netfilter/xt_connmark.h' 'include/uapi/linux/netfilter/xt_DSCP.h' 'include/uapi/linux/netfilter/xt_dscp.h' 'include/uapi/linux/netfilter/xt_MARK.h' 'include/uapi/linux/netfilter/xt_mark.h' 'include/uapi/linux/netfilter/xt_RATEEST.h' 'include/uapi/linux/netfilter/xt_rateest.h' 'include/uapi/linux/netfilter/xt_TCPMSS.h' 'include/uapi/linux/netfilter/xt_tcpmss.h' 'include/uapi/linux/netfilter_ipv4/ipt_ECN.h' 'include/uapi/linux/netfilter_ipv4/ipt_ecn.h' 'include/uapi/linux/netfilter_ipv4/ipt_TTL.h' 'include/uapi/linux/netfilter_ipv4/ipt_ttl.h' 'include/uapi/linux/netfilter_ipv6/ip6t_HL.h' 'include/uapi/linux/netfilter_ipv6/ip6t_hl.h' 'net/netfilter/xt_DSCP.c' 'net/netfilter/xt_dscp.c' 'net/netfilter/xt_HL.c' 'net/netfilter/xt_hl.c' 'net/netfilter/xt_RATEEST.c' 'net/netfilter/xt_rateest.c' 'net/netfilter/xt_TCPMSS.c' 'net/netfilter/xt_tcpmss.c' 'tools/memory-model/litmus-tests/Z6.0+pooncelock+poonceLock+pombonce.litmus' 'tools/memory-model/litmus-tests/Z6.0+pooncelock+pooncelock+pombonce.litmus'
```

리눅스는 기본적으로 파일 이름의 대소문자를 구분하고, 리눅스 커널 소스 코드에도 스펠링은 같지만 대소문자 차이가 있는 파일들이 `Makefile`을 통해 연결되어 있는 것이 많다. Mac에서는 해당 파일들이 같은 이름이지만 다른 내용을 담고 있는 것으로 판단했기 때문에, 빌드에 필요한 파일들이 사라졌고, 빌드가 제대로 되지 않는 문제가 있었다. 

또한, Docker를 이용해서 Ubuntu 컨테이너를 만들어도, 호스트 머신과 마운트 해둔 디렉토리에서 작업을 하면, 해당 디렉토리는 호스트 머신의 환경에 영향을 받는다. 다시 말하면, 마운트 해둔 디렉토리에서 `git clone`을 수행하면 똑같은 문제가 발생하는 것이다. 그래서 컨테이너 안에서도, 호스트 머신과 마운트 해둔 디렉토리가 아닌, 컨테이너 내부에서 `git clone` 을 수행하면 제대로 리눅스 커널 소스 코드가 받아지고, 빌드도 제대로 되는 것을 확인할 수 있었다. 

### Issue #2
`ftrace` 실습을 위해 `ftrace`의 옵션을 설정하는 쉘 스크립트를 작성하던 중에, 다음과 같은 명령을 수행할 때 `Invalid argument` 메시지가 떴다.
```bash
echo sys_clone do_exit > /sys/kernel/debug/tracing/set_ftrace_filter
```
로그가 찍히는 것으로 확인되기는 했지만, 아무래도 찝찝해서 약간 조사를 해보았다.

문법이 틀린 것은 아니지만, `sys_clone` 이라는 함수명이 실제로 해당 커널 버전에서 추적이 가능한 심볼로 노출되는지 확인해야 한다고 한다. 실제로 아래와 같이 확인해볼 수 있었다.
```bash
$ cat /proc/kallsyms | grep sys_clone
ffffd0008408c998 t __do_sys_clone
ffffd0008408ca40 t __do_sys_clone3
ffffd0008408ce00 T __arm64_sys_clone
ffffd0008408ce40 T __arm64_sys_clone3
```

그래서 처음의 명령문을 다음과 같이 수정했고, `Invalid argument` 메시지는 사라졌다!
```bash
echo __arm64_sys_clone do_exit > /sys/kernel/debug/tracing/set_ftrace_filter
```

#TroubleShooting