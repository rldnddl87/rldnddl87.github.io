---
title : Spring Boot + 외부 설정파일 이용하기
category : [Spring]
tags : [Spring]
date : 2020-04-27T23:02:00Z
comments : true
author_profile: false
---
- 개발 시 클라우드 접속 권한 정보나 외부 API 키값은 보안상의 이유로 JAR파일 외부에서 관리하는게 효과적
- 커스텀한 키와 밸류(properties)를 깃헙과 같은 소스코드 저장소에 노출시키지 않고 관리해보자.

## Spring Boot + external .properties(.yml) 접근 및 사용하기
---
1. commend line arguments / program arguments 사용하기
    - IDE 사용시 program arguments에 적용 가능하다.
    - 배포 후 jar파일을 실행시 커맨드라인 알규먼츠로 설정 가능하다.
<pre>
java -jar app.jar --spring.config.additional-location=file:/application-api.yml
</pre>
> 윈도우 + WSL환경일 경우 WSL에서 파일을 생성하고(리눅스) IDE를 통해서 program arguments로 설정해 주어도 적용이 되지 않는다.
IDE는 윈도우 파일 시스템 경로를 찾기 때문  
또한, 기본 application.properties에 additional-location을 추가해도 제대로 인식이 되지 않는다.
이는 applicational-location이 default properties location(application.properties)보다 먼저 인식되기 때문이다.

2. SpringApplicationBuilder 사용하기
<pre>
@SpringBootApplication
public class SpringBootExternalPropertiesApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(SpringBootExternalPropertiesApplication.class)
                .properties("spring.config.additional-location=c:/application-api.yml")
                .run(args);
    }

}

</pre>

3. PropertySourcesPlaceholderConfigurer 이용하기
<pre>
@Configuration
public class AppConfig {

    @Bean
    public PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        PropertySourcesPlaceholderConfigurer properties = new PropertySourcesPlaceholderConfigurer();
        YamlPropertiesFactoryBean yaml = new YamlPropertiesFactoryBean();
        yaml.setResources(new ClassPathResource("application-api.yml"));

        properties.setProperties(yaml.getObject());
        properties.setIgnoreResourceNotFound(false);
        return properties;
    }
}
</pre>