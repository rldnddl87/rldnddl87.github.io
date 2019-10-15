---
title : 도커 명령어 및 간단 사용법 정리
categories : [DevOps]
tags: [Infra, 환경구성, Docker]
date: 2019-10-16T03:00:00Z
comments: true
author_profile: false
---



- 개인적으로 사용해본 도커의 간단한 사용법 및 명령어를 정리해보았습니다.
- 개인적인 용도로 노션에 저장되어 있던 내용을 업로드해서 반말이 섞여 있습니다. ^^;

---

## 도커 배포 VS 기존 배포
- 기존 배포
    - 언어,프레임워크,WAS,서버 등등 다양한 방식 -> 한번에 배포가 어려움(미설치, 충돌 문제)
- 도커 배포
    - 컨테이너를 사용하면 배포방식이 동일해 진다 -> 이미지 다운 -> 컨테이너 실행

## 기본 명령어
<pre>
#docker 설치
curl -fsSL https://get.docker.com/ | sudo sh

#도커는 기본적으로 root 권한으로 실행된다. 사용자 권한주기
#sudo 없이 사용할 수 있도록..
sudo usermod -aG docker $USER #현재 사용중인 사용자에게 권한 주기
sudo usermod -aG docker giung #giung user에게 docer 사용 권한 주기
#로그인 중이라면 다시 로그인해야 적용이 된다.


#docker 버전 보기
docker version

#docker 실행 명령어
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]

#OPTIONS..
-d  #detached mode : 백그라운드 모드로 실행
-p  #호스트와 컨테이너의 포트를 연결(포트포워딩)
-v  #호스트와 컨테이너의 디렉토리를 연결(마운트)
-e  #컨테이너 내에서 사용할 환경변수 설정
-name  #컨테이너 이름 설정
-rm  #프로세스 종료시 컨테이너 자동 제거
-it  #-i와 -t를 동시사용 터미널 입력을 위한 옵션

#check running container
docker ps 
docker ps -a

#stop container
docker stop [OPTIONS] CONTAINER [CONTAINER...]

#remove container
docker rm [OPTIONS] CONTAINER [CONTAINER...]

#중지된 컨테이너 한번에 삭제하기
docker rm -v $(docker ps -a -q -f status=exited)

#docker images
docker images [OPTIONS] [REPOSITORY[:TAG]]

#docker image pull
docker pull [OPTIONS] NAME[:TAG|@DIGEST]

#docker image remove
docker rmi [OPTIONS] IMAGE [IMAGE...]

#read container logs
docker logs [OPTIONS] CONTAINER

#OPTIONS
-f : 실시간 로그 확인
-tail : tail 10 (마지막 10줄만 보기)

#컨테이너의 파일 실행 or 컨테이너에 들어가보기
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
</pre>

## 컨테이너 업데이트
1. 새로운 이미지 Pull
2. 기존 컨테이너 Stop, Remove
3. 새로운 이미지 기반 컨테이너 Run
- 컨테이너 삭제시 주의 사항
    - 컨테이너 상에서 생성된 파일 또한 사라진다. db data, 업로드 데이터 모두~
    - 컨테이너 삭제시 보존해야하는 데이터는 외부에 쌓이도록..
    
- 컨테이너에 데이터 볼륨 연결하기( 호스트 디렉토리 마운트)
<pre>
#run 옵션중 -v 사용한다(마운트)

#befor
docker -run -d -p 3306:3306 \
 -e MYSQL_ALLW_EMPTY_PASSWORD=true \
	--name mysql \
	mysql:5.7

#after
docker run -d -p 3306:3306 \
	-e MYSQL_ALLOW_EMPTY_PASSWORD=true \
	--name mysql \
	-v /my/own/datadir:/var/lib/mysql \ # <- volumn mount
 	mysql:5.7

#호스트의 /my/own/datadir 디렉토리를 컨테이너의 /var/lib/mysql 디렉토리로 마운트
</pre>

## 로그
- 컨테이너 안에서 사용되는 프로그램은 제 각각이다.
- 컨테이너 로그를 살펴보면 관련 프로그램 로그가 나온다.
- 컨테이너는 해당 컨테이너안에서 구동중인 프로그램의 표준 스트림 stdout, stderr을 수집한다.
- 컨테이너 상에 로그가 쌓이게 한다면 컨테이너가 종료되면 사라지니.. 로그를 컨트롤할 서비스를 이용하거나 설정을 통해 컨트롤 해주어야 한다.

## 도커 이미지 만들기
- DocerFile이라는 이미지 빌드용 dsl파일을 사용한다.( Domain Specific Language)
    - 예) app을 서버에 배포한다면 절차는 다음과 같다  
        1. 서버 설치(linux os)
        2. app 환경 설치(jdk, Ruby..)
        3. 소스 복사
        4. 패키지 설치( Gem..)
        5. 서버 실행(Sinatra for Ruby)
        <pre> 
        #Ruby를 ubuntu에 배포한다고 가정하고..
        #스크립트로 나타내면..
        
        # 1. ubuntu 설치(패키지 업데이트)
        apt-get update
        
        # 2.ruby 설치
        apt-get install ruby
        gem install bundler
        
        # 3. 소스 복사
        mkdir -p /usr/src/app
        scp Gemfile app.rb root@ubuntu:/usr/src/app # From host
        
        # 4. Gem 패키지 설치
        bundle install
        
        # 5. Sinatra 서버 실행
        bundle exec ruby app.rb
        
        #Ubuntu Container를 실행하고 위 스크립트를 실행하면 웹서버가 실행된다.
        
        #이를 Docker file로 만들면 된다.
        ########################################
        #1. ubuntu 설치(패키지 업데이트 + 만든사람 표시)
        FROM       ubuntu:16.04
        MAINTAINER giung.song@mement.com
        RUN        apt-get -y update
        
        #2. ruby 설치
        RUN apt-get -y install ruby
        RUN gem install bundler
        
        #3. 소스 복사
        COPY . /usr/src/app 
        
        #4. Gem 패키지 설치 (실행 디렉토리 설정)
        WORKDIR /usr/src/app
        RUN     bundle install
        
        #5. Sinatra 서버 실행(Listen 포트 정의)
        EXPOSE 4567
        CMD bundle exec ruby app.rb -o 0.0.0
        ##########################################
        #Docer file에서는 키보드입력이 안되니 -y를 추가 했다.
        
        
        #빌드 파일이 생성되었으니 이미지를 만들어 보자
        ##########################################
        #Docker build
        
        docker build [OPTIONS] PATH | URL | -
        #생성할 이미지 이름을 지정하기 위한 -t(--tag)옵션을 알면 빌드가 가능하다.
        
        #위에서 생성한 docker file의 위치로 이동하여..
        docker built -t app . 
        # app이라는 이름의 image를 Dockerfile 의 설정을 이용하여 생성하였다.!
        
        
        
        
        ###########################요약##################################
        FROM        ubuntu:16.04
        MAINTAINER  giung.song@mement.net
        RUN         apt-get -y update
        
        RUN  apt-get -y install ruby
        RUN  gem install bundler
        
        COPY . /usr/src/app
        
        WORKDIR /usr/src/app
        RUN     bundle install
        
        EXPOST 4567
        CMD    bundle exec ruby app.rb -o 0.0.0.0
        #################################################################
        #########################최적화###################################
        FROM ruby:2.3
        MAINTAINER subicura@subicura.com
        COPY Gemfile* /usr/src/app/
        WORKDIR /usr/src/app
        RUN bundle install --no-rdoc --no-ri
        COPY . /usr/src/app
        EXPOSE 4567
        CMD bundle exec ruby app.rb -o 0.0.0.0
        #################################################################
        </pre> 
    - 도커 파일 기본 명령어
        <pre>
        ###Dockerfile의 기본 명령어###
        FROM <image>:<tag>
        # FROM ubuntu:16.04
        # 베이스 이미지를 지정한다.<필수>
        # tag는 latest보다는 구체적인 버전을 지정하는것이 좋으며 생성되어진 base는
        # https://hub.docker.com/search/?q=&type=image 확인가능하다.
        
        MAINTAINER <name>
        MAINTAINER giung.song@mement.net
        # Dockerfile을 관리하는 사람의 이름 또는 이메일 정보를 적는다 빌드에 영향 X
        
        COPY <src>... <dest>
        COPY . /usr/src/app
        # 파일, 폴더를 이미지로 복사한다. 일반적으로 소스 복사에 사용. 
        # target(dest) 디렉토리가 없다면 자동으로 생성한다.
        
        ADD <src>... <dest>
        ADD . /usr/src/app
        # COPY 명령어와 유사하나 추가기능 존재
        # src에 파일 대신 url을 입력할 수 있고 src에 압축 파일을 입력하는 경우 자동으로 압축해제한 뒤 복사된다.
        
        RUN <command>
        RUN ["executable", "param1", "param2"]
        RUN bundle install
        # 가장 많이 사용하는 구문입니다. 명령어를 그대로 실행합니다.
        # 내부적으로 /bin/sh -c 뒤에 명령어를 실행하는 방식
        
        CMD ["executable", "param1", "param2"]
        CMD command param1 param2
        CMD bundle exec ruby app.rb
        # 도커 컨테이너가 실행되었을 경우 실행되는 명령어를 정의한다. 빌드할 때는 실행되지 않으며 여러개의 CMD가 존재할 경우 마지막 CMD만 실행
        # 컨테이너 실행 후 여러 프로그램을 실행하고 싶을경우 run.sh같은 스크립트를 작성하여 데몬으로 실행한다.
        
        WORKDIR /path/to/workdir
        # RUN, CMD, ADD, COPY등이 이루어질 기본 디렉토리를 설정한다.
        # 각 명령어의 현재 디렉토리는 한줄 마다 초기화 되기 때문에... RUN cd /path 를 하더라도 다음 명령어에선 다시 위치가 초기화된다.
        # 같은 디렉토리에서 작업하기 위해 WORKDIR를 사용한다.
        
        EXPOSE <port> [<port>...]
        EXPOSE 4567
        # 도커 컨테이너가 실행되었을 때 요청을 기다리고 있는(Listen) 포트를 지정. 여러 포트를 지정할 수 있다.
        
        VOLUME ["/data"]
        #컨테이너 외부에 파일 시스템을 마운트 할 때 사용합니다. 반드시 지정하지 않아도 마운트 할 수 있지만, 기본적으로 지정하는것이 좋다. 컨테이너 상으로 함녀 컨테이너 종료되면 다 사라지니까~
        
        ENV <key> <value>
        ENV <key>=<value> ...
        ENV DB_URL mysql
        # 컨테이너의 환경변수를 지정. 컨테이너 실행시 -e 옵션으로 환경변수를 따로 지정할 경우 기존 값을 오버라이딩 한다.
        </pre>
    - 빌드 과정
        <pre>
        ###Dockerfile의 기본 명령어###
        FROM <image>:<tag>
        # FROM ubuntu:16.04
        # 베이스 이미지를 지정한다.<필수>
        # tag는 latest보다는 구체적인 버전을 지정하는것이 좋으며 생성되어진 base는
        # https://hub.docker.com/search/?q=&type=image 확인가능하다.
        
        MAINTAINER <name>
        MAINTAINER giung.song@mement.net
        # Dockerfile을 관리하는 사람의 이름 또는 이메일 정보를 적는다 빌드에 영향 X
        
        COPY <src>... <dest>
        COPY . /usr/src/app
        # 파일, 폴더를 이미지로 복사한다. 일반적으로 소스 복사에 사용. 
        # target(dest) 디렉토리가 없다면 자동으로 생성한다.
        
        ADD <src>... <dest>
        ADD . /usr/src/app
        # COPY 명령어와 유사하나 추가기능 존재
        # src에 파일 대신 url을 입력할 수 있고 src에 압축 파일을 입력하는 경우 자동으로 압축해제한 뒤 복사된다.
        
        RUN <command>
        RUN ["executable", "param1", "param2"]
        RUN bundle install
        # 가장 많이 사용하는 구문입니다. 명령어를 그대로 실행합니다.
        # 내부적으로 /bin/sh -c 뒤에 명령어를 실행하는 방식
        
        CMD ["executable", "param1", "param2"]
        CMD command param1 param2
        CMD bundle exec ruby app.rb
        # 도커 컨테이너가 실행되었을 경우 실행되는 명령어를 정의한다. 빌드할 때는 실행되지 않으며 여러개의 CMD가 존재할 경우 마지막 CMD만 실행
        # 컨테이너 실행 후 여러 프로그램을 실행하고 싶을경우 run.sh같은 스크립트를 작성하여 데몬으로 실행한다.
        
        WORKDIR /path/to/workdir
        # RUN, CMD, ADD, COPY등이 이루어질 기본 디렉토리를 설정한다.
        # 각 명령어의 현재 디렉토리는 한줄 마다 초기화 되기 때문에... RUN cd /path 를 하더라도 다음 명령어에선 다시 위치가 초기화된다.
        # 같은 디렉토리에서 작업하기 위해 WORKDIR를 사용한다.
        
        EXPOSE <port> [<port>...]
        EXPOSE 4567
        # 도커 컨테이너가 실행되었을 때 요청을 기다리고 있는(Listen) 포트를 지정. 여러 포트를 지정할 수 있다.
        
        VOLUME ["/data"]
        #컨테이너 외부에 파일 시스템을 마운트 할 때 사용합니다. 반드시 지정하지 않아도 마운트 할 수 있지만, 기본적으로 지정하는것이 좋다. 컨테이너 상으로 함녀 컨테이너 종료되면 다 사라지니까~
        
        ENV <key> <value>
        ENV <key>=<value> ...
        ENV DB_URL mysql
        # 컨테이너의 환경변수를 지정. 컨테이너 실행시 -e 옵션으로 환경변수를 따로 지정할 경우 기존 값을 오버라이딩 한다.
        </pre>
        
        1. 임시 컨테이너 생성
        2. 명령어 수행
        3. 이미지로 저장
        4. 임시 컨테이너 삭제
        5. 새로만든 이미지 기반의 임시 컨테이너 생성
        6. 명령어 수행
        7. 이미지로 저장
        8. 임시 컨테이너 삭제 ..
        - 명령어를 실행할 때마다 이미지 레이어를 새롭게 저장하고 다시 빌드한다. -> 레이어 개념!
        
    - 도커의 이미지 저장소  
    ![](/assets/images/posts/2019-10-16/docker_image_repo.png){: width="50%"}
        - 빌드한 이미지를 서버에 배포시 직접 이미지를 복사하는 것이 아닌 도커 레지스트리/도커 허브라는 이미지 저장소를 사용한다.
        - 명령어를 이용하여 저장소에 push하고 필요시 pull 받아 사용하는 구조
        
## Docker Compose
- 다수의 컨테이너를 관리하는 툴
- 기존의 CLI에서 작성하던 명령어들을 한대모아 YAML 방식으로 작성한뒤 명령어를 통해 실행하면 해상 설정에 맞게 도커가 실행된다.
<pre>
#설치하기
curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
#https://docs.docker.com/compose/install/

chmod +x /usr/local/bin/docker-compose
# test
docker-compose version

#설정파일 만든후
docker-compose up
</pre>

## Docker Hub
<pre>
# 로그인
docker login
# 로그인 되고 ~/.docker/config.json에 인증정보가 저장되어 로그아웃 되기 전 까지 유지된다.

# Docker이미지 이름은 다음과 같은 형태로 구성된다.
[Registry URL]/[사용자 ID]/[이미지명]:[tag]
# Registry URL은 기본적으로 도커 허브를 바라본다.
# 사용자 ID를 지정하지 않으면 기본값(library)을 사용한다.
# 따라서 ubuntu = library/ubuntu = docker.io/library/ubuntu는 모두 동일한 표현

# tag명령어로 기존 image에 추가로 이름을 지어 줄 수 있다.
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
# ex app이라는 이름의 이미지에 tag를 이용하여 추가 정보를 달아보자
docker tag app giung.song/giung-app:1 
# giung.song이라는 ID를 사용하고 이미지 이름을 giung-app으로 변경하였다. 태그는 첫번째 버전으로 1을 추가

# 도커 허브에 이미지를 전송해보자
docker push dockerid/repositoryname:tag
# 도커 아이디/저장소이름:tag 로 업로드된다.
</pre>

## 테스트로 프로젝트 도커로 배포해보기
<pre>
Goal : 현재 프로젝트를 hiver-v에 Docker 컨테이너로 배포하기

프로젝트 구성 요소 : java , mysql, redis

Dockerfile 설정 후 이미지 생성한뒤 docker hub에 업로드 pull하여 실행까지해보기



1)실제 스크립트 작성 -> 2)dockerfile로 변경 -> 3)이미지 생성
</pre>

<pre>
# ubuntu update(설치)
apt-get update

# openjdk setup
apt-get install openjdk-8-jdk

# git setup
apt-get install git

# mysql setup
apt install mysql-server

# redis setup
apt install redis-server

# gradle setup
apt-get install gradle

#소스 복사


#db계정 설정
# 1)db생성
# 2)유저생성 및 권한 부여

#project run


</pre>
       
<pre>

FROM 		ubuntu:19.10
MAINTAINER  giung.song@test.net
RUN 		apt-get update

RUN 		apt-get install openjdk-8-jdk
RUN 		apt-get install git
RUN 		apt-get install mysql-server
RUN 		apt-get install redis-server
RUN         apt-get install gradle

RUN         CREATE DATABASE test CHARACTER SET utf8;
RUN			grant all privileges on test.* to giung@'%' identified by '1234';
RUN			flush privileges;

COPY 		. /usr/app/testproject

WORKDIR     /usr/app/testproject
EXPOSE 		8080
CMD 		nohup java -jar webservice-0.0.1-SNAPSHOT.jar &
</pre> 
## 마치며  
 - 며칠동안 도커를 사용해았다. 서버환경에 상관 없이 빠르게 인프라 환경을 구성하여 서비스를 구동할 수 있다는 점이 놀라웠고 실제 업무에서 활용할 기회가 와서 조금 더 깊이 있게 알아볼 수 있으면 좋겠다.
 
---
참고  
[https://subicura.com/2016/06/07/zero-downtime-docker-deployment.html](https://subicura.com/2016/06/07/zero-downtime-docker-deployment.html)  
[https://tacademy.skplanet.com/live/player/onlineLectureDetail.action?seq=125](https://tacademy.skplanet.com/live/player/onlineLectureDetail.action?seq=125)