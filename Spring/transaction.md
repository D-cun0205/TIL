### Transaction

---

#### 선언적 트랜잭션

    코드 내에서 직접 트랜잭션을 관리, 트랜잭션 스크립트 방식의 코드를 탈피 가능, 트랜잭션 정보를 담은 메소드 파라미터 사용 제거
    트랜잭션 스크립트 : 하나의 트랜잭션 안에서 동작해야 하는 코드를 한 군데 모아서 만드는 방식 (트랜잭션 1 : 메소드 1)
    A, B, C 오브젝트 또는 메소드가 있을 때 트랜잭션을 구분하기 위해서는 각 각의 트랜잭션을 설정하면 되고
    A -> B or A -> B -> C 를 트랜잭션으로 묶기 위해서는 트랜잭션 전파 속성을 REQUIRED로 설정 후 순서대로 호출하면 된다

#### 트랜잭션 추상화와 동기화

    추상화 : 데이터 엑세스 기술과 트랜잭션 서비스 사이의 종속성을 제거하고 스프링이 제공하는 트랜잭션 추상 계층을 이용
                    트랜잭션 서비스 종류 또는 환경이 변경되도 트랜잭션을 사용하는 코드를 유지 가능
        - PlatformTransactionManager ( 스프링 트랜잭션 추상화의 핵심 인터페이스 ) : 트랜잭션 경계 지정에 사용
            보유 메소드 : getTransaction(매개변수 : TransactionDefinition), commit, rollback (매개변수 : TrasactionStatus)
        - TransactionDefinition : 트랜잭션의 네 가지 속서을 나타내는 인터페이스
        - TrasactionStatus : 현재 참여중인 트랜잭션의 ID, 구분 정보
    동기화 : 트랜잭션 추상화, 데이터 액세스 기술을 위한 템플릿을 가능하게하는 핵심 기능

#### PlatformTransactionManager 의 구현체 종류

DataSourceTransactionManager

    Connection의 트랜잭션 API를 이용하는 트랜잭션 매니저
    해당 매니저를 사용하기 위해서는 DataSource가 스프링 빈으로 등록 되어 있어야 한다
    JDBC API를 이용해서 트랜잭션을 관리하는 데이터 액세스 기술인 JDBC, myBatis, iBatis Dao에 적용 가능

```xml
<!-- DataSource를 사용할 Dao 패키지 -->
<bean id="memberDao" calss="...MemberJdbcDao">
    <property name="dataSource" ref="dataSource" />
</bean>

<!-- DataSource 설정 -->
<bean id="dataSource" class="org...SimpleDriverDataSource"> ... </bean>

<!-- Transaction 설정 -->
<bean id="transactionManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```

    SimpleDriverDataSource의 Connection은 사용 후 반납하고 재사용시 Connection을 새로 연결하는 구조
    Connection을 연결한 상태에서 지속적으로 사용하기 위해서 DataSource의 getConnection() 을 사용하지 않고
    Spring이 제공하는 static getConnection(DataSource) 을 사용하여 해결 가능
    
    레거시 DAO를 사용하여 스프링의 트랜잭션 매니저와 연동하는 방법
    TransactionAwareDataSourceProxy : 스프링이 관리하는 트랜잭션을 인식하기위해 해당 DataSource의 프록시로 감싸도록 지원
    레거시를 사용하여 Connection을 매번 생성하는것처럼 보이지만 스프링이 관리하는 Connection으로 되돌려준다

```xml
<!-- 프록시를 통한 Connection 재사용을 하기위한 클래스 선언 -->
<bean id="dataSource"
    class="org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy">
    <property name="targetDataSource" ref="targetDataSource"></property>
</bean>

<bean id="targetDataSource" class="org...SimpleDriverDataSource"></bean>
```

JpaTransactionManager

    JPA 에서 사용하는 트랜잭션 매니저, JTA로 트랜잭션 서비스 이용할 때는 필요없음
    DataSource 레벨의 트랜잭션 기능 제공

```xml
<!-- JPA 팩토리 빈 -->
<bean id="emf" class="org...LocalContainerEntityManagerFactoryBean"></bean>

<!-- JPA 트랜잭션 설정 -->
<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="emf" />
</bean>
```

HibernateTransactionManager

    하이버네이트 에서 사용하는 트랜잭션 매니저, DataSource 레벨의 트랜잭션 기능 제공

```xml
<!-- 하이버네이트 세션 팩토리 -->
<bean id="sessionFactory" 
      class="org.springframework.orm...AnnotationSessionFactoryBean">...</bean>

<!-- 하이버네이트 트랜잭션 설정 -->
<bean id="transactionManager" 
      class="org.springframework.orm...HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
</bean>
```

JtaTransactionManager

    하나 이상의 DB, 트랜잭션 리소스가 참여하는 글로벌 트랜잭션 적용시 사용
    프로퍼티 설정 없이 등록하면 디폴트로 등록된 JNDI 이름을 통해 서버의 TransactionManger, UserTrasaction을 찾는다
    JNDI(Java Naming and Directory Interface) : 디렉터리 서비스에서 제공하는 데이터 및 객체를 발견하고 참고하기 위한 API

```xml
<bean id="transactionManager" 
      class="org.springframework.transaction.jta.JtaTransactionManager">
</bean>
```

#### 트랜잭션 경계설정 전략

코드에 의한 트랜잭션 경계설정 (실제로는 많이 사용되지 않으며 테스트에서 사용할 때 유용함)

```java
public class MemberService {
    
    @Autowired 
    private MemberDao memberDao;
    
    private TransactionTemplate transactionTemplate;
    
    //트랜잭션 탬플릿에 트랜잭션 매니저 설정
    @Autowired
    public void init(PlatformTransactionManger transactionManger) {
        this.transactionTemplate = new TransactionTemplate(transactionManger);
    }
    
    public void addMembers(final List<Member> members) {
        this.transactionTemplate.execute(new TransactionCallback {
            //트랜잭션 안에서 동작하는 코드, 매니저와 연결된 DAO는 같은 트랜잭션에 해당
           public Object doInTransaction(TransactionStatus status) {
               for(Member m : members)
                   memberDao.addMember(m);
               return null; //리턴되기 이전에 예외가 발생하면 롤백 아닌경우 커밋
            }
        });
    }
}
```

#### 선언적 트랜잭션 결계설정

    코드에 영향을 주지 않으며 특정 메소드 실행 전 후로 트랜잭션이 실행, 종료되거나 기존 트랜잭션에 참여 가능

트랜잭션 AOP를 적용하는 첫번째 방법

```xml
<!-- 트랜잭션 어드바이스 설정 -->
<tx:advice id="txAdvice" transaction-manger="transactionManager">
    <tx:attibutes>
        <!-- 
            특정 메소드명들에게 특정 속성을 주고싶은 경우
            <tx:method name="get*" read-only="true" />
        -->
        
        <tx:method name="*" />
    </tx:attibutes>
</tx:advice>

<!-- 포인트컷, 인터페이스에도 적용 가능, MemberDao의 모든 메소드에 적용 -->
<aop:config>
    <aop:pointcut id="txPointcut" expression="execution(* *..MemberDao.*(..))" />
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut" />

    <!-- 
        포인트 컷이 하나의 어드바이저에만 사용되는 경우 aop:pointcut 생략 가능
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* *..MemberDao.*(..))" />
    -->
</aop:config>

<!-- 클래스 프록시 설정 -->
<aop:config proxy-target-class="true">
    <!-- 나머지 설정은 인터페이스를 사용하는 위와 동일하며 expression - exuecution에 클래스를 지정-->
</aop:config>
```

트랜잭션 AOP를 적용하는 두번째 방법

    @Transactional 어노테이션 이용하는 방법으로 설정 파일에 포인트컷, 어드바이스를 정의하지 않음
    트랜잭션이 적용될 타깃인 인터페이스, 클래스, 메소드에 어노테이션을 부여하고 트랜잭션 속성 지정
    @Transactional 을 사용하기 위한 사전 설정으로 <tx:annotation-driven /> 입력

```java
/** 
 * @Transactional 우선 순위 높은순 class method, class, interface method, interface
 * @Transactional의 프록시 설정 : <tx:annotation-driven proxy-target-class="true" />
 * 주의 : 인터페이스에 @Transactional을 붙이고 프록시 설정값을 넣으면 interface에서 class로 전파되지 않음
 *        상속이 불가능한 final class에는 적용할 수 없음
 */
@Transactional //default (readOnly=false)
public interface MemberDao {
    public void add(Member member);
    
    public void add(List<Member> members);
    
    @Transactional(readOnly=true)
    public Long count();
}
```

#### AOP 방식 : 프록시와 AspectJ

    not used proxy : 클라이언트 - 타깃 오브젝트, in use proxy : 클라이언트 - 프록시 - 타깃오브젝트
    클라이언트가 타깃오브젝트에 보내는 요청을 프록시가 가로채서 필요한 부분을 더하고 타깃오브젝트에 전달
    타깃오브젝트가 자신의 메소드를 호출할 때는 프록시가 작동하지 않음

    스프링 트랜잭션 AOP 주의사항
    다른 서비스 계층 오브젝트에서 add()를 호출하면 프록시를 통해서 add 메소드의 @Transactional이 호출되어 REQUIRES_NEW 설정값으로 트랜잭션이 호출되지만
    MemberService의 compleWork()메소드를 통해 들어오거나 add()를 통하지 않고 들어오는 경우는 클래스단위의 @Transactional이 프록시를 통해 생성되고
    이후의 add() 메소드가 호출되어도 프록시가 요청을 가로챌수 없어서 add()의 @Transactional 설정을 반영할 수 없음을 주의해야 한다
    위의 문제점을 해결하기 위한 방법으로는 AopContext.currentProxy()(권장하지 않음), AspectJ AOP(권장)를 사용할 수 있다
    * AspectJ를 사용하면 @Transactional을 클래스 레벨, 메소드 레벨에 부여

```java
@Transactional
public class MemberService {
    
    @Transactional(propagation=Propagation.REQUIRES_NEW)
    public void add(Member m) {...}
    
    public void compleWork() {...}
}
```

#### 트랜잭션 속성

    propagation(트랜잭션 전파) : 트랜잭션 시작, 기존 트랜잭션에 참여 
        REQUIRED(디폴트) : 새로 시작, 기존 트랜잭션이 있는 경우 참여
        SUPPORTS : 기존 트랜잭션이 있으면 참여, 없으면 생성하지 않음
        MANDATORY : REQUIRED와 비슷하나 새로 시작하는경우 예외 발생, 독립으로 트랜잭션 진행하면 안되는 경우 사용
        REQUIRES_NEW : 항상 새로운 트랜잭션 시작, 이미 진행 중인 트랜잭션은 보류 (JTA는 보류 설정 필요)
        NOT_SUPPORTED : 트랜잭션 사용 안함, 진행중인 트랜잭션은 보류
        NEVER : 트랜잭션 사용 안함, 진행 중인 트랜잭션이 있으면 예외 발생
        NESTED : 진행 중인 트랜잭션에 중첩 트랜잭션 시작, 상속 구조, 부모의 트랜잭션 결과에는 영향이 있지만 
                    자신의 트랜잭션 문제에대해 부모에게 영향을 주지 않음

    isolation(트랜잭션 격리 수준) : 동시에 여러 트랜잭션이 진행될 때 결과를 어떻게 노출할지 결정하는 기준
        DEFAULT : 데이터 엑세스 기술, DB 드라이버의 디폴트 설정 반영
        READ_UNCOMMITTED : 가장 낮은 격리 수준, 하나의 트랜잭션이 커밋 전의 상태가 다른 트랜잭션에 그대로 노출(장점으로 성능 극대화)
        READ_COMMITTED : 다른 트랜잭션이 커밋하지 않은 정보는 읽을 수 없다, 다른 트랜잭션이 수정 가능
                            처음 트랜잭션이 같은 로우를 다시 읽을 경우 다른 내용이 발견 될 수 있다
        REPEATABLE_READ : 하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정 불가능, 새로온 로우 추가 가능
        SERIALIZABLE : 가장 강력한 격리 수준으로 트랜잭션을 순차적으로 진행하여 동시에 같은 정보를 액세스 불가

    read-only, readOnly : 읽기 전용으로 트랜잭션 시작 후 INSERT, UPDATE, DELETE 쓰기 작업 진행시 예외 발생

    rollback-for, rollbackFor, rollbackForClassName(트랜잭션 롤백 예외) :
        런타임 예외 시 롤백되며 체크 예외지만 롤백 대상으로 삼아야 하는 경우 위 속성을 사용

    no-rollback-for, noRollbackFor, noRollbackForClassName(트랜잭션 커밋 예외) : 트랜잭션 롤백의 반대 개념