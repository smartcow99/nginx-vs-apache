# NGINX vs Apache 성능 벤치마크 테스트

NGINX와 Apache 웹 서버 간의 성능을 비교하고, PHP-FPM과 FastCGI 설정을 통해 성능 최적화를 어떻게 수행하는지에 대해 설명합니다.

## 목차
1. [개요](#개요)
2. [테스트 환경 준비](#테스트-환경-준비)
3. [Apache 포트 변경 방법](#apache-포트-변경-방법)
4. [성능 테스트 도구 설치](#성능-테스트-도구-설치)
5. [ApacheBench(ab)를 이용한 테스트](#apachebenchab를-이용한-테스트)
6. [wrk를 이용한 부하 테스트](#wrk를-이용한-부하-테스트)
7. [FastCGI 및 PHP-FPM 설정](#fastcgi-및-php-fpm-설정)
8. [결과 분석](#결과-분석)
9. [결론](#결론)

## 개요
이 프로젝트에서는 NGINX와 Apache 웹 서버의 성능을 부하 테스트 도구를 사용하여 비교합니다. PHP-FPM 및 FastCGI 설정을 활용하여 웹 서버가 어떻게 PHP 요청을 처리하고, 성능을 최적화하는지 설명합니다.

## 테스트 환경 준비
1. **Apache 설치**: Apache와 PHP-FPM 모듈을 설치합니다.
   ```bash
   sudo apt install apache2 libapache2-mod-fcgid
   sudo apt install php-fpm

2. **NGINX 설치**: NGINX와 PHP-FPM 모듈을 설치합니다.
  ```bash
  sudo apt install nginx php-fpm
  ```
## Apache 포트 변경 방법
  Apache가 기본적으로 80번 포트를 사용하기 때문에 NGINX와 충돌하지 않도록 포트를 변경합니다.
1. `/etc/apache2/ports.conf` 파일을 엽니다.
  ```bash
  sudo nano /etc/apache2/ports.conf
  ```
2. `Listen 80`을 `Listen 8080`으로 변경한 후 저장합니다.

3. Apache를 재시작합니다.

  ```bash
  sudo systemctl restart apache2
  ```
## 성능 테스트 도구 설치
성능 테스트를 위해 **ApacheBench(ab)**와 **wrk** 도구를 사용합니다.

### ApacheBench 설치
  ```bash
  sudo apt install apache2-utils
  ```
### wrk 설치
  ```bash
  sudo apt install wrk
  ```
## ApacheBench(ab)를 이용한 테스트
ApacheBench(ab)를 사용하여 Apache와 NGINX의 성능을 테스트합니다.

### Apache에 대한 부하 테스트
  ```bash
  ab -n 10000 -c 100 http://localhost:8080/index.html
  ```
### NGINX에 대한 부하 테스트
```bash
  ab -n 10000 -c 100 http://localhost/index.html
```
- `-n 10000`: 총 10,000개의 요청
- `-c 100`: 동시에 100개의 요청
- `http://localhost/index.html`: 테스트할 URL

## wrk를 이용한 부하 테스트
wrk는 더 강력한 부하 테스트 도구로 대규모 트래픽 시뮬레이션이 가능합니다.

  ```bash
  wrk -t12 -c400 -d30s http://localhost/index.html
  ```
- -t12: 12개의 스레드를 사용합니다.
- -c400: 동시에 400개의 연결을 유지합니다.
- -d30s: 30초 동안 테스트를 실행합니다.

## FastCGI 및 PHP-FPM 설정
PHP 요청을 최적화하기 위해 Apache와 NGINX 모두 PHP-FPM 또는 FastCGI를 사용해야 합니다.

### NGINX FastCGI 설정
NGINX 설정 파일 `/etc/nginx/sites-available/default`에서 PHP 설정을 확인하세요:

  ```nginx
  location ~ \.php$ {
      include snippets/fastcgi-php.conf;
      fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
  }
  ```

### Apache PHP-FPM 설정
Apache에서 PHP-FPM을 사용하려면 다음 명령어로 모듈을 활성화하세요:

  ```bash
  sudo a2enmod proxy_fcgi setenvif
  sudo a2enconf php7.4-fpm
  sudo systemctl restart apache2
  ```

## 결과 분석
부하 테스트가 완료되면 결과를 통해 다음과 같은 항목을 비교할 수 있습니다:

- **Requests per second**: 초당 처리 요청 수
- **Time per request**: 요청당 소요 시간
- **Transfer rate**: 초당 전송된 데이터 양

NGINX와 Apache가 PHP-FPM을 통해 얼마나 성능 차이가 나는지 분석하고, 각각의 서버가 주어진 요청에 어떻게 응답하는지 평가합니다.

## 성능 테스트 결과

이 문서에서는 Apache와 NGINX 웹 서버의 성능을 비교한 결과를 제공합니다. 성능 테스트는 ApacheBench(`ab`)와 `wrk`를 사용하여 수행되었으며, 각 서버에 대해 100개의 동시 요청과 총 10,000개의 요청을 처리하였습니다.

### 테스트 환경

- **Apache 버전**: 2.4.52
- **NGINX 버전**: 1.18.0
- **테스트 URL**: `http://localhost/index.php`
- **동시 요청 수**: 100
- **총 요청 수**: 10,000

### 성능 테스트 결과

![image](https://github.com/user-attachments/assets/d05ee883-d85f-4b17-a54b-c3d60712056e)

#### ApacheBench (`ab`) 결과

| 서버   | 요청 수 초당 (Requests/sec) | 요청당 시간 (Time per Request, ms) | 전송 속도 (Transfer Rate, Kbytes/sec) |
|--------|-----------------------------|-------------------------------------|----------------------------------------|
| Apache | 12,904.92                   | 7.749                               | 5,683.71                               |
| NGINX  | 17,160.16                   | 5.827                               | 5,379.31                               |


#### wrk 결과


| 서버   | 요청 수 초당 (Requests/sec) | 요청당 시간 (Time per Request, ms) | 전송 속도 (Transfer Rate, MB/sec)   |
|--------|-----------------------------|-------------------------------------|--------------------------------------|
| Apache | 21,591.93                   | 263.56                               | 224.99                                |
| NGINX  | 46,290.34                   | 54.27                                | 482.12                                |




#### ApacheBench 결과

- **요청 수 초당**: NGINX가 17,160.16 요청으로 Apache의 12,904.92 요청을 초과하여 더 높은 성능을 보였습니다. 이는 NGINX가 동시 요청을 처리하는 데 더 효율적임을 나타냅니다.
  
- **요청당 시간**: NGINX는 요청당 평균 5.827ms의 응답 시간을 기록하며, Apache의 7.749ms보다 빠른 응답성을 보여주었습니다. 이는 사용자 경험 향상에 기여할 수 있습니다.
  
- **전송 속도**: Apache는 5,683.71 Kbytes/sec로 전송 속도가 다소 높았으나, NGINX는 전송 속도가 5,379.31 Kbytes/sec로 유사한 수준을 유지하고 있습니다. 전송 속도는 서버의 하드웨어 및 네트워크 성능에 따라 달라질 수 있음을 유의해야 합니다.

#### wrk 결과

- **요청 수 초당**: `wrk`로 측정한 결과에서 NGINX는 46,290.34 요청으로 Apache의 21,591.93 요청을 초과하여 더욱 높은 성능을 보였습니다. 이는 NGINX가 고부하 환경에서 더 잘 작동함을 나타냅니다.
  
- **요청당 시간**: NGINX는 요청당 평균 54.27ms의 응답 시간을 기록하며, Apache는 263.56ms로 상대적으로 더 긴 응답 시간을 보였습니다. 이는 NGINX의 성능이 매우 뛰어남을 시사합니다.
  
- **전송 속도**: NGINX는 482.12MB/sec로 Apache의 224.99MB/sec를 크게 초과하며, 데이터 전송 성능에서도 우위를 차지했습니다.




## 결론

이번 성능 테스트 결과, NGINX가 Apache에 비해 요청 수 초당, 요청당 시간 및 전송 속도에서 모두 더 나은 성능을 보여주었습니다. 이는 NGINX가 고성능 웹 서버로서의 역할을 충실히 수행할 수 있음을 나타냅니다. 그러나 전송 속도는 두 서버 간의 차이가 크지 않았습니다. 따라서 웹 서버 선택 시 다양한 요인을 고려해야 할 필요가 있습니다.

