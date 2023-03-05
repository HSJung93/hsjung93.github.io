---
title: 다른 서버의 방화벽이 열려있는지 확인하는 방법
date: 2023-02-20 10:02:00 +0900
categories: [SlipBox, Programming]
tags: [Linux, Shell]
---

한 서버에서 다른 서버의 방화벽이 열려 있는지 확인하기 위해서는 다른 서버의 IP 주소와 포트 주소 그리고 명령어가 필요하다. 다양한 방법이 있겠지만 주로 telnet, curl 명령어를 통하여 확인한다.

# telnet 명령어
`telnet [ip] [port]`
- 컴퓨터와 컴퓨터 사이를 이어주는 명령어
- 보안 이슈로 인하여 ssh로 대체되는 경우가 많다.
- endpoint health check에 사용된다.

## 성공 응답
```
Trying [ip]...
telnet: connect to address [ip]: Connection refused
```


## 실패 응답
```
Trying [ip]...
Connected to [ip].
Escape character is '^]'.
```


cf. ping 명령어
- L3 명령어
- 패킷을 보내고 대상이 보내는 응답을 분석한다.

# curl 명령어
`curl -v [ip]:[port]`
- 커맨드라인, 스크립트에서 데이터 전송을 위하여 사용되는 라이브러리
- HTTP, FTP 등 다양한 통신 프로토콜을 지원
- telnet이 설치되어 있거나, 보안 이슈일 때 대신 사용한다.

## 성공 응답
```
[root@teraone ~]# curl -v telnet://192.168.56.101:8080
* About to connect() to 192.168.56.101 port 8080 (#0)
*   Trying 192.168.56.101...
* Connected to 192.168.56.101 (192.168.56.101) port 8080 (#0)
```
## 실패 응답
```
[root@teraone ~]# curl -v telnet://192.168.56.101:1122
* About to connect() to 192.168.56.101 port 1122 (#0)
*   Trying 192.168.56.101...
* Connection refused
* Failed connect to 192.168.56.101:1122; Connection refused
* Closing connection 0
curl: (7) Failed connect to 192.168.56.101:1122; Connection refused
```

cf. telnet과 curl은 클라이언트에서 통신 여부를 확인할 때 사용한다.
- 서버단에서는 netstat 명령어를 많이 사용한다.
- `netstat -an | grep [port]` 로 검색
- LISTEN, ESTABLISHED, TIME_WAIT 으로 파악한다. 

# References
- https://uutopia.tistory.com/41
- https://velog.io/@km1031kim/telnet-curl-%EC%B0%A8%EC%9D%B4-%ED%8F%AC%ED%8A%B8%ED%99%95%EC%9D%B8