---
title : 프로젝트 - 리그오브레전드 전적검색 서비스 - 1
category : [Project, Spring]
tags : [Spring]
date : 2020-05-15T23:02:00Z
comments : true
author_profile: false
---
## 프로젝트를 시작해 봅시다.
- 주제 : 리그오브레전드 전적 검색 서비스
  - 개발 상세 스펙과 구현 목표는 [바로 여기](https://github.com/rldnddl87/lol-service)
  - 코드는 제 깃허브에 공개되어 있습니다.
> 개발 과정과 생각들을 블로그에 기록할 예정입니다.

---
## 프로젝트는 왜 만드는 건가요?
- 그동안 개발 공부를 하면서 느끼전 점은 인강을 듣고 책으로 코드를 보는것은 기억에 오래 남지 않는다고 느꼇습니다.
- 교육자 박재성님의 [영상](https://www.youtube.com/watch?v=fXIpMyrI3U8)과 노마드 코더스 니콜라스 의 [영상](https://www.youtube.com/watch?v=FF6CF8TZIhE&t=12s)에서도 강조하는 점은 결국 자기주도적으로 고민하고 스스로 개발 목표를 구현하면서 학습하는 것이 중요하다는 점입니다.
    > 거기다 채용 공고에서는 항상 서비스를 개발후 런칭한 점을 우대사항에 적어놓은 곳이 참 많습니다..

---
### 그럼 한번 해 봅시다. 

![](/assets/images/posts/2020-05-15/image01.jpg)

- 이번 포스트에서는
  - 라이엇이 제공하는 API를 이용합니다.
    - 챌린저 랭커 목록 제공 API([여기를 살펴봐 주세요!](https://developer.riotgames.com/apis#league-v4/GET_getChallengerLeague))
  - API 접근 키를 깃과 서비스 외부에 노출되지 않도록 프로젝트 외부에 설정파일을 작성합니다.
  - API주소를 자바의 Enum으로 다룹니다.
  - JPA를 이용하여 1:N관계를 설정하여 DB에 저장하고 조회합니다.  
  

- __Riot API 서비스 이용하기__
  - 리그오브레전드 회원 이라면 [Riot Developer Portal](https://developer.riotgames.com/) 로그인하면 임시 토큰을 발행해 줍니다.
   ![](/assets/images/posts/2020-05-15/image02.png)
  - 홈페이지 상단을 살펴보면
    - APIS : 제공하는 API상세 스펙
    - DOCS : API관한 정보
    - POLICIES : API를 사용하면서 지켜야할 정책
    - STATUS : 현재 API 서버와 라이엇 서버의 상태정보
    - CHANNELS : 라이엇 API를 이용하는 개발자들의 소통공간 정보
  - API란을 클릭한뒤 살펴보면 아래와같은 형식으로 정보를 요청합니다.
     ![](/assets/images/posts/2020-05-15/image03.png)

- __API접근에 필요한 API KEY를 저장하겠습니다.__
  - vim을 이용하여 파일을 생성합니다 파일의 확장자는 .yml로 작성합니다.
  ![](/assets/images/posts/2020-05-15/image04.png)
  - 저는 <code>/app/config/app-config.yml</code> 경로에 생성하였습니다.
    > 윈도우 환경에서 개발하시는 분들이라면 wsl2 + vscode(remote wsl extension)를 사용해보시길 권장합니다. 위와 같이 wsl2에 파일을 설정하고 /app/config와 같이 linux파일 경로로 접근이 가능합니다.  
    모든 환경 구성을 wsl2(리눅스)에 셋팅하고 vscode를 통해 윈도우 환경에서 편집 및 접근이 가능합니다.  
    wsl2를 사용하고 나서부터는 맥os를 써야할 이유를 전혀 느끼지 못하고 있습니다.  
    현재 intellij에서는 wls지원이 아직 원할하지 않은 것으로알고 있습니다.  
    저도 얼마전까지 intellij를 사용하다 vscode를 사용 중입니다.

    ![](/assets/images/posts/2020-05-15/image05.png)
    [이미지 출처](https://intellij-support.jetbrains.com/hc/en-us/community/posts/360004275400-Developing-in-Windows-Subsystem-for-Linux)

- __Spring Boot로 프로젝트를 생성합니다.__
  - 의존성은 우선 Spring Web / Data Jpa / webflux / h2 / lombok 을 추가해 생성합니다.
  - 기존의 application.yml이외에 위에서 작성한 app-config.yml파일에 접근 가능하도록 스프링 애플리케이션 빌더를 이용하여 아래와 같이 설정합니다.
  ![](/assets/images/posts/2020-05-15/image06.png)  
  - 설정파일을 자바 코드로 접근가능하도록 @ConfigurationProperties를 이용하여 설정합니다.
  ![](/assets/images/posts/2020-05-15/image09.png)
    - 이제 RiotProperties가 빈으로 등록되었습니다.  
  
  - __이제 API주소를 이용해서 호출해야 합니다.__
    - API URL을 저장하는 방식에는 2가지가 우선 떠올랐습니다. DB와 Enum입니다.
    - 값이 자주 변경된다면 DB를 이용하는 방식을 사용했겠지만 API 목록을 살펴보니 큰 변경이 없을것 같아 Enum으로 관리하기로 했습니다.
    ![](/assets/images/posts/2020-05-15/image07.png)
        > Enum 의 장점은 컴파일 시점 확인이 되기 때문에 타입 세이프하게 사용이 가능하고 의도와는 다른 값이 변수에 할당되는 점을 방지하며, 코드의 가독성을 좋게 합니다.
    - 해당 Enum을 이용하여 url을 생성해냅니다.
    ![](/assets/images/posts/2020-05-15/image08.png)
    - 외부 API를 호출기 위해서는 두 가지 선택지가 있습니다.
      - RestTemplate 와 WebClient
        - RestTemplate : blocking / synchronus
        - WebClient : non-blocking / asynchronous
      - 어떤걸 사용할까 검색해보고 ...
        - [The Guide to RestTemplate](https://www.baeldung.com/rest-template) 에서 RestTemplate가 곧 Deprecated된다는 내용을 확인했습니다.
        - 백기선님 강의 중..WebClient를 추천하는것을 보고 WebClient를 사용하기로 했습니다.
      - WebClient의 경우 RestTemplate와 마찬가지로 해당 빈을 직접 등록해주지 않고 Builder를 Bean으로 등록해주기 때문에 우리는 Builder를 커스텀하게 설정한뒤 WebClient 인스턴스를 생성해 사용하면 됩니다.
        - 모든 API중 공통되는 부분이 있습니다. 바로 <code>https://kr.api.riotgames.com/</code> 입니다.
        ![](/assets/images/posts/2020-05-15/image10.png) 
        - 공통 URL인 https://kr.api.riotgames.com/ 을 BASE로 하여 WebClient의 base url을 설정하고, 외부 설정파일에 작성하였더 Api key를 상단에서 설정하였던 RiotProperties를 빈으로 주입받아 헤더에 설정해 줍니다.  
         
    - API 설정 키값과 WebClient를 통해 url을 호출할 준비가 끝났습니다. 이제 응답 결과를 저장할 Dto와 DB저장에 사용할 엔티티를 설정해야 합니다.
      - Riot API에서는 데이터를 json형태로 반환하는 dto에 대한 명세를 제공합니다.
       ![](/assets/images/posts/2020-05-15/image11.png)  

    - __Dto와 엔티티 생성하기__
      - 엔티티와 Dto의 차이점은 엔티티는 setter를 작성하지않아 값이 무분별하게 변경되는 점을 방지합니다. 모든 데이터의 가공은 dto를 이용하여 가공되고 DB에 반영되기 전에 Dto를 통해 엔티티를 생성하고 반영합니다.
      - LeagueList Entity
        <script src="https://gist.github.com/rldnddl87/4aaba6ac13328fe939c185d1c859a04a.js"></script>
      - LeagueItem Entity
        <script src="https://gist.github.com/rldnddl87/986c85001f4d027eab45da0665748aae.js"></script>
      - LeagueListDto
        <script src="https://gist.github.com/rldnddl87/6326d53a85f0772e0cb39d22a8bda2fd.js"></script>
      - LeagueItemDto
        <script src="https://gist.github.com/rldnddl87/21073efcd6cf09e9c40596aef0ee335c.js"></script>   

      - 다음과 같은 규칙으로 만들었습니다. 
        - 엔티티
          - 엔티티는 기본생성자의 경우 protected로 설정하여 외부에서 생성자를 통한 무분별한 객체의 생성이 불가능하고 오로지 builder패턴을 이용하여 값을 설정하여 생성한다.
          - Api 명세서를 살펴보면 알 수 있듯이 LeagueList와 LeagueItem은 1 : N의 관계를 가지고 있기 때문에 엔티티에도 연관관계를 적용해 주었습니다.
        - Dto
          - getter와 setter를 설정하여 스프링의 기본 HttpMessageConverter를 통해 데이터가 맵핑 될 수 있도록 하였습니다.
          - Dto에 toEntity()라는 메서드를 작성하였고 이는 Dto의 값을 엔티티의 빌더를 이용하여 엔티티를 생성합니다.

    - __Api를 실제로 호출해 보겠습니다.__
       - 우선 임시로 작성한 컨트롤러를 통해 이미지의 saveChallengerRanking() 서비스를 호출할수 있도록 합니다.
        ![](/assets/images/posts/2020-05-15/image13.png)
        ![](/assets/images/posts/2020-05-15/image12.png)
        > 해당 작업은 추후에 배치작업과 같이 일정 시간마다 호출되어 db를 업데이트 하도록 변경할 예정입니다.
       - 메서드를 살펴보면 이전에 WebClientBuilder를 통해 설정한 헤더와 baseUrl에 추가로 상세 API주소를 추가합니다.
       - get방식으로 호출한뒤 데이터를 DTO에 저장합니다.
       - JpaRepository를 이용하여 db에 저장합니다.
       - 우선 메모리 db인 h2를 이용하였습니다.
     - /get을 통해 localhost를 호출하고 h2-db를 확인해보면..
       - LeagueList와 LeagueItem모두 값이 정상적으로 insert 되었습니다.
       ![](/assets/images/posts/2020-05-15/image14.png)
       ![](/assets/images/posts/2020-05-15/image15.png) 
    - 추가적으로 db를 조회한 결과(엔티티)를 리그 포인트(랭킹 판별 기준)를 이용하여 정렬하고 dto에 맵핑하여 리턴해주는 코드를 작성합니다.
        <script src="https://gist.github.com/rldnddl87/0390abb46494cb3cfe5d19f69a059eb2.js"></script>  
- __자 이제 포스팅을 마무리 하면 될까요...?__
    > 먼가 느낌이 쎄 합니다...
    
![](/assets/images/posts/2020-05-15/image16.gif){: width="50%"}  

--- 
### TDD!! TDD!! TDD!! 테스트 해야합니다.

- 테스트 코드는 DB에 데이터를 저장하는 것이 아닌 서버를 호출하여 200응답이 떨어지는지 확인하겠습니다.
    <script src="https://gist.github.com/rldnddl87/b2331a9cb87f03ed657b8186d41a54dc.js"></script>
    - 테스트 환경에서 설정 파일을 조회할 수 있도록 합니다.
    - 테스트 환경에서 외부 url을 호출하기 위해 apache httpclient를 사용하였습니다.
    - 상세 내용은 이전에 작성한 webclient와 같습니다.
    - url와 http method를 설정하고, api key와 헤더를 설정한뒤 호출하고 응답이 200이 떨어지는지 확인합니다.  
     
- JPA를 통한 데이터 저장/조회 테스트는 샘플 데이터를 Dto에 직접 설정하고 저장하고 조회하여 테스트 하겠습니다.
    <script src="https://gist.github.com/rldnddl87/6c9bd67f1d0dfbdfb9ed206db3db6fe0.js"></script>
    - @Before 어노테이션으로 테스트용 데이터를 저장합니다.
    - JpaRepository를 통해 데이터가 정상적으로 조회되는지 확인합니다.

--- 

- 최종적으로 db에 저장한 데이터가 json형식으로 잘 리턴되는지 확인해 보면..!!
    ![](/assets/images/posts/2020-05-15/result1.png) 
    
---
> 마무리
   

   
