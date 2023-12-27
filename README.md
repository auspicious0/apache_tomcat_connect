# 아파치 톰캣 연동

## 왜 연동하는가?

1. Apache는 웹 서버(WS)를 다루며 정적 데이터를 처리합니다.
2. Tomcat은 웹 어플리케이션 서버(WAS)를 다루며 동적 데이터를 처리합니다.
3. 웹 페이지는 정적 데이터, 동적 데이터 모두를 사용합니다.
4. Tomcat은 WS와 WAS를 모두 가지고 있기 때문에 정적, 동적 데이터를 다룰 수 있지만 Tomcat은 Apache보다 느립니다.


따라서 모든 데이터를 Tomcat이 처리한다면 처리 효율이 저하되기 때문에 부하를 분산시켜 속도를 향상시키기 위해 Tomcat과 Apache를 연동합니다.

## Apache Tomcat 연동 단계

### 1. WAS(Tomcat)와 Web(Apache)을 설치합니다.
![image](https://github.com/auspicious0/apache_tomcat_connect/assets/108572025/1c69b9bb-f8a1-46fa-90f0-6f1cee0e0e2f)  ![image](https://github.com/auspicious0/apache_tomcat_connect/assets/108572025/56674356-2ed5-49ed-954c-f1ea3e39cb87)

### 2. Apache 설정

#### 2-1. 서버와 어플 사이의 통신(workers.properties)

```properties
vi /etc/httpd/conf/workers.properties

worker.list=tomcat
worker.tomcat.port=8009 # 일반적 AJP 포트
worker.tomcat.host=... # 톰켓 아이피
worker.tomcat.type=ajp13 # 통신 사이 규약 버전
worker.tomcat.lbfactor=1 # Load Balancing을 위한 가중치
```

#### 2-2. 모듈 로드, 가상 호스트 설정
```httpd.conf
vi /etc/httpd/conf/httpd.conf
Find the Load Module - find the insert position


LoadModule jk_module modules/mod_jk.so # mod_jk 로드
<VirtualHost *:80> # 가상호스트 설정: 하나의 웹서버에서 독립적인 웹사이트를 호스팅
	ServerName ... # apache
	ProxyPass / http:// ...:8080/ # tomcat
	ProxyPassReverse / http://...:8080/ # tomcat
</VirtualHost>
Include conf.modules.d/*.conf – # 설정 파일 포함
```

### 3. Tomcat 설정

```server.xml
Vi $TOMCAT_HOME/server.xml

<Connector protocol=”AJP/1.3”
  address=”0.0.0.0” #모든 네트워크 바인딩
  port=”8009
  redirectPort=”8443” secretRequired=”false”/> # 비밀번호 보안 비활성화
```

### 4. 방화벽 해제 web, was 둘 다

```firewall-cmd
firewall-cmd –permanent –zone=public –-add-port=8009/tcp
firewall-cmd –reload
setenforce 0

```
### 5. 결과
![image](https://github.com/auspicious0/apache_tomcat_connect/assets/108572025/f5a8eac3-e26c-4009-ba00-0e06d34d2136)


