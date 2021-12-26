### 데이터베이스 초기화, 마이그레이션

---

초기화

    NONE : none 이라는 값은 없으며 키워드가 아닌 값을 입력하면 none 과 동일하며 아무런 변화가 없다
    CREATE-DROP : create 의 기능 + 어플리케이션 종료 시에도 drop
    CREATE : drop -> create
    UPDATE : 변경 감지 및 적용
    VALIDATE : 변경이 감지 되면 변경 사항 출력 및 어플리케이션 종료

    UPDATE 를 사용할 경우 엔티티를 수정하면 기존 필드를 수정하지 않고 새로 생성된다

```properties
#datasource 정보 생략

spring.jpa.hibernate.ddl-auto=update, create, validate
spring.jpa.generate-ddl=true, false
spring.jpa.show-sql=true

#postgres 를 사용하면 발생하는 Clob 을 생성하지 않았다는 에러인데 우회하는 방법
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```

스크립트를 사용한 초기화

    /resources/schema.sql 안에 내용은 아래와 같다
    spring.jpa.hibernate.ddl-auto=validate 로 설정해도 schema.sql 로 인해
    데이터베이스 초기화 작업을 진행한다
    순서 1.schema.sql, 2.data.sql

```sql
drop table if exists account cascade
drop sequence if exists hibernate_sequence
create sequence hibernate_sequence start 1 increment 1
create table account (id int8 not null, email varchar(255), password varchar(255), username varchar(255), primary key (id))
```

마이그레이션

    데이터베이스 변경 사항을 버전으로 관리한다
    /resources/db/migration/버전 관리 파일
    생성규칙 : V + 숫자 + __(언더바 2개) + 이름 + .sql
    숫자는 순차적으로 관리한다 (타임스탬프 권장)

    중요 : 데이터베이스의 변경 사항이 생기면 기존 sql은 사용하지 않고 새로운 버전의 파일을 생성한다
    
버전 관리 파일 위치 변경

```properties
spring.flyway.locations=변경위치
```

스키마 버전 히스토리 테이블 자동 생성

```properties
spring.flyway.baseline-on-migrate=true
```