# Caddy 사용메뉴얼

AWS의 EC2 인스턴스를 사용하고 있다는 것을 전제로 진행한다.

# 1. 방화벽 설정

먼저 인바운드 규칙에서 포트 80(http)와 포트 443(https)를 모두 허용한다.

# 2. 도메인 발급 받기

let’s encrypt 서비스를 이용하려면 먼저 도메인을 발급받아야 한다. 이 서비스는 도메인 존재 여부로 신뢰성을 확인한 후 SSL 인증서를 발급하기 때문이다.

[무료 도메인 발급 사이트](https://xn--220b31d95hq8o.xn--3e0b707e/)

위 사이트는 `[내도메인.한국](http://내도메인.한국)` 이라는 사이트로, 무료로 도메인 발급을 해준다. 여기서 로그인 후에 원하는 도메인을 발급받으면 된다. (망할 놈의 freenom 사이트를 이용할 필요없음.)

# 3. caddy 설치

stable realease 버전 설치하기 (리눅스 기준)

참고: ‘붙여넣기’는 ‘shift + insert’

```
$ sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
$ curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
$ curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
$ sudo apt update
$ sudo apt install caddy
```

중간 중간에 재시작하라고 뜰텐데, 무시하고 계속 OK 눌러주면 된다.

# 4. caddy 설정 파일 구성

[우분투에 Caddy 설치하기](https://wany.io/b/install-caddy-on-ubuntu)

caddy 설정 파일의 위치는 `/etc/caddy/Caddyfile` 이다. vi 에디터를 사용하여 caddyfile을 수정해보자. 

참고: vi 에디터 사용법 (반드시 관리자 모드로 접근할 것.)

- 파일 수정 - insert
- 수정 모드 해제 - esc
- 저장 - :w
- 종료 - :q
- 저장 및 종료 - :wq

```
$ sudo vi /etc/caddy/Caddyfile
```

일반적으로 caddy 서버를 활용하는 방법은 크게 두 가지이다.

1. caddy 서버 자체를 메인으로 사용하기
2. caddy 서버를 `리버스 프록시` 서버로 활용하기

여기서 우리는 메인 서버가 따로 있기 때문에, `리버스 프록시` 목적으로 서버를 세팅해줘야 한다. 

## (포워드) 프록시 서버

일반적으로 우리가 말하는 `프록시 서버` 는 `포워드 프록시 서버` 를 의미한다. 클라이언트가 서버로 요청을 보내면, 프록시 서버가 해당 요청을 가로챈 뒤에 그 요청을 대신 서버로 보낸다. 그리고 서버의 요청을 다시 프록시 서버가 받게 되고, 이 요청을 클라이언트에게 보냄으로써 아무런 문제 없이 클라이언트 - 서버 간 통신을 유지시킬 수 있게 된다. 

프록시 서버를 사용하는 이유는 아래와 같다.

- 특정 클라이언트 네트워크 그룹의 제한된 인터넷 사용을 위해 사용한다. 프록시 서버에 네트워크 규칙을 설정하여 특정 사이트에 접속하는 것을 막을 수 있다.
- 유저의 정체를 숨긴 채로 인터넷 활동이 가능해진다. IP 주소를 역추적해봤자 프록시 서버만 보이기 때문이다.

## 리버스 프록시 서버

`리버스 프록시 서버` 도 일반 프록시 서버와 동일하게 클라이언트의 요청을 받아서 해당 요청을 백엔드 서버로 전달하고, 백엔드 서버의 응답을 클라이언트에게 전달하는 기능을 한다. 하지만 가장 큰 차이점은 클라이언트 네트워크 그룹을 관리하는 프록시 서버와 달리, **리버스 프록시 서버는 웹 서버 네트워크 그룹을 담당하는 프록시 서버 역할을 한다는 것이다.** 

리버스 프록시 서버를 사용하는 이유는 아래와 같다.

- 로드 벨런싱을 위해 사용한다. 특정 서버가 과부하되지 않도록, 리버스 프록시 서버가 여러 대의 서버에 융통성 있게 트래픽을 관리한다.
- 본래 서버의 ip 주소를 노출시키지 않으므로 보안이 좋아진다. 메인 서버를 향한 DDos 공격은 다 막을 수 있다.
- 리버스 프록시 서버에 캐시 데이터를 저장하여 성능을 향상시킬 수 있다.
- SSL 암호화를 쉽게 만든다. 메인 서버와 클라이언트가 직접적으로 SSL 암호화 및 복호화 통신을 진행하면 통신 비용이 많이 들게 된다. 하지만 리버스 프록시 서버를 사용하면 메인 서버 대신 들어오는 요청을 복호화하고 나가는 요청을 암호화해주기 때문에, 비용 효율적인 안전한 통신을 보장한다.

`리버스 프록시` 형태로 Caddy 서버의 HTTPS 설정을 구성하면, Caddy 서버에 들어오는 요청을 백엔드 서버로 전달하면서 암호화된 HTTPS 연결을 제공할 수 있다. 기본적으로는 백엔드 서버과 리버스 프록시 서버를 연결하는 방법이 있으나, 우리는 프론트 서버가 따로 존재하기 때문에 프론트 서버에 리버스 프록시 서버를 걸어주면 된다. 결론적으로는 아래와 같이 caddyfile을 수정해주면 된다. (기존 내용은 전부 삭제해도 된다고 한다. 에러 잡으려면 깨끗히 지우고 하는게 좋을지도.)

```
발급받은_도메인_이름 {
    reverse_proxy ip주소:포트번호  # 요청을 전달할 주소

    tls 내 _이메일_주소
}
```

```
example.com {
    reverse_proxy 127.0.0.1:8000

    tls email@example.com
}
```

- reverse_proxy: 리버스 프록시 서버가 클라이언트 요청을 전달할 실제 어플리케이션 서버 주소를 입력한다. 퍼블릭 IP 주소를 넣어도 기능 상 문제는 없으나, 프라이빗 IP를 넣는 것이 더 보안에 좋다. (참고로 도커를 사용 중이라면 각종 어플리케이션 서버들의 프라이빗 IP 주소는 ‘docker inspect 컨테이너_ID’로 확인 가능하다.)
- tls: 서버에서 발생하는 각종 이벤트들(SSL 인증서 기간 연장 등)을 회신받고 싶으면 아래에 메일을 추가한다.

vi 파일 편집이 완료되었으면, 아래 명령어를 통해 caddy를 실행하여 변경사항을 적용한다. caddy를 백그라운드로 돌리기 위해서는 맨 마지막애 ‘&’를 추가한다.

```
$ sudo caddy run --config /etc/caddy/Caddyfile &
```

## 발생 가능한 에러들

- 이미 caddy 서버가 실행 중

```
Error: loading initial config: loading new config: starting caddy administration endpoint: listen tcp 127.0.0.1:2019: bind: address already in use
```

이 에러는 caddy 서버가 실행된 상태에서 다시 caddy run을 입력할 때 뜨는 오류이다. 먼저 caddy stop을 입력한 뒤에 다시 실행해보자.

- 관리자 권한이 필요해

```
Error: loading initial config: loading new config: http app module: start: listening on :80: listen tcp :80: bind: permission denied
```

이 에러는 http 포트를 사용할 권한이 없어서 caddy를 쓰지 못하는 상태이다. 관리자 권한으로 caddy를 실행시켜야 한다.

- SSL 인증서 발급 실패

```
ERROR   tls.obtain      will retry      {"error": "[www.qore.com] Obtain: [www.qore.com] solving challenge: www.qore.com: [www.qore.com] authorization failed: HTTP 0  -  (ca=https://acme.zerossl.com/v2/DV90)", "attempt": 4, "retrying_in": 300, "elapsed": 338.593628116, "max_duration": 2592000}
```

어떠한 이유로 ‘Let’s Encript’에서 SSL 인증서를 발급받지 못하면 이 에러가 발생한다. 혹시나 도메인이 없는 상태에서 진행하려고 하면 안된다. 먼저 개인 도메인을 발급받은 후에 진행하도록 하자.

- 너무 많은 잘못된 인증서 발급 요청

```
Error creating new order :: too many failed authorizations recently:
```

1시간 기다렸다가 다시 요청해야 함.

- 도메인 에러

```
"DNS problem: NXDOMAIN looking up A for myqore.org - check that a DNS record exists for this domain;
```

현재 내가 쓰려고 하는 실제 도메인이 존재하지 않는 경우. 혹시나 도메인이 없는 상태에서 진행하려고 하면 안된다. 먼저 개인 도메인을 발급받은 후에 진행하도록 하자.

# 4. 테스트

이제 발급받은 도메인에 접속해보자. https로 접속이 되면 성공이다.

# 5. 문제점

하지만 이런 방식으로 리버스 프록시 서버를 설정하면 기존 http 주소로 접속을 해도 접속이 된다. 이는 보안에 치명적인 구멍이다. 이를 보완하기 위한 해결책을 마련해야 한다.