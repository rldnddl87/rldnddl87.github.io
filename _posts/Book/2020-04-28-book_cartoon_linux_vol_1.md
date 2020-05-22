---
title : 만화로 배우는 리눅스 시스템 관리 vol1 - 명령어 & 셸 스크립트 입문
categories : [Book]
tags: [Infra, 환경구성]
date: 2020-05-8T14:00:00Z
comments: true
author_profile: false
---

- 이전 회사에서 CentOS를 사용하여 배포하거나 WSL에 설치된 Ubuntu 사용시 명령어가 익숙지 않았다.  
그때마다 필요한 내용들을 검색해서 사용하곤 했었는데 가볍게 공부하며 정리하기위해 책을 구입했다.

---
1. __외부 PC 조정하기 - ssh__
  - SSH(Secure Shell) 사용
  - 명령어 : <code> ssh 사용자명@네트워크주소</code>  
    
2. __임시로 관리자 권한 얻기 - sudo__
  - root는 모든 권한을 가지기 때문에 로그인을 막음
  - 패스워드를 알 경우 임시로 root 권한을 부여한다.
  - <code>sudo 사용할명령어</code>  
     
3. __다양한 문자열을 한번에 검색하자 - grep__
  - grep(global regular expression print)
  - 파일 내용 검색
  - 정규표현식과 함께 사용가능
  - -r(하위 경로까지 전부 탐색)
  - <code>grep -r "key" /app/api/</code>  
    
4. __터미널에서도 대화형 파일을 편집하자 - vim__
  - i(입력) / esc(입력종료)
  - /(검색) -> esc -> n or shift + n으로 검색내용 순차적으로 접근 가능
  - 모드 개념 : 최초 vim 실행시 노멀모드가 실행 i,v 등을 통해 편집 모드로 전환, esc 입력시 노멀모드로 복귀
  - :wq로 편집 종료  

5. __vim에서도 복사 & 붙이기 되돌리기를 하고 싶어 - yank__
  - vim 노멀모드에서 v키와 방향키로 범위 지정한 뒤 y입력 - 복사완료!
  - 붙여넣고 싶은 위치에서 <code>shift + p</code>로 붙여넣기 / 숫자 + shift + p로 붙여넣을 횟수 지정 가능
  - <code>ctrl + z</code> : 일시 정지되며 콘솔 입력창으로 전환됨(편집중이던 vim은 백그라운드로 이동)  
    fg입력시 편집중이던 vim으로 이동(background to forceground)
  - 노멀모드 + u : 되돌리기 // 노멀모드 + ctrl + r : 반대로 되돌리기  
    
6. __갑작스러운 네트워크 끊김에서 복귀하고 싶어(가상 터미널) - tmux__
  - 사용자 - 가상 터미널 - 서버 (연결은 ssh)
  - 일반적인 ssh로 서버에 접속할 경우 접속이 끊어지게 되면(네트워크 장애 등) 서버측에서는 ssh연결이 종료 되었기 때문에 실행중이던 프로세스를 종료한다.(작업하던것들 bye..)
  - 예를들어 ssh로 서버에 원격 접속하여 vim 작업 도중 네트워크가 끊어지면 작업 중이던 vim도 종료가 되지만 가상 터미널 이용시 이를 방지 할 수 있음.
  - 가상 터미널의 경우 중간 다리 역할을 함으로써 접속이 끊어지게 되더라도 실행 중이던 프로세스가 강제 종료되지 않는다.
  - screen, tmux, byobu등이 있음
  - tmux에 대해 알아보자
      - ssh로 서버 접속 후 <code>tmux</code> 명령어 실행
      - 접속이 끊어진 뒤에도 다시 ssh 연결한 뒤 <code>tmux attach</code>입력하면 작업 중이던 가상 터미널로 복귀된다.
      - 작업 중이던 tmux에서 강제로 빠져나오기 : ctrl + b and d(detach) //오래 걸래는 작업도중 ssh를 종료할 수 있다.
      - <code>ctrl + b s</code>  : 실행된 모든 tmux 세션 리스트 보기
      - <code>tmux a -t 2</code> : 실행중인 tmux 세션 중 특정한 세션에 접속하기
      - <code>Ctrl + B</code>이후에 입력키 들로 관리한다.(기존의 다른 명령어와 겹치지 않도록)
          - D(Detach) : 실행 중이던 tmux 가상 터미널에서 빠져나오기
          - C(Create) : 또 다른 가상 터미널 생성하기
          - P(Previous) / N(Next) : 이전/다음 가상 터미널로 스위칭 하기
              
7. __다른 화면도 보면서 작업하고 싶어(화면 분할)__
  - tmux 환경
  - Ctrl + B then "(가로분할) / %(세로분할) : 화면을 2개로 나눈다.(가로 분할뒤 세로로 또다시 분할이 가능하다 0_0;;)
  - Ctrl + B then 방향키 : 화면 커서 이동
  - tmux로 화면을 분할한 뒤 스크롤 사용하기 : Ctrl + B then [ 입력후 마우스 스크롤 또는 방향키
  - 분할선 조정하기 : Ctrl + B 입력 뒤에 Ctrl누른채로 방향키로 분할 크기 조정
  - exit : 분할 종료

8. __최근 실행한 명령어를 호출하고 싶어(명령어 이력)__
  - <code>~/.bash_history</code> 최근 입력한 모든 명령어가 순차적으로 저장되어 있다.
  - Ctrl + R : 최근 명령어 내역에서 검색
  - Ctrl + R 다음 검색어를 입력한 뒤 Ctrl + R을 계속 입력할 경우 해당 키워드를 순차적으로 역순으로 찾게 되는데 이때 지나가버린 내용들은 다시 검색을 할 수 없다.
  - 지나가버린 내용들을 다시 순차적으로 검색할 수 있도록 하려면 <code>~/.bashrc</code>를 vim으로 수정하여 <code>stty stop undef</code>라고 추가해 주어야 한다.
  - bash 설정을 추가한 뒤에는 Ctrl + S로 정방향 검색이 가능해진다.  

9. __오래전에 실행한 명령어를 호출하곳 싶어(명령어 이력 검색)__
  - 명령어 이력에는 오랫동안 사용하지 않은 명령어는 남아있지 않는다.
  - 명령어의 기록 기간을 더 늘리는 법
      - <code>vim ~/.bashrc</code>
      - shift+G를 이용하여 파일의 제일 마지막으로 이동한뒤 하단과 같이 작성(숫자는 둘다 동일하게)
      - <pre>
          export HISTSIZE 10000 //메모리에 저장할 이력의 최대 건수
          export HISTFILESIZE=10000 //.bash_history에 저장할 이력의 최대 건수
        </pre>
  - bash는 실행되면 명령어 이력을 .bash_history라는 파일에서 읽어온뒤 메모리에 저장
  - 그 다음부터 실행 명령어를 찾게 될경우 저장된 메모리에서 검색한다음 추가로 실행되 명령어 또한 메모리에 저장
  - bash가 종료될때 메모리에 있던 명령어 내역을 .bash_history에 저장하여 동기화 한다.
  - tmux를 실행하여 화면을 분할할경우 각각의 bash에서 .bash_history파일을 읽어오고 각각의 메모리를 이용하기때문에
     분할 화면간 명령어 실행 내역이 공유되지 않는다.
  - 아래와 같이 설정하여 각각의 분활 tmux bash에서 명령어내역을 공유하도록 설정할 수 있다.
        <pre>
        vim ~/.bashrc
        
        //add
        function share_history {
            history -a
            history -c
            history -r
        }
        PROMPT_COMMAND='share_history'
        shopt -u histappend
        </pre>
        
10. __네트워크 건너서 파일을 복사하고 싶어 (scp)__
  - scp : Secure CoPy(네트워크를 통해서 파일을 복사)
  - <code>scp A B //A파일을 B로 복사</code>
  - <code> scp ./file.ext 로그인할사용자@접속할 컴퓨터:복사할 경로</code>
  - -r(recursively) 옵션 등을통해 폴더 하위 파일들까지 전부 복사 가능

11. __시스템 과부하를 파악하고 싶어(top)__
  ![](/assets/images/posts/2020-04-28/top.png){: width="80%"}
  - top 명령어를 통해 현재 시스템 사용 정보를 알 수 있다.
  - 이 중, load average를 통해 부하상태를 쉽게 알 수있다.(CPU가 처리하길 기다리는 작업의 개수를 나타냄, 1분당 평균으로 나타냄)
  - %CPU란과 Time+를 함께 살펴보아야한다 얼마나 높은 점유율을 얼마동안 사용했는지를 통해 부하를 판단해야하니까
  - c키를 눌러 실행된 command를 좀 더 상세히 볼 수 있다.
  - q를 눌러 top명령어를 종료 가능
  - 종료할 프로세스의 PID를 <code>kill pid</code>명령어를 통해 종료 가능
  - load average 는 처음부터 1분간, 5분간, 15분간 평균적으로 쌓이는 일의 양을 나타냄
  - cpu가 멀티 코어일 경우는 load average >= cpu 코어개수 가 되면 평균적으로 과부하 상태로 볼 수 있다.

12. __시스템 메모리 부족을 파악하고 싶어(top 표시 전환)__
  - Cpu부하가 낮아도 load average가 높은 경우 : 메모리 부족
  - cpu는 작업 장소로 메모리를 사용한다. 메모리에 여유 공간이 부족하게 되면 최근 사용하지 않고 있는 메모리의 데이터를 하드디스크에 임시로 저장한뒤(스왑아웃),
    해당 데이터가 필요할 경우 다시 임시 저장 데이터를 메모리로 로드하는데(스왑 인) 스왑은 os가 알아서 관리하지만 빈번하게 발생할 경우 I/O대기 시간이 길어진다.
  - 바로 이러한 이유로 메모리 부족시 load average가 증가한다.
  - 필요할때만 실행해서 처리가 끝나면 종료되는 것 : 프로세스
  - 컴퓨터 실행 동안 계속 프로세스를 실행시켜야 하는 것 : 서비스
      - 서비스의 실행,종료는 전용 기동 스크립트를 사용한다.
      - <code>sudo service apache2 restart</code> 기타 여러 스크립트가 있다. 
  - load average가 높고 스왑이 많이 발생하여 os의 작동에 문제가 발생할 경우 os가 자체적으로 알아서 프로세스들을 강제 종료 시킨다.
  - 정리하면
    - load average가 높아도 cpu는 과부하 상태가 아닐수가 있다(cpu의 코어가 많을 경우..등)
    - 빈 메모리가 부족 -> 스왑이 자주 발생 -> cpu처리가 쌓이고.. -> load average가 높아짐
    - top 명령어 출력 내용을 정렬해 보자
      - <code>shift + m //메모리 사용량 순서로 정렬</code>        
      - <code>shift + t //cpu 시간 순서 정렬</code>
      - <code>shift + p //cpu 사용량 순서정렬로 돌아가기</code>

13. __로그 파일에서 필요한 줄만 뽑고 싶어__
  - <code>grep "찾을내용" "찾을 파일"</code> -r옵션 없이 사용하여 해당 파일안에서 일치하는 내용을 검색할 수 있다.
  - 예를들어 로그파일에서 특정 내용을 찾아서 전부 검색하고 싶을 경우
    1. <code>Ctrl + b</code> 를 이용하여 tmux를 실행하고 <code>grep</code> 을 이용하여 검색한다
    2. 그다음 shift + \를 통해 | 키를 입력한다. 해당 키는 직전에 받은 명령어와 다음에 나오는 명령어를 연결해준다.(이를 파이프라인이라 한다.)
    3. <code>grep "log to find" logFile.txt | less</code> less는 파일의 내용을 출력하는 명령어로 파일 이름과 함께 사용하지만 | 명령어를 통해 열어야할 파일 지정을 전달받기 때문에 생략한다.
    4. <code>grpe "/retro" access.log | grep -v "/live | less</code> 이런 식으로 파이프라인을 계속 확장할 수 있다.(v는 문자열을 제외한 내용을 검색한다.)
    - <code>cat</code> : 파일 내용을 그대로 읽어서 출력
    - <code>zcat</code> : gzip형식(.gz, .tgz)의 압축 파일의 내용을 출력
    - <code>xzcat</code> : xz형식 용(.xz 파일)
    - <code>unzip</code>
    - 현재 추가되는 로그만 보고 싶어
    - <code>tail -F access.log</code> : tail은 파일의 끝부분만 출력한다 -F를 통해 파일에 변경된 내용들만 출력할 수 있다.
    - tail명령어와 grep을 파이프라인으로 연결하면... : 원하는 내용이 추가될 경우만 실시간 확인이 가능!!
  - 텍스트를 다루는 명령어를 그룹화해 보자
    - 파일 내용을 다음 명령어에 출력하는 시작 그룹
      - cat / zcat / xzcat / tail -F 등
    - 중간에서 이전 명령어 출력을 가공하는 중간 그룹
      - grep(해당하는 라인만 출력) / sort(정렬) / cut(잘라내기) / uniq(중복 제거) / sed,awk(내용 변경)
    - 이전 명령어 출력을 가공하는 최종 그룹
      - less(스크롤 할 수 있게 출력) / tee(파일을 저장) / wc(줄 수나 문자 수를 카운트) / head(첫 부분만 추출)

14. __작업 절차를 자동화하고 싶어(셸 스크립트)__
  - 파이프 라인이 명령어를 가로로 연결한다면 스크립트는 세로로 연결한다.
  - sudo vim setup.sh라는 명령어로 파일을 생성한다.
      <pre>
      #!/bin/bash --해당 스크립트를 실행할 인터프리터를 지정한다
      순차적으로 작업할 내용을 작성한다
      </pre>
  - 작성이 끝난 뒤에는 실행 권한을 주어야 한다. <code>chmod +x setup.sh</code>
  - 파일 실행할때 <code>/bin/</code>이나 <code>/usr/bin/</code>경로 하위에 파일이 있는것이 아니라면 전체 경로를 입력하여야 실행이 된다.
  - 만일 해당 스크립트가 자동 실행중 에러가 발생한다면?
    - <code>if [ $? != 0 ]; then exit; fi</code>스크립트가 문제가 발생하면 종료하는 명령어를 추가할 수 있다.
    - exit를 실행해도 명령어를 실행한 터미널이 종료되는것이 아닌 스크립트를 실행할때 작동하는 별도의 프로세스가 종료되는 것이다.

15. __같은 문자열을 스크립트에서 재사용하고 싶어(셸 변수)__
  - vim에서 잘못된 내용을 일괄 수정할 때
    - vim 노멀 모드에서<code>:%s/원문/수정문/</code>과 같이 입력하면 해당 파일내의 모든 원문이 수정문으로 일괄 수정된다.
    - 반복적으로 사용되는 문자열/파일명 등은 변수에 할당하여 재사용할 수 있다.
      - <code>변수명 = 할당할 내용</code>
      - <code>$변수명 or ${변수명}</code>을 통해 호출 가능하다.
      - 스크립트 안에서도 변수를 할당하고 변수명을 통한 재사용이 물론 가능하다.
    - 실행 명령문 또한 변수에 할당이 가능하다
      - <code>tar_extract="tar xfv"</code> tar xfv 파일명 : 파일 압축 풀기
      - <code>tar_compress="tar cfv"</code> tar cfv 파일명 : 파일 압축하기
      - <code>eval</code> eval은 문자열을 명령어로 실행해준다.
      - 즉 <code> eval "$tar_extract file.tar.gz"</code>와 같이 실행 가능하다.
    - 변수명에 변수명을 할당 가능하다.
      <pre>...
      base=/var/log/apache2
      latest=${base}/access.log
      prev=${base}/access.log.7.gz
      ...
      </pre>

16. __작업 환경과 상태를 정해서 스크립트를 실행하고 싶어(환경 변수)__
  - 환경 변수 : 따로 정의하지 않아도 $변수명 으로 값을 참조 할 수 있는 특수한 변수.
  - <code>${HOME} OR $HOME</code> 사용자 홈 디렉터리 참조.
  - <code>env</code> 명령어로 실행가능한 모든 환경변수를 알 수 있다.
  - 주요 환경변수
    - <code>HOME</code> : 사용자 홈 디렉터리 경로
    - <code>PWD</code> : 현재 디렉토리 경로
    - <code>EDITOR</code> : 정해진 텍스트 에디터(vim, nano 등) 경로
    - <code>PAGER</code> : 정해진 페이저(less, lv 등) 경로
    - <code>USER</code> : 현재 사용자의 사용자명
    - <code>GROUP</code> : 현재 사용자의 그룹명
    - <code>HOSTNAME</code> : 현재 머신의 호스트명
  
  - 명령어 치환
    - <code>$(명령어열) or `명령어열`</code>로 작성시 해당 명령어열의 결과가 문자열로 치환된다.
    - 특정 파일을 그 날짜명으로 변경하는 스크립트를 설정해야 할 경우
      - <code>date</code> 입력시 날짜 정보 출력 <code>Thu May 21 22:53:26 KST 2020</code>
      - <code>date +%Y-%m-%d</code>와 같이 형식 지정 가능
      - <code>mv result.txt result-$(date +%Y-%m-%d).txt</code>와 같이 작성 가능하다.  
   
17. __로그 파일에서 필요한 줄만 뽑고 싶어(cut)__  
  - 아파치 로그에서 접속자 수가 많은 페이지와 적은 페이지 목록을 출력한다고 가정하면 집계 대상만 추린다.
    - <code>cat /var/log/apache2/access.log | grep -v "/live"</code> 실시간 로그는 제외하고..
      - 출력을 파이프라인을 통해 다음 명령어에게 전달한다.
      - grep -v옵션은 지정한 문자열을 제외한다.
      - 파일 확장자가 xz라면 xzcat을 zip이라면 zcat으로 내용을 읽는다.
  - 로그 각 줄에서 접속한 페이지 경로를 추출한다.
      - 로그 파일을 살펴보면 원하는 정보가 출력되는 위치의 특정 패턴이 있을것이고 이를 cut명령어를 이용하여 정리한다.
      - <code>cut -d "구분자" -f 추출위치</code> 즉 해당 줄을 구분자로 나누고 -f 뒤의 숫자를 통해 해당 위치번째의 내용을 선택한다.
      - <code>127.0.0.1 - - - /path/to/find</code>가 로그에 출력될 경우 <code>cut -d " " -f 5</code> 로 /path/to/find만 선택할 수 있다.
  - 기타
      - 변수로 지정한 문자열 또한 cut을 통해 접근이 가능하다 <code> echo "$PWD" | cut - d "/" -f 2</code> echo는 인수로 넘어온 문자열을 출력만 한다.
      - cat은 여러 파일들의 내용을 한번에 출력하는 명령어로 인수로 여러개의 파일을 주면 모든 파일을 순차적으로 이어서 출력한다.
      - tac는 출력 순서를 마지막 부터 역순으로 출력한다.

18. __같은 내용의 줄을 세어보고 싶어__
  - 17번에 이어서 이제 원하는 로그의 내용만 출력이 되서 나올 것이다
  - <code>cat /var/log/apache2/access.log | grep -v "/live" | cut -d " " -f 7</code>
  - <code>sor</code> : 알파벳 순으로 정렬
  - <code>uniq</code> : 중복을 제거 하되 이어서 출력되는 중복만 제거된다.
    <pre>
    test
    test
    and
    test
    test 
    </pre>
    - 위의 문자열을 uniq로 처리 시 다음과 같다.
      <pre>
      test
      and
      test
      </pre>
    - 즉, sort 후 uniq로 처리하면 모든 중복내용을 제거할 수 있다.
    - <code>uniq -c</code>옵션을 통해 중복된 내용이 몇번 나타났는지 count할 수 있다.
    - 이제 완성된 명령어는 다음과 같다.
    - <code>cat /var/log/apache2/access.log | grep -v "/live" | cut -d " " -f 7 | sort | uniq -c | less</code>
    - 이제 uniq -c 를 통해 카운트된 내용이 출력되면 카운트 수를 기준으로 정렬하면 끝~.
    - sort는 앞에서부터 비교해서 정렬하기 때문에 카운트 수를 기준으로 정렬하기 위해 sort를 한번더 사용하고 이때 역순으로 정렬할 경우 -r 옵션을 준다.
    - <code>cat /var/log/apache2/access.log | grep -v "/live" | cut -d " " -f 7 | sort | uniq -c | sort -r | less</code>
    - <code>tail and head</code>을 추가하여 가장 빈번하게 등장했던 로그내용과 가장 적었던 내용을  출력할 수 있다.
      - <code>tail</code> : 입력 끝에서 10줄만 추철
      - <code>head</code> : 입력 앞부분에서 10줄만 추철
        - 둘다 -n 옵션으로 출력할 줄의 개수를 조절할 수 있다
    - <code>cat /var/log/apache2/access.log | grep -v "/live" | cut -d " " -f 7 | sort | uniq -c | sort -r | tail | less</code>
    - 스크립트로 작성된 예시
      <pre>
      #!/bin/bash
      log=/var/log/apache2/access.log #집계 대상의 위치를 변수로 설정
      count=10 

      echo "접속수가 많은 ${count}개 페이지."
      cat $log | grep -v "/live" | cut -d " " -f 7 | sort | unic -c | sort -r | head -n ${count}
      </pre> 
19. __CSV 파일을 열의 내용에 따라 정렬하고 싶어(sort와 리다이렉트)__
  -  csv 파일은 일반 텍스트 파일이므로 문자열 조작 명령어로 가공가능하다.
  -  불필요한 내용을 삭제하고 특정 기준으로 재정렬하여 결과를 다시 파일로 만들면 된다.
    - 불필요한 열 삭제하기        
      - cut 명령어의 -f는 여러 숫자를 지정하거나 범위를 지정하는것도 가능하다.
      - <code> cat file.csv | cut -d "," -f 1-3</code> csv파일은 , 구분자인 텍스트 이므로 이를 기준으로 나누고 지우고자 하는 란을 제외한 범위를 cut을 이용하여 제거한다.
      -  sort 명령어 또한 특정 내용을 기준으로 정렬할 수 있는데 cut과 동일한 내용을 뜻하지만 명령어는 다르므로 주의해야 한다.
      - <cod>sort -t "," -k 3</code>-t로 구분자를 -k로 위치를 지정한다.
      - sort 사용시 1.~ , 2, .... , 11, 등이 있을경우 옵션울 주지 않으면 앞에서부터 차례대로 비교하여 1. 11. .... 2와 같이 비교하게 된다. 이럴 경우 숫자 정렬로 인식한다는 -n 옵션을 추가하면 된다.
      - 리다이렉트를 이용하여 명령어의 결과를 파일로 저장한다.
        - <code> > </code> 이미 파일이 있으면 지우고 새로운 파일로 대체한다.
        - <code>>></code> 이미 파일이 있으면 파일의 마지막에 내용을 추가한다.

20. __명령줄 지정으로 작업 내용을 바꾸고 싶어__
  - 스크립트를 실행할 때 명령어 라인 인수를 통해 스크립트 실행시 마다 인수를 달리할 수 있다.
  - 스크립트 파일 안에서 <code>$1</code>과 같이 작성하고 스크립트 실행시 인수를 주면 해당 내용에 적용이된다.
  - 스크립트마다 처리할 파일명들이 달라질 경우 유용하게 사용 가능하다.
  <pre>
  #!/bin/bash
  echo "$1"
  </pre>
  - 위와 같은 스크립트가 있고 이를 /test.sh giung 과 같이 사용 가능하다.
  - 여러개의 명령어 라인 인수를 할당하고 스크립트에서 다른 변수로 접근하는 경우 아래와 같이 사용한다.
    <pre>
     #스크립트 실행시
     ./test.sh -d today -p java  -o /home/giung 
     #여러 변수를 할당하고 이를 스크립트에서 재 할당할 경우

     #!/bin/bash

     while getopts b: OPT
     do
      case $OPT in
        d) default="$OPTARG" ;;
        p) program="$OPTARG" ;;
      esac         #case를 거꾸로 적은것
     done

     echo ${default}
    </pre>

21. __조건에 따라 처리 흐름을 바꾸고 싶어(조건 분기)__
  - 스크립트의 대부분이 동일하지만 일부 조건에 따라 내용이 달라질경우
  <pre>
  if [ $# = 2 ]  # $#은 스크립트에 지정한 인수 개수를 의미한다. [ 사이는 띄어 써야 한다. ]
  then
    echo "Hello!"
  fi # if를 거꾸로한 것으로 처리가 종료된 것을 의미한다.
  </pre>
22. __명령어 이상 종료에 대응하고 싶어(종료상태)__
  - <code>$?</code> 21번에 작성한 if문에 $?를 사용할 수 있고 이는 바로 전에 실행한 명령어 종료 상태이다. 0은 성공을 뜻한다.
  - 콘솔 창에 ll과 같이 간단한 명령어를 실행하고 <code>echo $?</code>를 입력하면 성공시 0이 출력될 것이다.
  - <code>if [ $? != 0 ]; then exit; fi</code> if문을 1줄로 작성하고자 한다면 줄 바꿈대신 ;를 사용하면 여러줄을 한줄로 작성 가능하다.
  - <code>exit 0 or 1</code>if문 마지막 exit에 0 또나 1을 인수로 주면 그것이 해당 스크립트의 종료 상태가 된다. 0은 정상 1은 비정상 종료이다.

23. 같은 처리를 반복해서 실행하고 싶어(for)
  - 여러 파일에(파일명은 다르지만) 같은 처리를 반복해서 적용하고자 할 경우
    <pre>
    #!/bin/bash
    
    for filename in redmine.log kintai.log download.log
    do
      ./create-report.sh $filename
    done
    </pre>
  - for문 안에서도 명령어 치환이 가능하다.
    <pre>
    #!/bin/bash

    for filename in cd /var/logs; ls *.log
    do
    ~
    done
    </pre>   
    - if문과 for문 한줄 작성시 주의할점 
      - <code>for file in data log scripts; do echo $file; done</code> do 뒤에는 ;를 붙이지 않는다.
      - <code>if [ $1 = "" ]; then echo $1; fi</code> then 뒤에는 ;를 붙이지 않는다.
  
  24. 공통 처리를 계속 재사용하고 싶어(셸 함수)  
    - 셸 스크립트 일부에 이름을 붙여 함수로 만들 수 있다.
    <pre>
    #!/bin/bash

    hello() {
      echo "$1"
      ..처리..
      .. 처리

      return 1  # return을 만나면 함수가 종료된다.
    }

    hello giung # 함수 호출 + 인수값 전달
    </pre>  
    - if문에서 exit는 스크립트의 종료상태를 반환하지만 함수에서 return은 함수의 종료상태를 반환한다.
    <pre>

    tody() {
      date +%Y-%m-%d
    }

    echo $(today) #함수 실행 결과는 명령어 실행 결과의 문자열이 된다.
    </pre>
