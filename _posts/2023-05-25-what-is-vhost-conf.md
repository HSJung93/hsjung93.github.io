---
title: vhost conf 파일이란 무엇인가?
date: 2023-05-25 22:00:00 +0900
categories: [메모상자, 네트워크]
tags: [아파치, 서버]
---

웹 서버가 가상 호스트들을 관리하고 정의하기 위한 파일이다. 한 가상 호스트로 여러 웹사이트들과 어플리케이션들이 한 물리 서버에 호스트되도록 할 수 있다. 각 웹사이트 혹은 어플리케이션은 도메인 이름과 설정을 가지고 있다.

예를 들어 Apache HTTP Server를 사용하다 보면 관습적으로 `httpd-vhosts.conf` 라는 이름의 vhost conf 파일을 사용하고 Apache 설정 파일(`httpd.conf`)에 포함되어 있다. 이 파일에는 어떻게 가상 호스트들을 정의하고 세팅하는지, 혹은 들어온 요청을 각 호스트에 맞게 어떻게 다루어야 하는지에 대한 지시 사항이 적혀 있다.

> 가상호스트란? 하나의 물리 서버에서 많은 웹사이트와 어플리케이션들을 호스팅하는 방법이다. 각각의 웹사이트나 어플리케이션이 스스로의 도메인 이름, 설정, 독립된 환경을 가지면서도 서버의 리소스는 공유하도록 할 수 있다. Apache HTTP Server나 Nginx 에서는 가상 호스트를 지원하고 있다. 
{: .prompt-tip }

대표적인 vhost conf 파일의 형식은 다음과 같다.

```
<VirtualHost *:80>
    ServerName www.example.com
    DocumentRoot /var/www/example

    <Directory /var/www/example>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

위 예시에서는 `www.example.com` 이라는 도메인에 대하여 가상 호스트가 정의되어 있다. `ServerName` directive 에서는 가상 호스트와 관련된 도메인을 적는다. `DocumentRoot` directive 에서는 웹사이트 파일이 위차한 디렉토리를 적어준다. 

`<Directory>` 에서는 정의한 디렉토리(`/var/www/example`)와 관련된 옵션과 권한을 적어주고 있다. 디렉토리 리스팅을 허용하고 심볼릭 링크를 디렉토리 안에서 사용할 수 있도록 허용하고 있다. 또한 `.htaccess` 파일 사용을 허가하여 이 디렉토리의 설정을 덮어쓸 수 있도록 하고 있다. 마지막으로 모든 요청에 대한 접근을 허용하고 있다.

> Indexes란? 기본 파일이 표시한 디렉토리에 없으면 웹서버가 자동적으로 디렉토리의 컨텐츠를 만들어서 웹 브라우저에 표시되도록 하는 옵션 
{: .prompt-tip }