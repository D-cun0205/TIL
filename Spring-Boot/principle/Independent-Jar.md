### 독립적으로 실행 가능한 JAR

---

    mvn package 를 진행하면 실행 가능한 JAR 파일 생성
        * mvn clean : target 폴더(하위포함) 제거
        * mvn clean 시 경로가 프로젝트 경로에서 시작 target 안에서 mvn clean 불가
    spring-maven-plugin 이 패키징을 진행
    java -jar 패키징 한 파일명을 입력하면 어플리케이션이 실행
    
JAVA 라이브러리 압축으로 uber jar 사용

    모든 라이브러리 파일들을 읽어서 하나의 클래스파일에 압축하여 사용
    어떤 내용이 어느 라이브러리에서 온지 확인 불가

스프링부트 전략

    압축하지 않고 lib 폴더안에 각 라이브러리(jar) 파일을 모아놓는다
    내장 jar 파일을 읽기 : loader > jar > JarFile.class 사용
    jar 파일을 실행 : loader > Launcher.class 사용