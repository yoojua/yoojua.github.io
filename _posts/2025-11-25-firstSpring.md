---
layout: post
title: "스프링 입문편"
---

# 1. 스프링 파일구조
📁.idea: intellij 설정 파일
📁gradle: gradle 파일
📁src - 📁main - 📁java: 실제 java 파일
📁src - 📁main - 📁resources: xml, properties, html
📁src - 📁test: test 코드

- git에는 코드 파일만 올라가고 나머지 빌드된 결과물등은 올라가면 안된다.
- 빌드 툴(Gradle, Maven)은 의존관계를 관리해준다.

---


# 2. 스프링의 세 가지 응답방식
컨트롤러에서 리턴 값으로 문자를 반환하면 뷰 리졸버(viewResolver)가 화면을 찾아서 처리한다.

1. 정적 컨텐츠  
   html 파일 그대로 내리준다. (`/static`)  
2. MVC 패턴  
   1. Model을 담아 View에서 렌더링  
   2. Controller는 비즈니스 로직과 서버를 다룬다.
   - 템플릿 엔진이 HTML을 동적으로 바꿔서 내린다.
   타임리프 템플릿을 사용하면 서버없이 바로 열어봐도 볼 수 있다.
3. API  
   JSON을 사용해 데이터로 바로 내려준다.

---

# 3. 스프링 빈과 의존 관계
1. 스프링 빈
Controller가 Service를 통해 데이터를 조회할 수 있는 것을 **의존관계가 있다**고 표현한다.
이때 **스프링 빈을 등록**해야 한다.

- 스프링 빈: 스프링 프레임워크에서 관리되는 객체
  - 스프링 컨테이너가 객체의 생성, 초기화, 소멸까지 관리해준다.

---
2. 스프링 빈을 등록하는 방법
![](https://velog.velcdn.com/images/mymy/post/a22a7b80-e2e8-4194-b572-b5f4f0e4daf0/image.png)

  **(1) 컴포넌트 스캔과 자동 의존 관계 설정**
  - `@Controller`를 생성하면 **스프링 컨테이너**가 생성되는데, 스프링 컨테이너에서 **스프링 빈**이 관리된다.  
  

- `@Autowired`가 있으면 스프링이 연관된 객체를 스프링 컨테이너에서 찾아서 넣어준다.  
  ⇒ 이렇게 객체 의존관계를 외부에서 넣어주는 것을 **DI(의존성 주입)**이라고 한다.  
  - 스프링 빈을 싱글톤으로 등록함
  싱글톤: 객체를 단 하나만 생성해서 여러 곳에서 공유하는 패턴

- 컴포넌트 스캔 원리  
  `@Component` 애노테이션이 있으면 **스프링 빈**으로 **자동 등록**된다.  
  `@Controller`, `@Service`, `@Repository` 등등도 이와 같다.  

```java
// ✅Controller
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller
public class MemberController {

    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}

// ✅Service
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}

// ✅Repository
@Repository
public class MemoryMemberRepository implements MemberRepository {}
```

---

  **(2) 자바 코드로 직접 스프링 빈 등록하기**
  3가지 방법이 있다. `필드 주입`, `setter 주입`, `생성자 주입`.
  - 생성자 주입을 권장한다.
  의존관계가 실행중에 동적으로 변하는 경우는 거의 없으므로 권장한다.
  - [주의] `@Autowired`를 통한 DI는 helloController, memberService 등과 같이 스프링이 관리하는 객체에서만 동작한다.

스프링 빈으로 등록하지 않고 **내가 직접 생성한 객체에서는 동작하지 않는다.**
  
  ```java
  @Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
  ```
  


> 실무팁
컴포넌트 스캔 -> 컨트롤러, 서비스, 레포지토리 같이 정형화된 코드
직접 설정 -> 정형화 되지 않거나, 상황에 따라 구현 클래스를 변경해야할 경우


---

# 4. 스프링 DB 접근 기술
애플리케이션 서버와 DB를 연결하는 방식으로 4가지가 있다.
JDBC, 스프링 JdbcTemplate, JPA, 스프링 데이터 JPA

1. JDBC
아주 오래전 기술이다.
gradle파일에 jdbc, 데이터베이스 관련 라이브러리를 추가하고, application.properties에 연결 설정을 추가하고, SQL쿼리, 리포지토리 등등을 일일이 다 만들어줘야하는 불편함이 있다.

2. 스프링 JdbcTemplate
실무에서 많이 사용한다. 위 JDBC와 동일하게 환경설정하면 된다.
스프링 JdbcTemplate과 Mybatis 같은 라이브러리는 JDBC의 반복 코드를 대부분 제거해준다.

```java
// ✅회원 Repository

public class JdbcTemplateMemberRepository implements MemberRepository {

    private final JdbcTemplate jdbcTemplate;
    
    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }

    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");

        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());

        Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query(
            "select * from member where id = ?", 
            memberRowMapper(),
            id
        );
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query(
            "select * from member", 
            memberRowMapper()
        );
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query(
            "select * from member where name = ?", 
            memberRowMapper(), 
            name
        );
        return result.stream().findAny();
    }

    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }
}

```


```java
// ✅스프링 JdbcTemplate 사용 전
@Configuration
public class SpringConfig { 

    private final DataSource dataSource;

    public SpringConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        // return new MemoryMemberRepository();
        // return new JdbcMemberRepository(dataSource);
        return new JdbcTemplateMemberRepository(dataSource);
    }
}

// ✅스프링 JdbcTemplate 사용 후
@Autowired
private JdbcTemplate jdbcTemplate;

public List<User> getAllUsers() {
    String sql = "SELECT * FROM users";
    
    return jdbcTemplate.query(sql, new RowMapper<User>() {
        @Override
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
            User user = new User();
            user.setId(rs.getInt("id"));
            user.setName(rs.getString("name"));
            user.setEmail(rs.getString("email"));
            return user;
        }
    });
}
```

3. JPA
JPA(Java Persistence API): 자바에서 객체와 관계형 DB를 연결하는 ORM 기술 표준
객체와 데이터베이스 테이블 사이의 매핑을 쉽게 해주고,
개발자가 **SQL 쿼리를 직접 작성하지 않아도 자바 객체로 데이터베이스 작업을 할 수 있게** 도와준다.

JPA 자체는 **인터페이스(약속)**만 제공하며, 실제 DB와 상호작용하는 기능을 JPA를 구현한 프레임워크나 라이브러리들이 담당한다.
그 예시로 Hibernate, OpenJPA 등이 있다. (표준은 JPA, 구현체는 Hibernate)

> 1. build.gradle에 라이브러리 추가
2. 스프링 부트에 JPA 설정 추가 (/application.properties)
3. JPA 엔티티 매핑
4. 스프링 설정 변경 (SpringConfig)


4. 스프링 데이터 JPA
스프링 데이터 JPA: JPA를 편리하게 도와주는 라이브러리
리포지토리에 구현 클래스 없이 인터페이스 만으로 개발할 수 있고, 기본 CRUD 기능도 제공한다.
복잡한 동적 쿼리는 `Querydsl`을 사용한다.

---

# 5. AOP
1. AOP(Aspect Oriented Programming): 관점 지향 프로그래밍으로,
어떤 로직을 기준으로 핵심 관점, 부가적인 관점으로 나누어 각각 모듈화 하는 것이다.
=> 공통 관심 사항과 핵심 관심 사항을 분리하고, 공통 작업을 처리하기 위해 AOP가 사용된다.

2. AOP 적용
> **공통 관심 사항**: 여러 부분에서 반복적으로 사용되는 부가적인 기능 (예: 로그, 보안, 트랜잭션)
**핵심 관심 사항**: 애플리케이션의 본래 기능을 수행하는 로직 (예: 돈 이체, 주문 처리)

- AOP는 **프록시 기반으로 동작**한다.
>가짜 프록시 객체 생성 -> 공통 작업 실행 -> 진짜 서비스 실행

![](https://velog.velcdn.com/images/mymy/post/c8a11797-a1b2-4870-ac1a-62aaf56b16cc/image.png)
![](https://velog.velcdn.com/images/mymy/post/9f4aa157-4c7b-435f-8bb4-e9f77aa39cb0/image.png)


어떤 객체에 접근할 때 그 객체를 직접 사용하지 않고, 대신 프록시가 **그 객체의 역할 대신 수행해주는 패턴**이다.
사용할때, 실제 Proxy가 주입되는지 콘솔에 출력해서 확인해야 한다.
프록시가 실제 메서드 호출되기 전에 먼저 실행될 수 있는 구조이다.
  - CGLIB 기술을 사용해 코드를 복제해서 코드를 조작한다.


