### Holoman 객체 자동 설정 만들기

---

    프로젝트를 생성하고 아래의 소스코드를 입력하고 Maven -> install 하면
    MAC OS 기준 /Users/유저명/.m2 하위 디렉토리에 해당 프로젝트 pom.xml 에서 설정한
    groupId 경로 artifactId 명으로 폴더 및 라이브러리 구성파일이 생성된다
    그리고 자동 설정하기 위한 프로젝트 pom.xml 에 위에서 생성한 라이브러리를 등록해주고
    application.properties 에 name, date 값을 할당하여 프로젝트를 시작하면
    properties 에서 입력한 값을 확인할 수 있다

```java
public class Holoman {
    private String name;
    private int date;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getDate() { return date; }
    public void setDate(int date) { this.date = date; }

    @Override
    public String toString() {
        return "Holoman{ name='" + name + '\'' + ", date=" + date + '}';
    }
}

@Configuration
//@ConfigurationProperties 로 선언되어 있는 객체에서 받은 값을 주입
@EnableConfigurationProperties(HolomanProperties.class)
public class HolomanConfiguration {
    @Bean
    @ConditionalOnMissingBean //동일한 빈이 이미 등록되어 있으면 이 빈을 등록하지 않도록 하는 어노테이션
    public Holoman holoman(HolomanProperties properties) {
        Holoman holoman = new Holoman();
        holoman.setName(properties.getName());
        holoman.setDate(properties.getData());
        return holoman;
    }
}

//properties 에 입력되어 있는 값을 주입
//속성값은 properties 에서 접근할 때 사용하는 prefix
@ConfigurationProperties("holoman")
public class HolomanProperties {
    private String name;
    private int data;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getData() { return data; }
    public void setData(int data) { this.data = data; }
}
```

META-INF > spring.factories 의 자동 설정 경로  
(자동 설정 파일 경로 : me.dcun, 클래스 : HolomanConfiguration )

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
me.dcun.HolomanConfiguration
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.dcun</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure-processor</artifactId>
            <optional>true</optional>
            <version>2.2.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
            <version>2.2.6.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```
