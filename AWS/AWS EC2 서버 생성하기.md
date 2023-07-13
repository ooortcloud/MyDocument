# 목차
- [목차](#목차)
- [AWS EC2 서버 만들기](#aws-ec2-서버-만들기)
- [AWS EC2 서버에 접속하기](#aws-ec2-서버에-접속하기)


# AWS EC2 서버 만들기

1. 먼저 AWS 루트 계정을 생성한다. 
2. AWS 메뉴에서 `컴퓨팅 → EC2` 항목을 클릭한다. (AWS 검색창에 ‘/EC2’라고 검색해도 됨.)
3. 연이어 나타나는 페이지에서 `인스턴스 시작` 버튼을 클릭한다.
4. `AMI(Amazon Machine Image)`를 선택한다. 보통은 Ubuntu로 선택.
5. `인스턴트 유형`을 선택한다. 프리티어는 `t2.micro` 만 선택 가능.
6. `키 페어` 설정에서 키를 발급받고 잘 저장해둔다. 발급받은 키는 생성할 인스턴스 서버에 관리자 권환으로 접근하기 위해 필요한 키이다.
7. `네트워크 설정` 항목에서 `보안 그룹 생성` 옵션을 클릭한 후, `보안그룹 추가`를 눌러서 TCP 유형에서 내가 추가할 포트들을 추가해준다.
8. `스토리지 설정`에서 생성할 인스턴스 서버의 저장 공간을 세팅한다. 프리티어는 기본으로 Amazon EBS(SSD)로 지정되어 있으며, 최대 30GB까지 설정 가능하다.

# AWS EC2 서버에 접속하기

1. 리눅스에 `awscli(AWS Command Line Interface)`를 설치한다.

```
# apt install awscli
```

위 명령어에서 apt update 관련 에러 뜨면, apt update 그대로 입력해 준 후에 다시 시도한다.

설치가 잘 되었으면, 버전 확인을 통해 확인한다.

```
# aws --version
aws-cli/1.22.34 Python/3.10.6 Linux/5.10.16.3-microsoft-standard-WSL2 botocore/1.23.34
```

1. 이제 shell 스크립트 통신으로 접속을 해보자. 키의 이름이 awspw.pem이고, 퍼블릭 ip 주소가 3.132.13.94 이라면 아래와 같이 명령어를 입력하면 된다. AWS에서 ubuntu 환경을 사용하는 경우, username은 ubuntu가 디폴트 이름이다. 

```
# ssh -i awspw.pem ubuntu@3.132.13.94
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

만약 privite key file 에러가 난다면, 현재 보유 중인 키의 권한을 pulic에서 private로 변경해줘야 한다. 그러기 위해서는 먼저 ubuntu라는 사용자 디렉토리를 만들어줘야 한다.(확실치는 않음)

```
# cd
# mkdir /home/ubuntu
# chmod 400 awspw.pem
```

그리고 권환이 잘 변경되었는지 확인해보자.

```
# ls -l awspw.pem기-- 1 root root 1678 May 29 14:46 awspw.pem
```

사용자만 읽기 권한이 있는 이 상태가 바로 private 상태이다. 이제 다시 접속을 시도해보면 잘 될 것이다.