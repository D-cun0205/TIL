### JAVA

---

운영체제(Operating System)에 종속되어있지 않고 독립적으로 실행 가능한 프로그래밍 언어  
각 운영체제를 지원하는 JVM을 설치하여 JAVA를 독립적으로 실행한다


* Compile Time Environment
  * Compiler
* Runtime Environment
  * Class Loader
  * Interpreter
  * JIT(Just-In-Time) Compiler
  * Garbage Collector
  

#### Compiler

    자바(JDK, JRE)를 설치하면 bin 디렉토리 안에 javac.exe 라는 실행 파일로 설치된다
    자바 소스 코드(.java)를 자바 가상 머신(JVM)이 이해할 수 있는 자바 바이트 코드(.class)로 변환


#### Class Loader

    자바는 동적으로 클래스를 읽어오며 런타임 단계에서 모든 클래스 파일이 JVM 과 연결된다
    동적으로 클래스를 로딩해주는 역할
    운영체제에서 할당받은 메모리 영역(Runtime Data Areas)에 적재
    클래스로더 서브 시스템의 Loading, Linking, Initialization 을 진행하여 메모리에 적재된다

* Loading
  * System Class Loader (Boot Class Loader)
  * Extension Class Loader
  * Application Class Loader
* Linking
  * Verify
  * Prepare
  * Resolve
* Initialization



System Class Loader (Boot Class Loader)
    
    $JAVA_HOME/jre/lib 디렉토리 안에 rt.jar 및 JDK 의 핵심 클래스들을 로드하는 역할  
    rt.jar : JAVA에서 사용하는 클래스들이 압축된 JAR 파일

Extension Class Loader
  
    $JAVA_HOME/jre/lib/ext 디렉토리 안에 파일들을 로드 하는 역할  
    확장 가능한 클래스를 로드하여 런타임 단계에서 찾아 사용할 수 있다
    
Application Class Loader

    모든 어플리케이션 레벨의 클래스를 로드 하며 개발자가 작성한 소스코드도 이에 해당된다

Verify

    .class 파일 유효성 체크

Prepare

    static 변수에 대해 메모리 할당 및 기본값 지정

Resolve

    Class class = new Class(); 의 심볼릭 메모리 레퍼런스를 메소드 영역의 실제 레퍼런스로 교체 

Initialization

    Prepare 단계에서 생성한 메모리 영역의 static 변수들에 값을 할당하는 작업


#### Interpreter

    자바 컴파일러에 의해 변환된 자바 바이트 코드를 한줄 한줄 읽고 해석하여 실행한다

#### JIT Compiler

    번역된 코드를 캐싱해두고 똑같은 코드가 있으면 번역하지않고 캐싱해둔 값을 사용
    매번 기계어 코드 생성(자바 메서드 호출할 때)을 방지하고 인터프리팅 시간을 단축한다
    캐싱 - 컴파일 임계치(JVM 내에 있는 메서드 호출 횟수 + 메서드 루프를 빠져 나오기 위한 반복 횟수)
    인터프리터와 JIT Compiler 의 차이는 특정 루프작업을 예로들면 되는데
    인터프리터는 루프안에서의 매번 코드를 다시 번역하고 사용되는 반면
    JIT Compiler 에서는 임계치가 넘어서는 순간부터 캐싱된 코드로 대체하여 사용하므로 번역이 불필요해진다

#### Garbage Collector

    사용하지 않은 메모리에 대해서 자동으로 회수 진행