---
categories: [BackEnd, Github Action, CI/CD]
tags: [BackEnd, Github Action, Docker, docker-compose, CI/CD]
---

# Github 와 Github Actions을 이용한 EC2 배포 자동화

- 이전 글에서 `S3`배포 자동화와 환경변수 세팅에 대해서 알아 보았다.  
  참고 : [Github Actions으로 S3 배포 자동화 하기](https://yongjun96.github.io/posts/Github-Actions%EC%9C%BC%EB%A1%9C-S3-%EB%B0%B0%ED%8F%AC-%EC%9E%90%EB%8F%99%ED%99%94-%ED%95%98%EA%B8%B0/)

<br>

- 이번에는 `EC2`에 `Spring Boot`프로젝트를 `Docker`로 빌드해서 `AWS ECR`에서 이미지를 관리하고 `EC2`에서 `docker-compose`를 사용해서 서버를 실행하는 배포 자동화를 알아 보도록 하겠다.


<br>

---

### Spring Boot API서버 CI/CD 구성

<br>

![EC2 CI_CD](../assets/img/postimg/2024-04-24/EC2%20CI_CD.jpg)


**간단한 구성도**  

프로젝트 `root`경로에 위치하는 `Dockerfile`를 빌드시키고 `ECR`에 올린 후, 
`EC2`에서 `Redis`와 `pull`받은 `Spring`프로젝트를 `EC2`에 만들어 둔 `docker-compose`파일을 이용해서 
같은 네트워크에서 컨테이너를 만들어 실행 시킨다.


<br>

---

<br>

### GitHub Actions를 이용한 CD 구성

<br>

해당 프로젝트의 `GitHub Repository`로 접속해 `Actions` 탭을 클릭

<br>

![img.png](../assets/img/postimg/2024-04-24/actions.png)

<br>

`java with gradle`을 검색하고 `Configure` 클릭

<br>

![img.png](../assets/img/postimg/2024-04-24/java%20with%20gradle.png)

<br>

이름을 알맞게 입력해 주고 

<br>

![img.png](../assets/img/postimg/2024-04-24/yml이름.png)

<br>

`Commit changes...` 클릭 후, 


초기 커밋 메세지와 상세 설명입력 후 커밋해준다.

<br>

![img_1.png](../assets/img/postimg/2024-04-24/yml커밋.png)

<br>

여기끼지 `CI/CD`를 구성하기 위한 `yml`파일을 미리 만들어 두었다.  
다음으로는 `환경을 구성`하기 위한 `사전 작업`을 `진행`해야 한다.

<br>

---

<br>

#### Submodule 세팅

- 프로젝트에서 환경을 세팅해 주는 정보는 `민감정보`이기 때문에 `yml`파일을 그대로 노출해서 사용하면 <span style="color: #FF4500;">위험</span>하다.
- 그렇기 때문에 `private`로 관리할 수 있는 `Submodule`프로젝트를 하나 생성해서 `민감정보`를 포함하는 `yml`파일을 관리한다.  


- `GitHub Actions`에서 해당 프로젝트에 포함되어 있는 `Submodule`을 사용하려면 `Token`을 발급하고 발급받은 `Token`으로 접근 후, `초기화`를 진행해 줘야 한다.

<br>

---

<br>

##### Submodule private 프로젝트 생성
<br>

우선 `submodule`로 사용할 `private`프로젝트를 만들어 준다.

<br>

![img.png](../assets/img/postimg/2024-04-24/서브%20모듈%20생성.png)

<br>

1. `yongjun-store-submodule`로 이름을 한번에 알아볼 수 있도록 설정 
2. `private`로 비공개 설정  
3. 초기 커밋을 위해서 `README`를 생성

<br>

![img.png](../assets/img/postimg/2024-04-24/서브모듈내부.png)

<br>

위와 같이 `원하는 파일`만 올려두고 나머지 `필요없는 파일들은 전부 제거`해 준다.

<br>

그 후, `submodule`을 사용할 프로젝트에 `Git Bash`로 접속  
`submodule`이 필요한 경로까지 이동해 준다. 

<br>

![img.png](../assets/img/postimg/2024-04-24/git%20bash.png)

<br>

`git submodule add {submodule Repo URL}` 입력  

<br>

ex).
```shell
git submodule add https://github.com/yongjun96/yongjun-store-submodule.git
```

<br>

해당 명령어를 실행하게 되면 `root`경로에 `.gitmodules`파일이 생성된다.

<br>

![img.png](../assets/img/postimg/2024-04-24/생성깃모듈.png)

<br>

`submodule`의 `URL`과 `add`해준 경로가 `path`에 설정된다. 당연히 임의로 수정해도 작동한다.  
`path`는 `.gitmodules`이 존재하는 `root`경로를 기준으로 한다.  
그 후, `커밋/푸쉬`해 준다면 적용된다.

<br>

`submodule`프로젝트도 관리해줘야 하기 때문에
```shell
git clone https://github.com/yongjun96/yongjun-store-submodule.git
```

<br>

프로젝트를 `clone`해 주고
```shell
git submodule init

git submodule update
```

<br>

`submodule`을 초기화하고 `update`한다.  
**처음 딱 한번만 진행해 주면 된다.**

<br>

---

<br>

##### submodule update 내용 반영하기

<br>

`submodule`프로젝트도 엄연히 **독립적인** `Repository`로 관리가 되기 때문에, 변경 사항이 생겼을 때 <span style="color: #FF4500;">무작정 메인 프로젝트에서 수정해서 반영하면 안된다.</span>



![img.png](../assets/img/postimg/2024-04-24/메인반영안됨.png)

<br>

이런 식으로 메인 프로젝트에서 직접 변경 사항을 `커밋/푸쉬`하게 되면 `submodule`의 `Git History`가 누락되거나 꼬일 수 있기 때문에 하지 않도록 하자.

<br>

그렇다면 어떻게 해야 할까?

<br>

방법은 직접 `submodule`프로젝트로 접근해 변경 사항을 `커밋/푸쉬`해주면 된다.

![img.png](../assets/img/postimg/2024-04-24/서브모듈에서%20직접%20수정.png)

변경 사항을 `커밋/푸쉬`해 주었다면 메인 프로젝트에 `Git Bash`로 접근해서 `submodule`경로 까지 이동해 준다.

<br>

![img.png](../assets/img/postimg/2024-04-24/서부모듈%20경로이동.png)

<br>

이렇게 이동하면 밑줄 친 것 처럼 `변경 사항이 있다고 표시`된다.  

```shell
git fetch

git merge origin/main
```
<br>

위와 같이 `fetch`를 하고 `main`브런치에 `merge`하면 `submodule`의 변경 사항이 메인 프로젝트에 `merge`된다.

![img.png](../assets/img/postimg/2024-04-24/변경사항%20적용.png)


<br>

---

<br>

##### 운영 환경에 맞춰 yml 구성

<br>
 
그리고 운영 환경에 맞춰 `yml`을 다르게 구성해야 하는 경우

![img.png](../assets/img/postimg/2024-04-24/운영환경%20구성에%20맞춤.png)

<br>

현재 `yml`이 위치한 경로를 기준으로 `import`를 이용해 `submodule`의 `yml`파일을 여러개 가져와서 적용 시킬 수 있다.

<br>

---

<br>

##### Submodule Token 생성

<br>

프로젝트의 `Settings`가 아닌 계정의 `Settings`로 들어가 준다.

![img.png](../assets/img/postimg/2024-04-24/서브모듈%20토큰.png)

<br>

하단에 위치한 `Developer settings` 클릭

![img.png](../assets/img/postimg/2024-04-24/Developer%20settings%20클릭.png)

<br>

`Personal Access Tokens` - `Tokens (classic)` - `Generate new token (classic)` 클릭

<br>

![img.png](../assets/img/postimg/2024-04-24/뉴토큰클레식.png)

<br>

`note`에 토큰의 이름을 입력해 주고 `repo`권한만 체크해 준 후, 원하는 `Expiration` 토큰 기한을 설정해 준다.  
`No expiration`를 선택하면 기한 없이 계속 유효한 토큰이 생성되지만 보안상 추천하지 않는다.

<br>

![img.png](../assets/img/postimg/2024-04-24/토큰생성완료.png)

<br>

이렇게 발급하게 되면 토큰 값을 복사해 놓는다. 

<br>

---

<br>

#### 환경 변수 세팅

<br>

`Spring boot Repository`의 `Settings` - `Secrets and variables` - `Actions` 이동

![img_1.png](../assets/img/postimg/2024-04-24/환경변수%20세팅이동.png)

<br>

세팅해야 하는 `환경 변수`는 크게 3가지로 나뉜다.  
1. `ECR`에 접속하기 위한 `AWS_ACCESS_KEY_ID`와 `AWS_SECRET_ACCESS_KEY` / `AWS` 계정의 `AWS_ACCOUNT_ID` 및 `AWS_REGION`
2. `EC2`에 접근하기 위한 `EC2_HOST`, `EC2_USERNAME`, `EC2_PRIVATE_KEY`, `EC2_SSH_PORT`
3. `submodule`의 접근하기 위한 `SUBMODULE_TOKEN`

![img.png](../assets/img/postimg/2024-04-24/환경%20변수%20정보.png)

<br>

자세히 알아보면



1 . `AWS_ACCESS_KEY_ID` : `AWS` - `IAM 대시보드` - `사용자` - `사용 중인 사용자 선택` - `보안 자격 증명` - `엑세스 키 만들기` or `사용 중인 엑세스 키 확인`
으로 값을 받을 수 있다. `SECRET_ACCESS_KEY`를 까먹었을 경우 기존에 사용하던 엑세스 키를 삭제하고 다시 만들어야 한다... 꼭 안전하게 기록하고 보관하자.  


2 . `AWS_ACCOUNT_ID` : `AWS`의 해당 `계정 ID`를 복사해서 입력하면 된다.

![img_1.png](../assets/img/postimg/2024-04-24/계정%20ID.png)


3 . `AWS_REGION` : 자신이 사용하고 있는 리전을 입력하면 된다. 굳이 `환경 변수`로 사용하지 않아도 된다. 기본 적으로 한국을 사용할 것 이기 때문에 `아시아 태평양 (서울)`의 
`ap-northeast-2`를 입력한다.

4 . `AWS_SECRET_ACCESS_KEY` : `AWS_ACCESS_KEY_ID`와 한 쌍으로 `AWS_ACCESS_KEY_ID`를 발급 받을 때 **한번만** 값을 확인 할 수 있다. 발급 받은 `SECRET_ACCESS_KEY`를 입력해 준다.

5 . `EC2_HOST` : `EC2`에 접근할 때 필요한 `퍼블릭 IPv4 주소` 입력 ex). `127.0.0.1`

6 . `EC2_PRIVATE_KEY` : `EC2` 내부에 있는 `~/.ssh/id_rsa`의 값을 입력해야 한다. 아래 에서 자세히 서술 예정.

7 . `EC2_SSH_PORT` : `EC2`에 접근할 때 사용할 포트 번호를 지정한다. ex). `22`

8 . `EC2_USERNAME` : `centos`를 사용하고 있으므로 `centos`의 `username`인 `ec2-user`를 입력

9 . `SUBMODULE_TOKEN` : 위에서 생성한 `SUBMODULE_TOKEN`의 토큰 값을 입력해 주면 된다.

<br>

---

<br>

##### EC2_PRIVATE_KEY 확인 하기

<br>

**일단 해당 부분에서 좀 많은 시간을 소모했다.**  

`EC2_PRIVATE_KEY`로 사용하는 `id_rsa`와 `gitHub`의 접근을 허용해 주는 `SSH 공개 키`인 `id_rsa.pub`의 지식이 없어서 엄청나게 많은 시도를 하면서 커밋 내역이 쌓여 버렸다..  

뒤에 설명할 yml 파일 세팅에서 `push`했을 때 이벤트를 실행하도록 했기 때문에 `yml`을 수정하고 `push`하면서 커밋 내역이 엄청 쌓인 것이다.  

제대로 세팅되기 전에는 `Repository`를 `Fork`해서 테스트 해봤어야 했는데, 빨리 해결해야 된다는 마음에 그렇게 하지 못해서 아쉬운 부분이다.

<br>

---

<br>

먼저 `EC2`에 접속해야 한다.  
접속은 `AWS`에서 `EC2 인스턴스 연결`로 해도 되고 `xshell`이라는 프로그램을 이용해도 된다.

<br>

![img.png](../assets/img/postimg/2024-04-24/centos%20접속.png)

<br>


```shell
cd ~

cd .ssh
```
이동 해주면 기본적으로 `id_ed25519`나 `id_rsa`가 존재할 것이다. 나의 경우, `id_rsa`를 사용할 것이다.

<br>


```shell
cat id_rsa
```
해당 명령어로 `id_rsa`의 값을 복사해 준다.

<br>

주석을 포함해서 전부 복사해 준다.

-----BEGIN OPENSSH PRIVATE KEY-----  
...  
-----END OPENSSH PRIVATE KEY-----  

<br>

6번 `EC2_PRIVATE_KEY` 값은 해당 값이 들어가야 한다.  
`SSH`로 접근하려면 해당 `인스턴스`의 `비공개 SSH 키`인 `id_rsa`값이 필요하기 때문이다.

<br>

---

<br>

#### yml 작성

- 이제 사전 세팅은 끝났다.
- `GitHub Actions`를 사용하기 위해 `yml`에 `CI/CD`명령을 구성하면 된다.

<br>

```yml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: GitHub Action에서 현재 Repository를 체크아웃 및 서브모듈 사용
      uses: actions/checkout@v2
      with:
        submodules: 'recursive'
        token: {% raw %}${{ secrets.SUBMODULE_TOKEN }}{% endraw %}


    - name: Docker Buildx, 다중 플랫폼 이미지를 생성하고 관리하기 위해 필요한 Docker 환경을 설정
      uses: docker/setup-buildx-action@v1


    - name: 이미지 빌드
      run: docker build -t yongjun-store .


    - name: 이미지에 latest 태그 추가
      run: docker tag yongjun-store:latest {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store:latest


    - name: Login to AWS ECR
      env:
        AWS_ACCESS_KEY_ID: {% raw %}${{ secrets.AWS_ACCESS_KEY_ID }}{% endraw %}
        AWS_SECRET_ACCESS_KEY: {% raw %}${{ secrets.AWS_SECRET_ACCESS_KEY }}{% endraw %}
      run: aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com


    - name: Push Docker image to Amazon ECR
      run: docker push {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store:latest


    - name: SSH로 EC2를 접근해 명령 시작
      uses: appleboy/ssh-action@master
      with:
        host: {% raw %}${{ secrets.EC2_HOST }}{% endraw %}
        username: {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}
        key: {% raw %}${{ secrets.EC2_PRIVATE_KEY }}{% endraw %}
        port: {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %}
        script: |
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker stop $(docker ps -a -q)"
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker rm $(docker ps -a -q)"
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker rmi {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store"          
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com"
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker pull --pull always {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store:latest"
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker-compose -f /home/ec2-user/docker-compose.yml down && docker-compose -f /home/ec2-user/docker-compose.yml up -d"

```

구간 별로 잘라서 알아 보자.

<br>
<br>

보이는 대로 `main`브런치가 `push`될 때 작업을 수행한다는 뜻이다.  
`ubuntu`환경의 `최신 버전`에서 작업을 수행한다.

```yml
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
```

<br>
<br>
<br>


해당 구간은 `name`에도 적어 놓았듯 현재 `Repository`를 체크아웃하고 `recursive`로 `submodule`을 초기화 하고 `token`을 이용해 접근 권한을 얻는다.

```yml
    - name: GitHub Action에서 현재 Repository를 체크아웃 및 서브모듈 사용
      uses: actions/checkout@v2
      with:
        submodules: 'recursive'
        token: {% raw %}${{ secrets.SUBMODULE_TOKEN }}{% endraw %}
```

<br>
<br>
<br>

`buildx`는 다양한 플랫폼 및 아키텍처에 대한 빌드 작업을 수행할 수 있도록 `Docker CLI`를 확장하는 기능이다.
`컨테이너 이미지`를 `다양한 플랫폼`에서 사용할 수 있도록 빌드하고 배포하기 위해서 설정했다.

```yml
    - name: Docker Buildx, 다중 플랫폼 이미지를 생성하고 관리하기 위해 필요한 Docker 환경을 설정
      uses: docker/setup-buildx-action@v1
```

<br>
<br>
<br>

프로젝트 `root`경로에 존재하는 `Dockerfile`을 빌드하는 구간이다. 

```yml
    - name: 이미지 빌드
      run: docker build -t yongjun-store .
```

<br>

`Dockerfile`의 이름은 항상 `Dockerfile`이여야 한다. 여러 `Dockerfile`을 사용하고 싶다면 다른 경로에 `Dockerfile`을 생성해서 사용하면 된다.  

```dockerfile
FROM openjdk:17-alpine AS builder

COPY gradlew build.gradle settings.gradle ./
COPY gradle ./gradle
COPY src/main ./src/main

RUN chmod +x gradlew
# gradle 이 로컬에 설치되지 않아도 gradle을 사용할 수 있게 해줌
RUN ./gradlew clean build

FROM openjdk:17-alpine

COPY --from=builder /build/libs/yongjun-store-0.0.1-SNAPSHOT.jar /app.jar

ENTRYPOINT ["java", "-jar", "-Dspring.profiles.active=prod", "/app.jar"]
```

간단하게 `jar`를 빌드해서 `prod`환경으로 실행할 수 있도록 구성했다.

<br>
<br>
<br>


`이미지`의 이름을 `ECR`에서 `관리하는 이미지명`과 같도록 수정하고 태그 또한 항상 `latest`로 `push`하도록 설정해 주었다. 

```yml
    - name: 이미지에 latest 태그 추가
      run: docker tag yongjun-store:latest {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store:latest
```

<br>
<br>
<br>

`ECR`에 `Docker Image`를 `push`하기 위해 미리 세팅해 둔 `환경 변수`를 이용해서 접근해 준다.

```yml
    - name: Login to AWS ECR
      env:
        AWS_ACCESS_KEY_ID: {% raw %}${{ secrets.AWS_ACCESS_KEY_ID }}{% endraw %}
        AWS_SECRET_ACCESS_KEY: {% raw %}${{ secrets.AWS_SECRET_ACCESS_KEY }}{% endraw %}
      run: aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com
```

<br>
<br>
<br>

빌드한 `Docker Image`를 `push`

```yml
    - name: Push Docker image to Amazon ECR
      run: docker push {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store:latest
```

<br>
<br>
<br>

여기서 부터는 `SSH`로 `EC2`에 접근하여 `docker-compose`로 `redis`와 같은 네트워크를 공유하는 `Spring Boot`프로젝트를 실행한다.

```yml
    - name: SSH로 EC2를 접근해 명령 시작
      uses: appleboy/ssh-action@master
      with:
        host: {% raw %}${{ secrets.EC2_HOST }}{% endraw %}
        username: {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}
        key: {% raw %}${{ secrets.EC2_PRIVATE_KEY }}{% endraw %}
        port: {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %}
        script: |
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker stop $(docker ps -a -q)"
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker rm $(docker ps -a -q)"
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker rmi {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store"          
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com"
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker pull --pull always {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store:latest"
          ssh {% raw %}${{ secrets.EC2_USERNAME }}{% endraw %}@{% raw %}${{ secrets.EC2_HOST }}{% endraw %} -p {% raw %}${{ secrets.EC2_SSH_PORT }}{% endraw %} "sudo docker-compose -f /home/ec2-user/docker-compose.yml down && docker-compose -f /home/ec2-user/docker-compose.yml up -d"
```

<br>

1 . 먼저 현재 실행 중인 `컨테이너를 종료`시킨다.

 ```shell
sudo docker stop $(docker ps -a -q)
```
2 . 존재하는 `컨테이너를 전부 삭제`시킨다.

```shell
sudo docker rm $(docker ps -a -q)
```
3 . `ECR`에서 `pull`받았던 `이전 이미지를 삭제`시킨다.

```shell
sudo docker rmi {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store
```
4 . `ECR`에 로그인 해 준다.

```shell
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com
```
5 . `최신버전`의 `이미지`를 `pull`받는다.

```shell
sudo docker pull --pull always {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store:latest
```

6 . `docker-compose` 파일에 정의된 모든 `컨테이너`, `네트워크`, `볼륨`, 및 `기타 서비스`들을 `중지`하고 `제거`  
6-1 . `docker-compose`를 다시 실행
```shell
sudo docker-compose -f /home/ec2-user/docker-compose.yml down && docker-compose -f /home/ec2-user/docker-compose.yml up -d
```

<br>

---

<br>

##### docker-compose.yml 작성

<br>

- `/home/ec2-user/docker-compose.yml`해당 경로에 `docker-compose.yml`을 생성해서 관리한다.
- `docker-compose.yml`또한 이름은 `docker-compose.yml`고정으로 사용하며, 다른 `compose`파일을 사용하고 싶다면 경로를 다르게 설정해서 사용해야 한다.
<br>

```yml
version: '3.4'

services:
  redis:
    image: redis
    container_name: redis
    ports:
      - "6379:6379"
    networks:
      - my-network

  yongjun-store:
    image: {% raw %}${{ secrets.AWS_ACCOUNT_ID }}{% endraw %}.dkr.ecr.ap-northeast-2.amazonaws.com/yongjun-store
    container_name: yongjun-store
    ports:
      - "80:8081"
    networks:
      - my-network
    depends_on:
      - redis

networks:
  my-network:
    driver: bridge
```

<br>

1 . `redis`와 `yongjun-store`를 같은 네트워크 상에서 구동 시킬 수 있도록 하기위해 구성했다.  

2 . `image:`를 통해서 `image`의 이름을 지정해주고 `container_name:`으로 원하는 컨테이너 이름을 설정해 준다.  

3 . `ports:`로 사용하고 싶은 `port`를 지정해 주고, `networks:`를 이용해서 같은 `network`를 공유하도록 둘 다 같은 `my-network`를 입력해 주었다. 

4 . `depends_on:`를 이용해서 `yongjun-store`가 `redis`를 의존하도록 지정하고 `redis`서비스가 시작될 때까지 대기한 후에 시작 되도록 설정.  

5 . `networks:`설정으로 `my-network:`라는 네트워크를 생성하고, `driver:`를 사용하여 네트워크 유형을 `bridge`로 지정  

6 . `bridge`네트워크는 `Docker`의 `기본 네트워크 드라이버` 중 하나로, `여러 컨테이너가 동일한 네트워크에서 통신`할 수 있도록 해준다.

<br>

---

<br>

##### redis 설치

`redis`를 `EC2`서버에 직접 다운받아서 `이미지`를 관리하는 방식으로 사용하도록 구성했다.  

```shell
# redis가 존재하지 않으면 알아서 latest로 pull받고 6379포트로 실행
docker run --name redis -d -p 6379:6379 redis
# 간단하게 redis-cli를 실행하고, redis컨테이너에 연결하여 ping / pong 확인 해보기
docker run -it --link redis:redis --rm redis redis-cli -h redis -p 6379
```

<br>

`--link redis:redis`해당 옵션은 `구식`이기 때문에 사용을 권장하지는 않지만, 간단한 테스트를 할 때는 사용하기 좋다.  

<br>

---

<br>

### 마무리

<br>

모든 세팅이 끝났고 코드를 `push`해서 배포해 보자.  

![img.png](../assets/img/postimg/2024-04-24/정상적으로%20빌드%20성공!.png)

<br>

정상적으로 빌드가 성공한 모습.

<br>

![img.png](../assets/img/postimg/2024-04-24/정상빌드설공인증샷.png)

<br>

서버에서도 정상적올 `컨테이너`가 실행되었다.

<br>

![img.png](../assets/img/postimg/2024-04-24/스웨거%20접속확인.png)

<br>

`Swagger`에 접속도 성공하였다.

<br>

---

<br>

### 마무리

<br>

이번에 `CI/CD`를 `프론트`와 `백엔드` 전부 `jenkins`가 아닌, `GitHub Actions`을 통해서 구현해 보았다.  
사실 시행착오도 정말 많았고 모르는 부분 투성이 였기 때문에 시간도 엄청나게 많이 걸렸다.  

그럼에도 포기하지 않고 끝까지 완성했고, 조금 성장했구나 라는 뿌듯함도 느낄 수 있었다.  
앞으로 새로운 기술이나 기능들이 나왔을 때 바로 바로 활용하고 사용할 수 있게 실력도 키워야 하겠지만 두려워 하지 않고 시도할 수 있는 마음가짐도 중요하다는 것을 느꼈다.  

막연하게 "사용해 봐야지"라는 생각만 가지고 있어서는 아무것도 해결되지 않는다.  
구현할 용기가 있다면 죽이 되든 밥이 되든 뭐라도 나온다.  

생각만 하지말고, 보고만 있지말고 행동으로 옮기는게 중요한 것 같다. 










