### 웹 Jar, Index 페이지, 파비콘

---

Web Jar

```xml
<!-- Jquery -->
<dependency>
    <groupId>org.webjars.bower</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1</version>
</dependency>
<!-- script src 에 버전을 명시하지 않아도 되게 설정 -->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator-core</artifactId>
    <version>0.36</version>
</dependency>
```

```html
<!-- webjars-locator-core 메이븐 설정 전 -->
<script src="/webjars/jquery/3.3.1/dist/jquery.min.js"></script>
<!-- webjars-locator-core 메이븐 설정 후 -->
<script src="/webjars/jquery/dist/jquery.min.js"></script>

<script>
  $(function () {
     alert('Jquery 적용'); 
  });
</script>
```

Index 페이지

* 기본 리소스 위치
    * classpath:/static
    * classpath:/public
    * classpath:/resources
    * classpath:/META-INF/resources


    위에 나열한 기본 리소스 위치에 index.html 을 설정하고 루트로 요청하면
    index.html 을 클라이언트에 출력한다
    ex) resources/static/index.html

favicon

    파비콘 생성해주는 웹사이트에서 파비콘을 생성
    파비콘 파일을 기본 리소스 위치에 복사
    ex) resources/static/favicon.ico
    페이지 리로드해서 파비콘 변경 확인