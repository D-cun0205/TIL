### 로깅

---

    로깅퍼사드 : Commons Logging, SLF4j
    로거 : Log4J2, Logback, JUL
    Commons Logging > SLF4j > Logback 을 호출하여 최종적으로 Logback 사용
    로그파일은 10MB 단위로 자동 파일 생성

Spring Boot 로깅 설정 방법

    디폴트 로깅의 Stack Trace 에는 INFO 레벨의 메시지를 출력
    Run/Debug Configuration 에서 VM Options(-Ddebug), Program arguments(--debug) 을
    설정하여 메시지 출력 레벨을 변경할 수 있다

application.properties 를 사용한 설정 변경

```properties
# 프로젝트 내부를 기준으로 파일명 또는 경로/파일 입력
logging.file=file
# 디렉토리명을 넣어주면 프로젝트폴더 안에 폴더, spring.log 파일을 생성 
logging.path=directory
# 특정 경로에 level 설정
logging.level.패키지.클래스=debug
# 프레임워크 관련 level 설정
logging.level.org.springframework=debug
```

```java
@Component
public class SampleRunner implements ApplicationRunner {

    Logger logger = LoggerFactory.getLogger(SampleRunner.class);

    @Autowired String hello;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        logger.info("=====================");
        logger.info("logger : " + hello);
        logger.info("=====================");
    }
}
```

로깅 커스텀

    main > resources > logback-spring.xml 파일 생성
    로깅 설정을 변경해서 성능 이슈와 관련된 속성들을 제거하거나
    디폴트 설정을 변경해서 로깅을 다른방식으로 출력

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cofiguration>
    <include resource="org/springframework/boot/logging/logback/base.xml" />
    <logger name="패키지명" level="레벨"/>
</cofiguration>
```
































