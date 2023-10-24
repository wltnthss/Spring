# JPA

* 기본적인 SQL도 JPA가 직접만들어서 실행해줍니다.
* SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환 할 수 있습니다.

## build.gradle, application.properties JPA 추가

```java
// build.gradle
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

// application.properties
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
```

* jpa는 객체를 보고 테이블을 만드는데 기존에 만들어져있는 테이블을 만드므로 auto 기능 none

## JPA 변환

```java
// domain패키지 Member

@Entity
public class Member {
    
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

* @Entity 어노테이션이 있으면 JPA가 관리하는 Entity 입니다.
* PK인 인스턴스 변수를 @Id 로 지정해주고 @GeneratedValue(strategy = GenerationType.IDENTITY) 는 DB가 알아서 생성해준다는 의미를 뜻합니다.
  
## JpaMemberRepository 생성

```java
public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member Save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}
```

* Jpa 를 스프링에게서 주입받으려면 EntityManager 를 사용합니다.
* em.createQuery("select m from Member m", Member.class).getResultList(); 객체를 대상으로 쿼리는 날리는 JPQL..
* JPA는 모든 변경이 Transactional 안에서 이루어져야하기 때문에 MemberService에도 @Transactional 어노테이션을 추가해줍니다.
* 수행하려면 SpringConfig 파일내에서도 JPAMemberRepository 를 추가해주어야합니다.

```java
@Configuration
public class SpringConfig {

    private EntityManager em;

    public SpringConfig(EntityManager em) {
        this.em = em;
    }

    @Bean
    public MemberRepository memberRepository(){
//        return new MemoryMemberRepository();
//        return new JdbcMemberRepository(dataSource);
//        return new JdbcTemplateMemberRepository(dataSource);
        return new JpaMemberRepository();
    }
}
```

# 스프링 데이터 JPA

* 리포지토리에 구현 클래스 없이 인터페이스만으로 개발이 가능하며, CRUD 기능도 스프링 데이터 JPA가 제공합니다.

## repository 패키지내에 SpringDataJpaMemberRepository 생성

```java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    @Override
    Optional<Member> findByName(String name);
}
```

* Interface 끼리는 extends 사용이 가능합니다.
* SpringDataJpa가 JpaRepository를 받고 있으면 자동으로 등록해줍니다.

## SpringConfig 파일 변경

```java
private final MemberRepository memberRepository;

    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```