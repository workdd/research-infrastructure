# research-infrastructure
DDPS 연구실 실험 환경을 위한 쿠버네티스 인프라 구축

## 세팅 순서
### 1. Docker, Kubeadm 등 필수 패키지 설치 및 linux 설정
```
source setup.sh
```

### 2. kubeadm init
- Master Node에서 init
```
sudo kubeadm init \
		--apiserver-advertise-address=0.0.0.0 \
		--pod-network-cidr=<public subnet cidr> \
		--apiserver-cert-extra-sans=<master node public ip> \
		--ignore-preflight-errors=ALL
```

- Worker Node에서 join
Master Node에서 init 완료 시 join에 대한 명령어 확인 후 복사하여 Worker Node에서 실행


### 3. kubectl 설치
- Master Node에서만 진행
```
source kubectl.sh
```

## Docker 환경 설정 변경
  - Docker service 파일 변경
    ```
    sudo vi /lib/systemd/system/docker.service
    ```
  - ExecStart 구문 뒤에 systemd 관련 명령어 추가
    ```
    [Service]
    Type=notify                                                                    
    # the default is not to use systemd for cgroups because the delegate issues still
    # exists and systemd currently does not support the cgroup feature set required
    # for containers run by docker                                                
    ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd
    ExecReload=/bin/kill -s HUP $MAINPID
    TimeoutSec=0
    RestartSec=2
    Restart=always
    ```
- Docker 설정 반영
  ```
  sudo systemctl daemon-reload
  sudo systemctl restart docker
  ```
