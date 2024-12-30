Mac에서 리눅스 커널을 `git clone` 하면 파일을 제대로 받아오지 못하는 문제


#### Download Linux kernel source code
- 라즈베리 파이용 리눅스 커널 소스 코드 다운로드
```bash
$ mkdir RPI_kernel
$ cd RPI_kernel
$ git clone --depth=1 https://github.com/raspberrypi/linux
```
(`--depth=1` 옵션은 최신 커밋만 가져오도록 한다.)

#### Docker를 이용한 Ubuntu 환경 구축
- `Ubuntu` 최신 이미지 다운로드
```bash
$ docker pull ubuntu
```
- 컨테이너를 생성하는 쉘 스크립트 작성
```bash
#!/usr/bin bash

if [ $# -lt 1 ]; then
  echo "Usage: $0 <container_name>"
  echo "Example: $0 my-container"
  exit 1
fi

CONTAINER_NAME="$1"

HOST_DIR="$HOME/Desktop/RPI_kernel"

CONTAINER_DIR="/workspace"

docker run -it --name "$CONTAINER_NAME" \
  -v "${HOST_DIR}:${CONTAINER_DIR}" \
  ubuntu:latest \
  /bin/bash
```
`workspace` 디렉토리에 리눅스 커널 소스 코드가 있는 디렉토리를 마운트했다.
#### Cross-compile the kernel
- 필수 의존성 및 툴체인 설치
```bash
$ apt-get update
$ apt-get install bc bison flex libssl-dev make libc6-dev libncurses5-dev
$ apt-get install crossbuild-essential-arm64
```
- 빌드 설정 및 빌드를 위한 쉘 스크립트 작성
```bash
#!/bin/bash

echo "Configure build output path"

KERNEL_TOP_PATH="$( cd "$(dirname "$0")" ; pwd -P )"
OUTPUT="$KERNEL_TOP_PATH/out"
echo "$OUTPUT"

KERNEL=kernel_2712
BUILD_LOG="$KERNEL_TOP_PATH/rpi_build_log.txt"

echo "move to kernel source"
cd "$KERNEL_TOP_PATH/linux"

export ARCH=arm64 
export CROSS_COMPILE=aarch64-linux-gnu-

echo "make defconfig"
make O="$OUTPUT" bcm2712_defconfig

echo "kernel build"
make O="$OUTPUT" Image modules dtbs -j$(nproc) 2>&1 | tee -a "$BUILD_LOG"  
```

