# Docker 사용 메뉴얼

# 목차
- [Docker 사용 메뉴얼](#docker-사용-메뉴얼)
- [목차](#목차)
- [도커란?](#도커란)
- [WSL 설정하기 (Window에서 도커를 사용하는 경우)](#wsl-설정하기-window에서-도커를-사용하는-경우)
- [도커 기본 명령어](#도커-기본-명령어)
  - [이미지 관련 명령어](#이미지-관련-명령어)
  - [컨테이너 관련 명령어](#컨테이너-관련-명령어)
- [도커 사용 예제](#도커-사용-예제)
  - [Base 이미지 설치하기](#base-이미지-설치하기)
  - [도커 네트워크 생성하기](#도커-네트워크-생성하기)
  - [Dockerfile을 사용하여 이미지 편집하기](#dockerfile을-사용하여-이미지-편집하기)
    - [mariaDB 이미지 편집하기](#mariadb-이미지-편집하기)
    - [JAVA 이미지 편집하기 - Spring 프로젝트](#java-이미지-편집하기---spring-프로젝트)
    - [Nodejs 이미지 편집하기 - React 프로젝트](#nodejs-이미지-편집하기---react-프로젝트)
    - [서버 실행하기](#서버-실행하기)
- [AWS에 배포하기](#aws에-배포하기)
  - [awscli 설치하기](#awscli-설치하기)
  - [EC2 키 파일을 private key로 만들기 및 접속하기](#ec2-키-파일을-private-key로-만들기-및-접속하기)
  - [EC2 내부 서버에서 도커 환경 세팅하기 (ubuntu 기준)](#ec2-내부-서버에서-도커-환경-세팅하기-ubuntu-기준)
    - [도커 설치 및 옵션 설정](#도커-설치-및-옵션-설정)
    - [도커 허브에 이미지 저장하기](#도커-허브에-이미지-저장하기)
    - [도커 허브에서 이미지 불러오기](#도커-허브에서-이미지-불러오기)
    - [도커 네트워크 설정하기](#도커-네트워크-설정하기)
    - [번외) 생각해보기](#번외-생각해보기)


# 도커란?

`도커`는 리눅스 상에서 컨테이너 방식으로 프로세스를 격리해서 실행하고 관리할 수 있도록 도와주는 컨테이너 기반 가상화 플랫폼이다. 하나의 게스트 OS 위에서 동작시키는 무거운 가상머신과 다르게, 도커는 호스트 OS의 커널을 공유하면서 다른 프로세스와 격리된 환경에서 애플리케이션과 필요한 라이브러리, 의존성 등을 “**이미지**”라는 개념으로 함께 패키징하여 프로세스를 “**컨테이너**”라는 단위로 경량화하여 실행한다는 점이 가상머신과 차별화된 매우 큰 장점이다.

이외에도 도커의 장점을 나열하자면 아래와 같다.

1. **빠른 배포**: 컨테이너를 사용하여 애플리케이션과 의존성을 패키징하므로, 개발 환경과 운영 환경 간의 차이를 최소화하고, “도커 허브”를 사용하여 애플리케이션을 빠르게 배포할 수 있다.
2. **프로그램 확장성**: 컨테이너를 사용하면 필요에 따라 애플리케이션을 쉽게 확장할 수 있다. 필요한 만큼의 컨테이너를 추가하거나 삭제하여 리소스를 효율적으로 활용할 수 있다.
3. **격리**: 각 컨테이너는 독립적인 공간에서 실행되기 때문에 서로 영향을 미치지 않고 독립적으로 동작하여, 프로그램 간 충돌로부터 안전을 보장한다.
4. **표준화**: 도커는 컨테이너를 표준화하여, 해당 시스템에 도커 엔진만 존재한다면 다양한 환경에서 동일한 방식으로 애플리케이션을 실행할 수 있다.
5. **운영 및 관리의 용이성**: 도커는 컨테이너를 이미지로 관리하기 때문에, 프로그램 버전 관리 및 롤백 등이 용이하다.

# WSL 설정하기 (Window에서 도커를 사용하는 경우)

window에서 linux를 사용하기 위해 WSL을 사용하게 되는데, WSL에서 docker를 사용하려면 추가적인 세팅이 필요하다. 이 문서는 컴퓨터에 ubuntu 환경의 wsl이 설정되어 있다는 것을 전제로 설명한다. 

1. 먼저 바이오스에서 가상화를 활성화해야 한다. 그리고 `windows 기능 켜기/끄기` 에서 `Linux용 Windows 하위 시스템` 을 체크해준다.
2. windows용 `Docker Desktop`을 다운로드 받는다. 
3. 설치 후 설정 화면에서 `General` 탭에서 `Use the WSL 2 based engine`을 활성화한다. 그리고 `Resource - WSL Integration - Enable integration with additional distros`에서 ubuntu를 체크해준다.
4. 이제 우분투에 들어가서 docker가 인식되는지 확인해보자.

```
# docker --version
```

# 도커 기본 명령어

## 이미지 관련 명령어

- 보유 중인 이미지 확인

```
# docker images
```

- 이미지 실행

```
# docker run <이미지_이름>
```

- 이미지 삭제

```
# docker rmi <이미지_이름>:<이미지_태그>
```
Image ID로도 삭제 가능하다.

- 이미지 빌드

```
# docker build -t <새_이미지_이름> <저장할_경로>
```

- 이미지 커밋

```
# docker commit <컨테이너_아이디> <이미지_이름>:<태그>
```
이미지를 커밋하면 이전 이미지에서 변경된 사항을 반영하여 로컬 이미지 저장소에 새 이미지로 생성함.

## 컨테이너 관련 명령어

- 현재 실행 중인 컨테이너 확인

```
# docker ps
```
-a 옵션을 사용하면 동작 중지된 컨테이너까지 볼 수 있다.

- 백그라운드에서 실행 중인 컨테이너와 통신할 shell 인터페이스를 만듦

```
# docker exec -it <컨테이너_이름> /bin/bash
```

주로 DB 서버와 shell 통신할 때 활용한다.

- 컨테이너 정지

```
# docker container stop <CONTAINER_ID>
```

- 컨테이너 실행 목록에서 삭제

```
# docker rm <CONTAINER_ID>
```

컨테이너를 정지한다고 해서 끝나는 것이 아니다. 컨테이너 실행 목록에서 완전히 제거해야 끝이다.

- 실행 중인 프로세스의 로그를 실시간으로 확인

```
# docker logs mariadb -f
```

- 실행했던 이미지와 현재 상태의 차이점 확인

```
# docker diff <컨테이너_아이디>
```

# 도커 사용 예제

이번에 Spring과 MariaDB를 사용하여 AWS에 업로드하는 예제를 다뤄본다. 자신의 서버 상황에 맞춰 자유롭게 수정해서 사용하자.

## Base 이미지 설치하기

java와 mariaDB의 official image를 설치하자.

```
# docker pull openjdk:11
11: Pulling from library/openjdk
001c52e26ad5: Pull complete
d9d4b9b6e964: Pull complete
2068746827ec: Pull complete
9daef329d350: Pull complete
d85151f15b66: Pull complete
66223a710990: Pull complete
db38d58ec8ab: Pull complete
Digest: sha256:99bac5bf83633e3c7399aed725c8415e7b569b54e03e4599e580fc9cdb7c21ab
Status: Downloaded newer image for openjdk:11
docker.io/library/openjdk:11
```

```
# docker pull mariadb:lts
lts: Pulling from library/mariadb
1bc677758ad7: Pull complete
196e1740aea4: Pull complete
3d4df0997938: Pull complete
9fa4e4184824: Pull complete
a134c78ed13c: Pull complete
d7cd475067d8: Pull complete
348ae7ff2d74: Pull complete
2e83ab58cb77: Pull complete
Digest: sha256:b0cf417b73294881d7a756f45d209c18d4ba9eefdfb5552627dbecd1ec03f8af
Status: Downloaded newer image for mariadb:lts
docker.io/library/mariadb:lts
```

각각 java openJDK 11버전과, mariaDB LTS 버전을 설치하는 것이다. 각 이미지가 설치되었는지 확인해보자.

```
# docker images
```

다운로드 받은 이미지들이 잘 보이면 성공이다.

## 도커 네트워크 생성하기

`도커 네트워크`는 각각의 격리되어 있는 컨테이너들이 서로 네트워크 통신을 할 수 있도록 도커 내부에서 네트워크 망을 만들어 주기 위해 필요하다. 사실 도커 네트워크는 클라우드와 같이 실제로 돌아갈 서버 컴퓨터에서 설정해줘야 하는 내용이다. 하지만 우선 작업 컴퓨터에서도 테스트를 해보기는 해야 하니, 연습 겸 세팅해보도록 하자. 

우선 도커 네트워크를 만들자.

```python
$ docker network create springboot-server
69bf5b8418204738d0ec2eab07f28245840c261db89fa12fbef822dfdcfe1a9c
```

## Dockerfile을 사용하여 이미지 편집하기

### mariaDB 이미지 편집하기

Dockerfile을 사용하면 DB 서버 환경 설정부터 db를 생성하고 table을 생성하는 것까지 전부 자동화할 수 있다. 그래서 아무것도 없는 새 컴퓨터에 이 편집된 이미지를 다운받아서 도커 엔진 위에서 실행시키면, 지가 알아서 DB 환경 세팅부터 db 생성 및 table 생성까지 자동으로 다 해 놓는다.

Dockerfile은 아래와 같이 작성한다.

```
FROM mariadb

ENV MYSQL_USER=root
ENV MYSQL_ROOT_PASSWORD=1234
ENV MYSQL_DATABASE=mydb

COPY create_tables.sql /docker-entrypoint-initdb.d/

# Specify the volume at runtime
VOLUME /var/lib/mysql

CMD ["--character-set-server=utf8mb4", "--collation-server=utf8mb4_unicode_ci"]
```

1. FROM mariadb
    
    이미지 빌드 과정에서 Base 이미지를 “mariaDB” 이름의 이미지로 세팅한다. 아까 다운받은 그 Base 이미지다.
    
2. ENV
    
    이미지 빌드 과정에서 mariaDB의 주요 환경 변수를 설정한다. MYSQL_USER, MYSQL_ROOT_PASSWORD, MYSQL_DATABASE를 입력해두었다.
    
3. COPY create_tables.sql /docker-entrypoint-initdb.d/
    
    이미지 빌드 과정에서 현재 Dockerfile과 같은 디렉토리에 존재하는 “create_tables.sql” 이름의 파일을 복사하여, 컨테이너 내의 “/docker-entrypoint-initdb.d/” 경로에 저장한다. 이 경로는 도커 엔진 위에서 초기에 mariaDB 컨테이너가 실행되는 순간 자동으로 파일이 실행되는 경로여서, 해당 이미지가 실행되면 자동으로 SQL 스크립트를 실행하여 테이블을 생성할 것이다. 
    
    복사할 “create_tables.sql” 파일의 SQL 스크립트는 아래와 같이 작성한다. 기존의 SQL 스크립트 문법과 동일하므로, 필요에 따라 자유롭게 편집해서 쓰면 된다.
    
    ```
    CREATE DATABASE IF NOT EXISTS projectqr;
    USE projectqr;
    
    CREATE TABLE IF NOT EXISTS menuTbl (
      id INT AUTO_INCREMENT PRIMARY KEY,
      foodName CHAR(30),
      price INT
    );
    
    CREATE TABLE IF NOT EXISTS orderTbl (
      id INT AUTO_INCREMENT PRIMARY KEY,
      foodName CHAR(30),
      numberOfFood CHAR(30)
    );
    ```
    
4. VOLUME /var/lib/mysql
    
    이미지 빌드 과정에서 컨테이너 내부의 `로컬 마운트` 설정을 해둔다. “docker run” 명령 단계에서 로컬 마운트 설정을 완료하면, 이 설정을 통해 컨테이너가 종료되어도 해당 PC에 모든 데이터들을 저장해두기 때문에 데이터가 영구적으로 보존된다.
    
    host에 저장할 위치는 “**docker run**” 의 -v 옵션에서 넣어줘야 한다. 관련 내용은 후에 후술한다.
    
5. CMD ["--character-set-server=utf8mb4", "--collation-server=utf8mb4_unicode_ci"]
    
    컨테이너가 성공적으로 실행되었을 때 매크로 명령으로 위 설정을 적용한다. 이 설정을 해줘야 DB에 한글 문자열을 깨지지 않게 넣을 수 있다.
    

이제 이미지를 빌드하여 나만의 이미지를 만들어보자. 

먼저 Dockerfile이 있는 디렉토리로 이동해야 한다. 만약 C드라이브의 "dockerProject" 폴더에 존재하는 경우, 아래 명령어로 이동하면 된다.

```
# cd /mnt/c/dockerProject
```

```
# docker build -t mydb .
```
생성된 이미지는 “**docker images**” 명령어를 통해 확인할 수 있다.

이 명령어는 현재 Dockerfile이 존재하는 디렉토리에 “mydb”라는 이름의 이미지로 빌드하겠다는 의미이다.

### JAVA 이미지 편집하기 - Spring 프로젝트

먼저 JAVA 이미지 편집 전에, 도커 네트워크에 맞춰 SpringBoot 내부의 “application.properties” 파일을 수정해야 한다.

```
spring.datasource.driverClassName=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://mydb:3306/projectqr
spring.datasource.username=root
spring.datasource.password=1234
```

여기서 url의 도메인에 주목하라. 도커 네트워크 안에서는 컨테이너 이름을 마치 해당 컨테이너의 도메인 주소처럼 사용할 수 있다. 도커 컨테이너가 실행될 때 내부 ip 주소가 가변적으로 바뀌기 때문에 반드시 컨테이너 이름으로 url을 설정해줘야 한다.

그리고 IDE에서 빌드를 하여 .jar 파일을 생성한다. (상황에 따라 .war 파일을 써야 한다면 그렇게 빌드하라.) 우리는 이 파일을 Java Base 이미지에 포함시켜서 나만의 이미지로 만들 것이다. gradle을 사용한다면 아래 명령어를 통해 Spring에서 jar 파일을 빌드하면 된다.
```
./gradlew build
```
빌드된 jar 파일은 "<프로젝트_파일_이름> - build - libs" 폴더 내에 있다.

Dockerfile은 아래와 같이 작성한다.

```
FROM openjdk:11

# Set the working directory
WORKDIR /home/api-server

# Copy the JAR file to the container
COPY dockerProject.jar /home/api-server/dockerProject.jar

# Set the environment variable
ENV SPRING_PROFILES_ACTIVE=dev

# Expose the desired port
EXPOSE 8080

# Set the entry point
ENTRYPOINT ["java", "-jar", "/home/api-server/dockerProject.jar"]
```

1. FROM openjdk:11
    
    이미지 빌드 과정에서 “openjdk:11” 이름의 Base 이미지를 기반으로 편집하겠다는 뜻.
    
2. WORKDIR /home/api-server
    
    이미지 빌드 과정에서 컨테이너 내부의 작업 디렉토리를 “/home/api-server”로 설정하겠다는 뜻. 이 디렉토리에 jar 파일을 저장해둘 거임.
    
3. COPY dockerProject.jar /home/api-server/dockerProject.jar
    
    이미지 빌드 과정에서 Dockerfile과 같은 디렉토리 내의 “dockerProject.jar” 파일을 컨테이너 내의 “/home/api-server/dockerProject.jar”로 복사하여 저장해두겠다는 뜻. 아까 빌드했던 jar 파일 명을 "dockerProject.jar"로 변경하거나, 여기서 해당 파일 명으로 변경한다.
    
4. ENV SPRING_PROFILES_ACTIVE=dev
    
    이미지 빌드 과정에서 환경변수 “SPRING_PROFILES_ACTIVE”를 dev 프로파일로 설정. 환경 변수를 설정하면 애플리케이션은 해당 환경 변수를 이용하여 개발 환경에 맞는 설정을 가져온다. dev 프로파일은 주로 DB 연결 설정, 보안 설정, 디버깅 설정 등을 위해 개발자가 활용한다고 한다. (일단 난 아님)
    
5. EXPOSE 8080
    
    도커 네트워크 상에서 내부 포트 8080을 오픈한다. 하지만 host의 외부 포트 8080번을 열겠다는 뜻은 아님.
    
6. ENTRYPOINT ["java", "-jar", "/home/api-server/dockerProject.jar"]
    
    이 컨테이너가 시작하는 순간 입력할 자동 명령을 세팅함. 당장 위 명령만 보면 “java -jar /home/api-server/dockerProject.jar” 명령을 실행하라는 의미.
    

이제 Dockerfile과 함께 나만의 이미지를 빌드하자. Dockerfile이 있는 디렉토리에서 아래 명령어를 입력한다.

```
# docker build -t api-server .
```

### Nodejs 이미지 편집하기 - React 프로젝트

window 환경의 WSL을 이용하여 npm을 활용하는건 너무 불안정하다. 문법 페러다임 충돌로 인해 제대로 작동하지 않는다. 

그래서 cmd를 활용하여 React 시작 파일을 다운로드받는다. 아래 명령어를 입력한 디렉토리에 설치되므로 디렉토리를 잘 선정해서 다운로드 받자. 파일 크기가 매우 크기 때문에, 되도록이면 작업 디렉토리에 다운받는다.

```
> npx create-react-app my-app
...
We suggest that you begin by typing:

  cd my-app
  npm start

Happy hacking!
```

이제 프론트 개발자 분이 만든 파일을 이식해야 한다. “my-app” 이름의 디렉토리 내의 “src”폴더를 바꿔치기하면 끝난다.

Dockerfile은 아래와 같이 작성한다.

```
FROM node:lts-alpine

WORKDIR /my-app

COPY package*.json ./

RUN npm install

COPY . ./

RUN npm run build

EXPOSE 3000

CMD [ "npm", "start" ]
```

1. FROM node:lts-alpine
    
    “node:lts-alpine”라는 Base 이미지로부터 편집한다. 이 이미지가 없으면 지가 알아서 도커 허브에서 다운로드 받아온다.
    
2. WORKDIR /my-app
    
    이미지 빌드 과정에서 이 도커 이미지의 작업 디렉토리를 “/my-app”으로 설정한다.
    
3. COPY package*.json ./
    
    이미지 빌드 과정에서 패키지 설정 파일인 “package.json”과 “package-lock.json”(또는 yarn.lock) 파일을 작업 디렉토리에 복사해서 저장한다.
    
4. RUN npm install
    
    이미지 빌드 과정에서 npm을 설치하도록 한다. 그래서 되도록이면 클라우드 서버의 OS는 리눅스 계열로 정하자.
    
5. COPY . ./
    
    이미지 빌드 과정에서 현재 작업 PC에 남아있는 프로젝트 파일들을 도커 이미지의 작업 디렉토리에 전부 복사하여 저장한다.
    
6. RUN npm run build
    
    이미지 빌드 과정에서 React 앱을 빌드하도록 한다.
    
7. EXPOSE 3000
    
    도커 네트워크의 내부 포트 3000번을 이 컨테이너의 포트로 개방한다. 구체적인 도커 네트워크 선택은 “docker run” 명령에서 진행한다.
    
8. CMD [ "npm", "start" ]
    
    컨테이너가 성공적으로 실행되었을 때, 매크로 명령으로 React 앱을 실행하도록 한다.
    

이제 Dockerfile로 나만의 이미지를 빌드하자.

```
# docker build -t kimdm/frontend .
```

### 서버 실행하기

아래 명령어를 통해 새로 빌드한 mariaDB 이미지를 실행할 수 있다. “**docker run**” 명령에서 관리하는 옵션들을 Dockerfile에서 자동화를 할 수 없기 때문에 어쩔 수 없이 필수 옵션들을 추가해서 입력해줘야 한다.

```
# docker run -d --name mydb --network springboot-server -p 3307:3306 -v /mnt/c/dockerProject/data:/var/lib/mysql mydb
```

1. -d 옵션은 해당 컨테이너를 실행시킬 때 백그라운드에서 실행시키라는 옵션이다. 도커 언어로 표현하면 “Detach” 모드로 실행시키는 것. 일반적인 서버 프로그램들은 실행시키면 콘솔창에서 글씨들을 주르륵 써내려가면서, 해당 프로그램을 종료하지 않는 한 해당 shell의 입력을 막아버리기 때문에 이 옵션을 애용함.
2. —name 옵션은 컨테이너의 이름을 지정하는 옵션이다.
3. —network 옵션은 어떤 도커 네트워크 망에 해당 컨테이너를 실행시킬건지를 의미한다. 아까 만들어둔 네트워크 망에 넣어주자.
4. -p 옵션은 포트 설정이다. 포트는 host를 기준으로 외부와 내부로 나뉜다. 컨테이너끼리는 내부 포트를 이용하며, 외부 클라이언트와 컨테이너 간의 통신은 host를 거쳐서 이뤄지므로 외부 포트를 사용한다. 하지만 진짜 mariaDB가 설치된 컴퓨터에서 해당 컨테이너의 외부 포트를 3306으로 지정하는 경우 기존 mariaDB와 충돌하기 때문에, 이 포트를 피해서 대충 아무거나 설정해줘야 한다.
    
    하지만 프론트나 백 서버의 경우에는 외부 포트도 신중하게 오픈해야 한다. 로드 벨런서를 사용하는 경우 로드 벨런서와 host가 통신하는 방식은 외부 포트 통신인데, 로드 벨런서는 해당 서버의 상태를 파악하기 위해 실시간으로 접속하여 정상 호출이 오는 지를 파악한다. 그래서 외부와 연결될 수 있도록 외부 포트 열어둬야 함.
    
5. -v 옵션은 `로컬 마운트` 디렉토리 설정 옵션이다. “host_디렉토리 : 컨테이너_디렉토리” 형식의 문법을 갖춘다.

마찬가지로 새로 빌드한 JAVA 이미지를 실행해보자.

```
# docker run -it --rm --name api-server-jdk --network springboot-server -p 8080:8080 api-server
```

1. -it 옵션은 shell 인터페이스를 통해 쌍방향 통신을 하라는 옵션이다. 명령을 주고 로그를 받을 수 있게 만든다는 의미이다.
2. —rm 옵션은 shell 인터페이스를 종료하면 해당 컨테이너를 종료하다는 옵션이다.

Nodejs 이미지도 실행하자.

```
# docker run -d --name react-app-container --network springboot-server -p 3000:3000 kimdm/frontend
```

이제 POST MAN을 활용하여 서버끼리 서로 API 통신하는 모습을 테스트해보자.

# AWS에 배포하기

AWS 서버를 만드는 방법은 다른 문서에서 다뤘다. 거기서 참고하기 바람.

## awscli 설치하기

작업 PC 환경에서 AWS 서버에 ssh 통신 방식으로 접근하기 위해서는 awscli(AWS Command Line Interface)를 설치해야 한다. 

```
# apt install awscli
```

버전 확인을 통해 잘 설치되었는지 확인해보자.

```
# aws --version
```

## EC2 키 파일을 private key로 만들기 및 접속하기

설치가 되었다면, 우선 EC2 서버가 ubuntu라는 것을 기준으로 “/home/ubuntu” 디렉토리로 이동한다. 여기에 EC2 서버의 암호화 키 파일을 저장해야, 이 파일을 private key로 변환할 수 있다. 반드시 public이 아닌 private key로 인식되어야 AWS 서버에 접속할 수 있다.

아래의 순서를 따르면 문제없이 해당 키 파일을 private key로 변경할 수 있을 것이다.

```
# cd
# mkdir /home/ubuntu
# chmod 400 <키_파일_이름>
# ls -l <키_파일_이름>
-r-------- 1 root root 1678 May 29 14:46 <키_파일_이름>
```

그러면 아래 명령어를 통해 shell 통신으로 인스턴스 서버에 접속할 수 있다.

```
# ssh -i awspw.pem ubuntu@{가변 퍼블릭 ip 주소}
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.19.0-1025-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon May 29 05:48:05 UTC 2023

  System load:  0.0               Processes:             95
  Usage of /:   20.6% of 7.57GB   Users logged in:       0
  Memory usage: 23%               IPv4 address for eth0: 172.31.21.104
  Swap usage:   0%

...
ubuntu@ip-172-31-21-104:~$
```

1. -i 옵션에 키 파일 이름을 넣어준다. EC2 서버를 만들면서 생성했던 키 파일인 “awspw.pem”이 그것이다.
2. ubuntu 서버로 인스턴스 서버를 열었다면, 유저 이름은 ubuntu가 디폴트 값이다.
3. EC2 인스턴스 서버의 퍼블릭 ip 주소를 넣어준다.

## EC2 내부 서버에서 도커 환경 세팅하기 (ubuntu 기준)

이제 본격적으로 EC2 인스턴스 서버에 도커를 설치하고 컨테이너를 실행시켜보자.

### 도커 설치 및 옵션 설정

```
$ sudo apt update
$ sudo apt install docker.io
$ docker -v
Docker version 20.10.21, build 20.10.21-0ubuntu1~22.04.3
```

그리고 추가적인 보조 옵션을 설정해주자.

```
$ sudo service docker start
$ sudo usermod -aG docker ubuntu
$ sudo systemctl enable docker
```

1. 도커 실행.
2. 도커 그룹에 “ubuntu” 유저 추가. 이렇게 해야 작업 PC에서 EC2 서버에 ssh 접속을 했을 때 도커를 바로 제어할 수 있어 편리해진다. 참고로 작업 PC에서 ssh로 접속할 때 디폴트 유저가 “ubuntu”이다. 
3. 인스턴스 서버 부팅 시에 자동으로 도커가 실행되도록 한다.

### 도커 허브에 이미지 저장하기

아까 만들었던 나만의 이미지들을 도커 허브에 올리자.

일단 작업 PC로 돌아오자.

```
# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username:  // 여기에 내 도커 계정의 닉네임을 입력
Password:  // 내 도커 계정의 비밀번호를 입력
Login Succeeded
```

로그인에 성공했으면 일단 이미지 이름을 개인에 맞춰 편집해줘야 한다. 그렇지 않으면 나중에 이미지를 끌어올 때 내 이미지가 아닌 공식 이미지가 깔릴 수 있기 때문. 

```
// # docker tag <local-image> <docker-hub-username>/<repository-name>:<tag>
# docker tag mydb kimdm/mydb:latest
# docker tag api-server kimdm/api-server:latest
```

아, 만약 이미지를 생성할 때 이미 이런 양식으로 생성했으면 이 과정은 건너 뛰어도 된다.

“docker images” 명령어를 통해 이미지가 새로운 이름으로 복사된 것을 볼 수 있다. (이미지 ID가 동일한 것을 볼 수 있다.)

이제 진짜로 도커 허브에 이미지를 저장하자.

```
// # docker push <docker-hub-username>/<repository-name>:<tag>
# docker push kimdm/mydb:latest
# docker push kimdm/api-server:latest
# docker push kimdm/frontend:latest
```

그리고 [자기 계정의 도커 허브 주소](https://hub.docker.com/u/kimdm) 에 접속하여 실제로 업로드가 되었는지 확인해보자. (로그인 하기는 너무 빡세서 본인 url 타고 접속하는 것이 속 편함. 비로그인 사용자도 볼 수 있음.)

### 도커 허브에서 이미지 불러오기

다시 EC2 인스턴스 서버 ssh 통신 shell로 돌아오자. 도커 로그인을 한 후, 도커 허브에서 아까 올린 이미지를 불러올 것이다.

```
$ sudo su
# rm /root/.docker/config/.json
# docker login
# cat /root/.docker/config/.json  
```

1. 모든 명령을 root 명령으로 변환.
2. 보안을 위해서 기존의 암호화된 자격 증명을 일부러 폐기. 로그인 후 다시 확인해보면 새로 발급된 것을 볼 수 있다.

이제 이미지들을 불러오자.

```
# docker pull kimdm/mydb:latest
# docker pull kimdm/api-server:latest
# docker pull kimdm/frontend:latest
# docker images
```

그러면 내가 편집했던 이미지들이 해당 인스턴스 서버에 깔려 있는 것을 볼 수 있다. 이제 아까 했던 것과 동일하게 각 이미지들을 실행시켜 컨테이너를 만들면 된다!

### 도커 네트워크 설정하기

인스턴스 서버에서도 다시 도커 네트워크를 구축해줘야 한다.

```
# docker network create springboot-server
```

그리고 이제 다운받은 도커 이미지들을 실행하면 된다.
- mariaDB
    ```
    # docker run -d --name mydb --network springboot-server -p 3307:3306 -v /mnt/c/dockerProject/data:/var/lib/mysql mydb
    ```
- Spring
    ```
    # docker run -d -it --rm --name api-server-jdk --network springboot-server -p 8080:8080 api-server
    ```
- Nodejs
    ```
    # docker run -d --name react-app-container --network springboot-server -p 3000:3000 kimdm/frontend
    ```

각각의 내부 ip는 아래 명령어를 통해 확인할 수 있다.

```
# docker inspect <컨테이너_아이디>

...
            "Networks": {
                "springboot-server": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "76a..."
                    ],
                    "NetworkID": "30...",
                    "EndpointID": "c5...",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.4",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:...",
                    "DriverOpts": null
                }
            }
        }
    }
]
```
하지만 도커 네트워크 내부 ip는 도커가 마음대로 할당하는 것이기 때문에, 항상 해당 이미지의 컨테이너가 해당 ip를 가질 것이라고 보장하지 못한다.

도커는 이 문제점을 해결하기 위해 "도커 컨테이너 이미지"로 주소를 설정해도 내부적으로 통신이 가능하도록 설계되었다.

- 아래는 nodeJS의 js 파일에 백엔드 서버의 API로 연결하기 위해 작성하면 되는 코드 예시이다.

    ```Javascript
    const backendContainerName = 'backend'; // 백 서버의 도커 컨테이너 이름
    const apiUrl = `http://${backendContainerName}:포트번호/your-api-endpoint`;

    /** 예시
     * const apiUrl = `http://${backendContainerName}:8080/api/data`;
     * /
    ```

### 번외) 생각해보기
1. **만약 프론트 서버와 백 서버가 서로 다른 도커 네트워크에 존재한다면, 백 서버 API를 얻기 위해서 프론트 서버 파일에 url을 어떻게 작성해야 하는가?**
   
    도커에서는 어떤 컨테이너를 다른 도커 네트워크에 연결하는 기능을 제공한다. 그러기 위해서는 아래 단계를 거쳐야 한다.

    1. 도커 컨테이너 간 통신을 위해 새로운 네트워크를 생성

        두 컨테이너는 서로 다른 도커 네트워크에 속해있기 때문에, 둘이 서로 통신할 수 있는 새로운 도커 네트워크를 생성해줘야 한다. 이렇게 컨테이너 간 통신을 위한 네트워크를 "사용자 정의 네트워크"라고 한다. 이건 앞서 도커 네트워크를 만들듯이 만들어주면 된다.
        ```
        # docker network create my-network
        ```
        이후 프론트 서버와 백 서버를 이 네트워크에 연결해주면 두 서버 간 통신이 가능해질 여지가 생긴다. (?)

    2. 도커 컨테이너 간 통신을 위한 볼륨 매핑

        하지만 두 서버는 서로 다른 네트워크에 있기 때문에 일반적인 네트워크 통신 방식으로는 통신할 수 없다. 이들을 통신하게 만들기 위해서는 "볼륨 매핑"이라는 것을 해줘야 한다. "볼륨 매핑"이란 호스트 시스템과 컨테이너 사이의 파일 시스템을 공유하는 것을 말하며, 볼륨 매핑을 함으로써 컨테이너가 호스트의 파일 시스템에 접근하는 권한을 얻을 수 있다. 이렇게 하면 호스트 시스템에 특정 디렉토리를 생성하고, 프론트 서버와 백 서버가 동시에 이 디렉토리를 볼륨 매핑하면 두 서버가 서로 다른 도커 네트워크에 있어도 파일 시스템 공유를 통해 통신이 가능해진다.

        "docker run" 명령의 -v 옵션에서 볼륨 매핑을 설정할 수 있다.

2. **만약 프론트 서버와 백 서버가 서로 다른 인스턴스 서버에 존재하고 두 서버는 하나의 로드 벨런서에 연결되어 있다면, 백 서버 API를 얻기 위해서 프론트 서버 파일에 url을 어떻게 작성해야 하는가? 로드 벨런서는 도메인을 발급받은 상태로 가정한다.**

    이때는 `로드 벨런서`를 적극 활용해야 한다. 애초에 로드 벨런서의 역할이 서로 다른 서버들을 트래픽을 제어하면서 적절히 통신을 시키는 것. 로드 벨런서 내부에 이미 각 서버에 대한 ip 정보가 다 저장되어 있기 때문에 오히려 두 서버를 통신하게 만들기 쉽다. 프론트 서버는 그냥 로드 벨런서의 도메인 이름으로 연결해두면 된다. 
    
    중요한 건 로드 벨런서의 세팅이다. ELB 기준으로 "Listeners and rules" 옵션창이 핵심이다. 여기서는 각 요청마다의 조건을 걸어두고, 해당 조건을 만족 시키면 특정 "타겟 그룹"으로 포워딩하게 설정할 수 있다. (관련 내용은 AWS ELB 편에서 참고) 이 성질을 활용하면 어렵지 않게 두 서버를 연결시킬 수 있다.