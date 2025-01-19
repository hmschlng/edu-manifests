# lecture-8-1 : CI/CD 파이프 라인 (gitHub Action)
- ### gitHub Action 활용 CI : Repository Secrets, Workflow, Run workflow 실행 
- ### AgroCD 활용 CD : Application 등록, 배포(Sync)
- ### [Kustomize를 이용한 쿠버네티스 오브젝트의 선언형 관리](https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/kustomization/)

# 1. github Action 생성 & CI

## 1.1 gitHub Actions Secrets 생성
- 이미지 레포지토리(dockerhub) 로그인을 위한 Secret 생성
  - docker 이미지 build와 이미지 레포지토리에 push 할때 사용할 secret
- 형상관리(github) push 위한 Secret 생성
  - CI 시 빌드된 이미지 버전(TAG) manifests 내(kustomization.yaml) 이미지 버전 수정후 commit, push 위해

### 1.1.1 사전 준비 : dockerhub PAT(Personal Access Token) 생성
- 우측 상단 프로필 사진 선택 > Account settings 선택 > Personal Access Token 선택
  ![dockerhub token 생성](/lecture8-1/img/lecture8-1-git-docker-token01.png)
- Generate new token 버튼 선택 > Create access token 화면에서 토큰 정보 입력
  - 접근권한 : Read, Write, Delete 선택
  
  ![dockerhub token 생성](/lecture8-1/img/lecture8-1-git-docker-token02.png)

- 토큰 복사, 저장
  ![dockerhub token 생성](/lecture8-1/img/lecture8-1-git-docker-token03.png)

### 1.1.2 Repository Secrets 생성
```dtd

#. 이미지 레포지토리(dockerhub) 서버 URL
   o Name: DOCKERHUB_LOGIN_SERVER
   o Secret: docker.io

#. 이미지 레포지토리 로그인 계정
   o Name: DOCKERHUB_USERNAME
   o Secret: mosy@paran.com

#. 이미지 레포지토리 비밀번호 : 여기서는 1.1 dockerhub PAT
   o Name: DOCKERHUB_PASSWORD
   o Secret: dockerhub PAT

#. github Access 토큰(PAT)
   o Name: REPO_TOKEN
   o Secret: github PAT (lecture8 zz-lec8-1.md 참조)
```
- 우측 상단 Settings 선택 > Secrets and variables 선택 > Actions 선택 > New repository secret 선택 
  ![Repository secrets 생성](/lecture8-1/img/lecture8-1-git-action-secret-01.png)
- 이미지 레포지토리(dockerhub) 서버 URL
  ![Repository secrets docker url](/lecture8-1/img/lecture8-1-git-dockerhub-login-server.png)
- 이미지 레포지토리 로그인 계정
  ![Repository secrets docker user](/lecture8-1/img/lecture8-1-git-dockerhub-username.png)
- 이미지 레포지토리 비밀번호 : 여기서는 1.1 dockerhub PAT
  ![Repository secrets docker pw](/lecture8-1/img/lecture8-1-git-dockerhub-password.png)
- github Access 토큰(PAT)
  ![Repository secrets github pat](/lecture8-1/img/lecture8-1-git-repo-token.png)
- Repository Secrets 목록
  ![Repository secrets list](/lecture8-1/img/lecture8-1-git-action-secrets.png)

## 1.2 Workflow 생성 : CI 실행 Job 생성
- gradle build 워크플로우 : ci.yaml
```yaml

## 워크플로우 이름 정의
name: Java CI with Gradle

## 워크플로우 실행을 trigger 시키는 이벤트 설정
# workflow_dispatch : 수동으로 실행
# github에 소스 push, pull request or schedule, fork, release.. 등 
on: workflow_dispatch

## 워크플로우 전역에서 사용할 환경 변수  
env:
  REPO_ACCOUNT: ydcho0902
  IMAGE_NAME: k8s-edu

## 실행할 작업 정의 
jobs:
  build:                     # 종류 : test, deploy, lint(정적분석), staging, production, db 등 ..
    runs-on: ubuntu-latest   # 작업이 진행될 가상 환경 정의 : 여기서는 ubuntu 최신 버전에서 build 진행 

# 작업에서 실행할 단계, 순차적으로 실행 됨
    steps:
      - uses: actions/checkout@v4         # 현재 리포지토리의 코드를 체크아웃하는 GitHub Actions 제공 기본 액션
        with:
          token: ${{secrets.REPO_TOKEN}}  # github 리포지토리 접근 위한 Secret

      - name: Set up JDK 17               # Gradle 빌드 위한 jave 설치 및 환경 설정
        uses: actions/setup-java@v4       # GitHub Actions 제공하는 java setup 기본 액션
        with:
          java-version: '17'              # 설치할 Java의 버전 지정
          distribution: 'temurin'         # JDK 배포판(zulu, adopt, openjdk..) 지정, Eclipse Temurin은 OpenJDK 기반 배포판

      - name: Build with Gradle Wrapper   # Gradle 빌드 사용해 프로젝트를 빌드
        run: ./gradlew build              # 이 단계에서 실행할 쉘 명령어 : gradle build 실행

      - name: Setup Kustomize                 # Kubernetes 리소스 관리, 배포하는 Kustomize 설치
        uses: imranismail/setup-kustomize@v1  # GitHub Actions 제공하는 setup-kustomize 액션 사용(imranismail 만든 Action)

      - name: 'Gen Version'                   # 컨테이너 이미지 버전 관리를 위한 TAG로 사용한 값 생성 
        id: gen-version                       # 이후의 다른 단계에서 이 단계 출력값이나 결과 참조할 수 있도록 ID 정의
        run: |                                # 이 단계에서 실행할 쉘 명령어, 타임존(한국 표준시), 날짜 포멧 정의
          export TZ=Asia/Seoul                
          echo "VERSION=$(date +%Y%m%d%H%M)" >> $GITHUB_ENV
# echo "VERSION=$(...)" >> $GITHUB_ENV : 환경변수 VERSION에 date 값을 저장하고, GitHub Actions의 전역 환경 변수 $GITHUB_ENV에 저장
      
      - name: 'Dockerhub login'               # 이미지 빌드를 위해 dockerhub에 로그인 
        uses: docker/login-action@v1          # GitHub Actions에서 제공하는 Docker 레지스트리 로그인 Action
        with:
          login-server: ${{ secrets.DOCKERHUB_LOGIN_SERVER }}   # dockerhub 주소(docker.io) 
          username: ${{ secrets.DOCKERHUB_USERNAME }}           # dockerhub 로그인 계정
          password: ${{ secrets.DOCKERHUB_PASSWORD }}           # dockerhub Access 토큰 : PAT

      - name: 'Build & Tag Image'             # docker build, tag 설정     
        run: |
          docker build -t ${{ secrets.DOCKERHUB_LOGIN_SERVER  }}/${{ env.IMAGE_NAME }}:${{ env.VERSION }} -f Dockerfile .
          docker tag ${{ secrets.DOCKERHUB_LOGIN_SERVER  }}/${{ env.IMAGE_NAME }}:${{ env.VERSION }} ${{ env.REPO_ACCOUNT  }}/${{ env.IMAGE_NAME }}:${{ env.VERSION }}

      - name: 'Push image'                    # 이미지 푸시를 위해 dockerhub에 로그인 
        uses: docker/login-action@v1          # docker login <login-server> -u <username> -p <password> 내부에서 처리
        with:
          login-server: ${{ secrets.DOCKERHUB_LOGIN_SERVER  }}
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_PASSWORD }}

      - run: |                               # dockerhub에 이미지 푸시 : docker push ydcho0902/k8s-edu:202501191123     
          docker push ${{ env.REPO_ACCOUNT }}/${{ env.IMAGE_NAME }}:${{ env.VERSION }}

      - name: Update Kubernetes resources    # kubernetes 배포 파일에서 docker 이미지 태그 업데이트 
        run: |
          echo "`ls`"
          cd manifests/overlays/prod        
          kustomize edit set image ${{ env.REPO_ACCOUNT }}/${{ env.IMAGE_NAME }}:${{ env.VERSION }}
          cat kustomization.yaml
 # kustomize edit set image : prod DIR 아래 kubernetes 리소스 설정 파일에서 docker 이미지 태그 변경 (예:ydcho0902/k8s-edu:202501191123 로.)
      
      - name: Commit files   # kubernetes 설정 파일 변경 사항을 커밋하고 GitHub에 푸시
        run: |
          cd manifests
          echo "`ls`"
          git config --global user.email "github-actions@github.com"
          git config --global user.name "github-actions"
          git commit -am "Update image tag"
          git push -u origin main
```
- Workflow 생성 페이지 진입 경로 : Action 선택 > New workflow 선택 > Java with Gradle 선택
  ![Repository secrets list](/lecture8-1/img/lecture8-1-git-act-ci01.png)
- Workflow 생성 페이지 : github default 템플릿
  ![Repository secrets list](/lecture8-1/img/lecture8-1-git-act-ci02.png)
- Workflow 생성 : ci.yaml 파일 복사하여 내용 수정 권장
  ![Repository secrets list](/lecture8-1/img/lecture8-1-git-act-ci-yml.png)

## 1.3 Workflow 실행 : CI
- Workflow 실행 : Action 선택 > Job CI With Gradle > Run workflow > Job CI With Gradle 재선택 > 실행 Job
  ![Workflow 경로](/lecture8-1/img/lecture8-1-git-act-ci-run00.png)
- Workflow 실행 모니터링 : Action 선택 > Job CI With Gradle > Run workflow > Job CI With Gradle 재선택 > 실행 Job
  ![Workflow 실행 모니터링](/lecture8-1/img/lecture8-1-git-act-ci-mon01.png)
- kubernetes 배포 파일 docker 이미지 태그 확인
  ![kustomize 이미지 tag](/lecture8-1/img/lecture8-1-git-act-kz-01.png)
- 이미지 레지스트리 push된 이미지 버전 확인 : duckerhub Tag & github kustomization.yaml 내 이미지 Tag
  ![dockerhub 이미지 Tag](/lecture8-1/img/lecture8-1-git-act-docker01.png)

  
# 2. ArgoCD 설정 : CD

## 2.1 Application 설정
```dtd

#. 컨테이너 배포(동기화) 정책 : SYNC POLICY 
   o Manual, Automatic 중 수기 배포(Manual) 선택

#. kubernetes 배포 파일 관리
   o Repository URL : github repository 주소 선택 (없으면 Settings > Repositories > 추가 등록)
   o 참고 : lecture8 > z-lec8.md > 4.3 ArgoCD 설정 참조

#. kubernetes 리소스 배포 도구 선택
   o Kustomize

#. 배포 이미지 버전(Tag) 확인
   o 버전 잘못된 경우 : manifests/overlays/prod/kustomization.yaml 수정 후 git push 후 Sync

```
![dockerhub 이미지 Tag](/lecture8-1/img/lecture8-1-argocd-app.png)

## 2.2 서비스 배포(Sync)
![dockerhub 이미지 Tag](/lecture8-1/img/lecture8-1-argocd-sync.png)

## 2.3 서비스 배포 모니터링
![dockerhub 이미지 Tag](/lecture8-1/img/lecture8-1-argcd-mon.png)
