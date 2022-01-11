### Message Converter

---

HTTP  메시지 컨버터

    (@RequestBody)요청 본문에서 메시지를 읽어들이거나 (@ResponseBody)응답 본문에 메시지를 작성할 때 사용

설정방법

    1.기본으로 등록해주는 컨버터에 새로운 컨버터 추가하기 extendMessageConverter
    2.기본으로 등록해주는 컨버터는 다 무시하고 새로 컨버터 설정하기 configureMessageConverter
    3.의존성 추가하여 컨버터 등록

```java
@Configuration
public class MessageConverterConfig implements WebMvcConfigurer {

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new MappingJackson2HttpMessageConverter());
    }

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new MappingJackson2HttpMessageConverter());
    }
}
```

의존성 추가하여 컨버터 등록하는 방법

    스프링부트는 의존성에 Jackson2 를 포함하고 있어 별도의 설정 없이 JSON 메시지 컨버터를 사용할 수 있다
    스프링 프레임워크를 사용하고 있는 경우 의존성을 별도로 추가해야 한다

내부적으로 등록되는 과정

    WebMvcConfigurationSupport 에서 메시지 컨버터가 의존성으로 등록되어 있으면 자동으로 사용할 메시지 컨버터로 설정한다
    예시)
    1.스태틱 변수
    private static final boolean jackson2Present;
    2.정적 초기자에서 의존성 찾아서 대입
    static {
		jackson2Present = ClassUtils.isPresent("com.fasterxml.jackson.databind.ObjectMapper", classLoader) &&
				ClassUtils.isPresent("com.fasterxml.jackson.core.JsonGenerator", classLoader);
	}
    3. addDefaultHttpMessageConverters 메소드에서 messageConverters.add 에 메시지 컨버터 생성하여 등록
    addDefaultHttpMessageConverters {
		if (jackson2Present) {
			Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.json();
			if (this.applicationContext != null) {
				builder.applicationContext(this.applicationContext);
			}
			messageConverters.add(new MappingJackson2HttpMessageConverter(builder.build()));
		}
        ...
    }

JSON 메시지 컨버터 테스트

    Spring Boot 를 사용하여 별도의 설정없이 테스트가 가능하다

```java
import javax.xml.transform.stream.StreamResult;
import java.io.StringWriter;

public class Person {
    String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}


@RestController
public class SampleController {
    @GetMapping("/hello")
    public String helloTest(@RequestBody Person person) {
        return "Hello " + person.getName();
    }
}


@WebMvcTest(SampleController.class)
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Autowired
    ObjectMapper objectMapper;

    @Test
    public void jsonTest() throws Exception {
        Person person = new Person();
        person.setName("spring");
        String jsonString = objectMapper.writeValueAsString(person);

        mockMvc.perform(get("/hello")
                        .contentType(MediaType.APPLICATION_JSON_VALUE)
                        .accept(MediaType.APPLICATION_JSON_VALUE)
                        .content(jsonString))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("Hello spring"));
    }

    @Test
    public void xmlTest() throws Exception {
        Person person = new Person();
        person.setName("spring");
        

        mockMvc.perform(get("/hello")
                        .contentType(MediaType.APPLICATION_JSON_VALUE)
                        .accept(MediaType.APPLICATION_JSON_VALUE)
                        .content(jsonString))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("Hello spring"));
    }
}
```

XML 메시지 컨버터 설정 및 테스트

```xml
<!-- jaxb 인터페이스 -->
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
</dependency>
<!-- jaxb 구현체 -->
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
</dependency>
<!-- 마샬링 & 언마샬링 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-oxm</artifactId>
    <version>${spring-framework.version}</version>
</dependency>
```

```java
@Configuration
public class MessageConverterConfig implements WebMvcConfigurer {
    @Bean
    public Jaxb2Marshaller jaxb2Marshaller() {
        Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
        //JDK 1.8
        jaxb2Marshaller.setPackagesToScan(Person.class.getPackage().getName());
        //JDK 1.9
        //jaxb2Marshaller.setPackagesToScan(Person.class.getPackageName());
        return jaxb2Marshaller;
    }
}

@WebMvcTest(SampleController.class)
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Autowired
    Marshaller marshaller;

    @Test
    public void xmlTest() throws Exception {
        Person person = new Person();
        person.setName("spring");

        StringWriter stringWriter = new StringWriter();
        Result result = new StreamResult(stringWriter);
        marshaller.marshal(person, result);
        String xmlString = stringWriter.toString();

        mockMvc.perform(get("/hello")
                        .contentType(MediaType.APPLICATION_JSON_VALUE)
                        .accept(MediaType.APPLICATION_JSON_VALUE)
                        .content(xmlString))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(xpath("person/name").string("spring"));
    }
}
```
