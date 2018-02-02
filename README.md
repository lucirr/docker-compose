# docker-compose sample
* nginx(ssl) + spring-boot + spring-boot.
* ubuntu 16.04 LTS 기준.
* amazon 에서 테스트.

## install
```
$ sudo apt-get update
$ sudo apt-get install docker.io
$ sudo usermod -aG docker $USER
$ docker --version
logout, login
$ sudo curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
$
$ sudo apt-get purge openjdk*
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
$ java -version
```

## sample source
```
$ git clone https://github.com/lucirr/docker-compose.git
$ cd docker-compose
$ cd spring
$ ./mvnw package
```

## ssl setting
* /etc/hosts
```
127.0.0.1    ec2-54-165-15-2.compute-1.amazonaws.com
```
* private key 생성
```
$ cd ~/docker-compose/nginx
$ openssl genrsa -out server.key 2048
```
* csr 생성 (인증서 서명 요청), 실제 공인인증서 발급받을 경우 아래 CN 값을 실제값으로 수정.
```
$ cd ~/docker-compose/nginx
$ openssl req -new -key server.key -out server.csr -subj "/C=KR/ST=Seoul/L=Gang-nam/O=SecureSign Inc/OU=Dev Team/CN=ec2-54-165-15-2.compute-1.amazonaws.com"
```
* crt 생성 (서버 인증서) -> 공인인증서를 사용할 경우는 제외
```
$ cd ~/docker-compose/nginx
$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

## docker run
```
$ cd ~/docker-compose
$ docker-compose up --build -d
$ docker-compose ps
$
$ docker-compose logs -f
$
$ docker-compose stop
```
* scale 적용
```
$ docker-compose up --build -d --scale spring1=3 --scale spring2=3
```

## docker container 접속
```
$ docker ps
$ docker exec -it [CONTAINER ID] /bin/sh
```

## logging 관련
* https://docs.docker.com/compose/compose-file/#logging
* location search : docker inspect --format='{{.LogPath}}' CONTAINER_ID
* location : /var/lib/docker/containers

## config
* docker-compose.yml
```
version: '2'
services:
  web:
    build:
      context: ./nginx
    ports: 
     - "8081:80"
     - "443:443"    
  spring1:
    build:
      context: ./spring
      args:
        JAR_FILE: target/gs-spring-boot-docker-0.1.0.jar  # 실제 파일로 수정
    volumes:
    - ./file:/home/file
    logging:
      driver: "json-file"
      options:
        max-size: "100k"
        max-file: "3"
  spring2:
    build:
      context: ./spring
      args:
        JAR_FILE: target/gs-spring-boot-docker-0.1.0.jar  # 실제 파일로 수정      
    volumes:
    - ./file:/home/file
    logging:
      driver: "json-file"
      options:
        max-size: "100k"
        max-file: "3"
         
```


