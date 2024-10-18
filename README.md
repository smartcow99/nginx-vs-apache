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

## 결론
PHP-FPM을 사용하면 NGINX와 Apache 모두 성능이 개선됩니다.
기본 설정에서는 NGINX가 동시 요청 처리에서 더 우수한 성능을 보이는 경우가 많습니다.
성능 테스트 결과는 서버 설정에 따라 달라지므로 최적의 환경을 찾는 것이 중요합니다.
