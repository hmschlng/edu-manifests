# kubernetes


## 주요 실습 내용

1. Single Master Cluster 구성 
   - lecture0 : VM 생성, kubernetes 클러스터 구성
2. 쿠버네티스 리소스 배포, 삭제 실습
   - lecture1 : Replication Controller, RelicaSet, Deployment, Rollout
   - lecture2 : Service
   - lecture3 : DaemonSet, StatefulSet
   - lecture4 : Ingress
   - lectrue5 : ConfigMap, Secrets, Namespace
   - lecture6 : Volume(PV & PVC), nfs
   - lecture7 : 배포 전략, nodeSelector, Affinity, AntiAffinity
3. CI/CD 파이프라인
   - lecture8 : docker build 및 수기 배포 & argoCD 구성/배포
   - lecture8-1 : github Action 활용 CI, argoCD 사용 CD
4. Service Mash
   - lectrue9 : istio 구성
5. 개인 과제
   - lecture99 : 개인 실습 과제 

## 사전 준비

- [ ] kt cloud 회원 가입 후 무료 체험 쿠폰(50만원) 발급(https://cloud.kt.com)
- [ ] git hub 계정 생성(https://github.com)
- [ ] docker hub 계정 생성(https://www.docker.com)
- [ ] chatGPT 계정 생성(https://chatgpt.com)

```bash

# docker build 테스트 소스 : 수정 중
git clone git@github.com:yeongdeokcho/edu-demo.git
```

## 기타

```dtd
IDE 다운로드 및 설치(택 1)
  - IntelliJ(https://www.jetbrains.com/idea)
  - Visual Studio Code(https://code.visualstudio.com)

쿠버네티스 참고 자료
  - 따베쿠 : https://www.youtube.com/playlist?list=PLApuRlvrZKohaBHvXAOhUD-RxD0uQ3z0c
  - 페스트캠퍼스 : 처음 시작하는 Docker & Kubernetes : 컨테이너 환경을 위한 필수 입문 강의 
```