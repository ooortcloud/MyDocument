이 페이지에서는 당장의 DB 개발을 위해 실무 훈련을 우선적으로 하면서 바로 기본적인 수준의 개발을 진행할 수 있는 상태로 하는 것을 목표로 한다. 이를 통해 이론적 내용도 간접적으로 학습해나갈 수 있을 것이다. 세부적인 이론은 나중에 스프링 DB 1편을 이론 중심으로 재학습하도록 하자.

# 목차

- [목차](#목차)
- [데이터 접근 기술 - 시작 세팅](#데이터-접근-기술---시작-세팅)
    - [스프링 컨테이너](#스프링-컨테이너)
    - [개발 팁](#개발-팁)
    - [Spring](#spring)
    - [JAVA](#java)
    - [참고 코드](#참고-코드)
- [데이터 접근 기술 - JPA](#데이터-접근-기술---jpa)
    - [영속성 컨텍스트](#영속성-컨텍스트)
  - [개발](#개발)
    - [개발 환경 세팅](#개발-환경-세팅)
    - [개발 코드 예제](#개발-코드-예제)
- [데이터 접근 기술 - 스프링 데이터 JPA](#데이터-접근-기술---스프링-데이터-jpa)
    - [JpaRepository 사용법](#jparepository-사용법)
    - [쿼리 메소드 기능 사용법](#쿼리-메소드-기능-사용법)
    - [JPQL 수동으로 정의하기](#jpql-수동으로-정의하기)
  - [개발 코드 예제](#개발-코드-예제-1)
- [데이터 접근 기술 - Querydsl](#데이터-접근-기술---querydsl)
  - [개발 환경 설정](#개발-환경-설정)
    - [Q 타입 생성](#q-타입-생성)
  - [개발 코드 예제](#개발-코드-예제-2)
- [데이터 접근 기술 - 활용 방안](#데이터-접근-기술---활용-방안)
  - [개발 코드 예제](#개발-코드-예제-3)
  - [다양한 데이터 접근 기술 조합](#다양한-데이터-접근-기술-조합)


이번 장에서는 JPA와 Hibernate, Spring Data JPA, QueryDSL에 대해 전부 간단하게 배워본다. 실무에서는 이들을 적절히 조합하여 DB 연결 단을 구축하며, 스프링 MVC 모델을 활용하여 프론트엔드를 구현한다.

# 데이터 접근 기술 - 시작 세팅

### 스프링 컨테이너

스프링 컨테이너는 자동으로 `빈`을 생성하고 관리하여 빈의 라이프사이클을 관리해주는 역할을 한다. 여기서 `빈` 이란 스프링에서 직접 생성하고 관리하는 자바 객체라고 생각하면 된다. 그래서 개발자는 일일이 각 로직에 대한 객체를 관리하는 코드를 작성할 필요없이, 스프링에서 알아서 객체를 생성하고 관리하고 뒷처리까지 해주기 때문에 편하게 개발할 수가 있는 것이다.

이때 빈 객체는 제각각 각자의 목표를 가지고 생성되는데, 대표적으로 컨트롤러, 서비스, 리포지토리가 그 예이다. 이미 스프링에서 서버를 위한 자체적인 객체 관리 설계도를 갖추고 있기 때문에, 개발자는 개발할 때 이 설계도에 알맞게 각 클래스를 적절한 위치에 끼워넣기만 하면 된다.

특히나 빈을 이용하면 의존성 주입을 활용하여 인터페이스 각각의 구현체를 손쉽게 갈아낄 수 있게 된다. (구성 클래스를 이용하면)

### 개발 팁

- 클래스나 변수 이름의 경우에는 어떤 정답이 있다기 보다는, 팀 내에서 일관성을 유지할 수 있는 것을 목표로 하면 된다.
- 객체의 주 목적이 “데이터를 전달하는 용도”라면 `DTO` 라고 명시하는 것이 좋다.
    - DTO의 경우 가장 직접적으로 사용하는 클래스나 인터페이스와 동일한 디렉토리에 저장하여 관리하는 것이 좋다.(즉, 최종 호출자와 동일한 디렉토리 안에 있도록 관리하면 됨.)
- 인터페이스는 주로 변경(업데이트) 가능성이 높은 클래스를 업데이트하기 쉽게 개발하기 위해서 작성한다. 하지만 서비스 로직을 변경하는 경우는 많지 않기 때문에, 일반적으로 서비스 인터페이스를 만들지는 않는다.
    - Spring에서는 `@Bean` 을 활용하여 서버를 띄울 때 스프링 컨테이너에 최종적으로 들어갈 구현체를 수동으로 정의할 수 있다. 이때 확실하게 인터페이스로 개발이 되어 있으면 손쉽게 해당 기능에 대한 구현체를 변경할 수 있다.
- 일반적으로 테스트를 할 때는 인터페이스를 대상으로 해야 테스트 코드를 작성하기 편리하다.
    - 물론 특정 기능을 테스트해야 하는 경우에는 해당 클래스를 테스트하는 것이 맞고.
- DB 설계를 할 때, 자연 키(현실 세계에서 실제 값)보다 대리 키(컴퓨터가 발급하는 값)를 PK로 지정하는 것이 좋다. 왜냐면 비즈니스는 변할 수 있기 때문이다.
- 버그가 터졌을 때는 최신 버전 말고 구 버전을 사용하도록 하자.

### Spring

- `@EventListener(ApplicationReadyEvent.class)` 는 스프링이 컨테이너 세팅 및 초기화를 전부 끝낸 후 서버를 띄운 바로 그 시점에 이 어노테이션이 달린 메소드를 가장 먼저 실행시키도록 하는 어노테이션이다. 컨테이너 세팅이 완벽히 끝난 상태에서 동작을 명령하므로 AOP 관련 문제가 전혀 발생하지 않아 시스템적으로 안전한 기능이다.
- `@Import(클래스명.class)` 는 스프링 컨테이너에 특정 “구성 클래스”를 수동으로 추가시키는데 사용하는 어노테이션이다. 메인 클래스에서 사용한다.
- `@SpringBootApplication(scanBasePackages = "디렉토리 경로")` 는 컴포넌트의 스캔 경로를 수동으로 지정하는데 사용하는 어노테이션이다. 기본적으로 메인 클래스에 이 어노테이션이 선언되어 있으며, 이에 따라 현재 프로젝트의 모든 클래스를 스프링 컨테이너에 등록하게 된다. 하지만 특정 디렉토리 내 클래스들만 스프링 컨테이너에 등록하고 나머지는 수동으로 스프링 컨테이너에 추가시키고 싶은 경우에는 scanBasePackages 경로를 수동으로 정의해줄 수 있다.
- `@Profile` 은 특정 빈 클래스를 특정 프로필에서만 스프링 컨테이너에 등록되도록 제한하는 어노테이션이다.
    
    만약 application.properties에서 아래처럼 프로필을 정의한다면,
    
    ```java
    spring.profiles.active=dev
    ```
    
    프로필 어노테이션을 사용하는 여러 클래스 중 `@Profile(”dev”)` 라고 정의된 것만 빌드 시에 활성화되도록 한다. 이를 통해 각 테스트 환경마다 특정 스프링 컨테이너 환경을 구성할 수 있도록 개발할 수 있다.
    
- `@AfterEach` 는 각 테스트가 끝날 때마다 해당 메소드가 동작하도록 하는 어노테이션이다. 주로 테스트가 끝날 때마다 DB에 저장된 데이터를 초기화하기 위해 사용된다.
- 

### JAVA

- 자바 리스트 내 객체 주소를 참조한다 → 값을 변경하면 즉각적으로 변경값이 적용된다. = 값을 바꾼 후 대입하는 과정이 불필요함.
- `Optional` 객체는 어떤 객체를 저장할 수 있는데, 그 값이 null이어도 된다.
- `instanceof` 키워드를 활용하면, 동일한 인터페이스에 서로 다른 구현체를 사용하는 환경인 경우 특정 상황에 대해서 특정 구현체에만 있는 기능을 사용하고자 하는 경우에 유연하게 코드가 실행되도록 개발할 수 있다.
    
    ```java
    @AfterEach
    void afterEach() {
            //MemoryItemRepository 의 경우 제한적으로 사용
            if (itemRepository instanceof MemoryItemRepository) {
                ((MemoryItemRepository) itemRepository).clearStore();
            }
        }
    ```
    
    이렇게 특정 구현체에 대해서만 동작하도록 분리하여 개발하면, 실제 빌드할 서버에는 위험한 기능들을 제거해둔 상태로 개발하되 테스트 환경에서는 위험한 기능들을 전부 사용할 수 있도록 개발할 수 있다.
    
- `...` 문법을 활용하면 특정 메소드의 파라미터 개수를 제한시키지 않을 수 있다.
- 

### 참고 코드

- 메소드에서 filter를 활용하여 특정 조건을 만족시키는 값들만 리스트에 담아서 리스트 객체를 리턴할 수 있는 코드.

```java
return store.values().stream()
                .filter(item -> {
                    if (ObjectUtils.isEmpty(itemName)) {
                        return true;
                    }
                    return item.getItemName().contains(itemName);
                }).filter(item -> {
                    if (maxPrice == null) {
                        return true;
                    }
                    return item.getPrice() <= maxPrice;
                })
                .collect(Collectors.toList());
```

# 데이터 접근 기술 - JPA

SQL 중심적인 개발의 문제점 : **개발은 객체 지향적으로 하는데, DB는 SQL밖에 모름(패러다임의 불일치)** → 개발자는 맨날 객체를 SQL로 변환시켜 DB에 저장시키는 동일한 작업을 반복해야 한다. 이렇게 되면 개발도 힘들고 유지보수하기도 까다롭다. 특히나 외래키를 사용하고 Join을 사용하게 되면 릴레이션 관계가 복잡해지면서 미치는 거지.

또한 객체 그래프 탐색 시 엔티티 신뢰 불가 문제가 발생해버린다. (중략)

그러면 자바에서 객체를 자바 컬렉션에 저장하듯이 동일하게 객체를 DB에 저장할 방법이 없을까? → ORM(Object-Relational Mapping) 기술의 등장. ORM은 객체와 RDB 사이를 매핑해주는 프레임워크임. 이것의 가장 큰 장점은 패러다임 불일치 문제를 해결해준다는 것이다. 

JPA는 JAVA 영역에서의 ORM 표준 인터페이스이며, 대표적인 JPA의 구현체는 Hibernate이다. JPA를 사용하면 엔티티를 신뢰할 수 있게 된다. (중략) 순수히 개발 관점에서 생각하면 JPA를 적용하면 DB와 CRUD할 때 JAVA 컬렉션과 동일한 패러다임으로 CRUD할 수 있다는 것이다. 성능 최적화는 덤.

- 지연로딩 : 객체가 실제 사용될 때마다 로딩
- 즉시로딩 : SQL에서 JOIN 기능을 사용하여 연관된 객체까지 한번에 로딩해버림. → 캐싱

하지만 ORM 기술을 잘 쓰려면 이전처럼 객체와 RDB 전부 이해해야 한다. 그냥 공부할 것은 그대로인데 개발 환경은 더 좋아졌다~ 정도로 이해하면 됨.

### 영속성 컨텍스트

`영속성 컨텍스트` 는 JPA에서 엔티티를 영구 저장소에 저장하고, 엔티티의 상태를 추적하며, 엔티티 간의 관계를 관리 논리적 개념이다. 영속성 컨텍스트는 트랜잭션이 시작되면 생성되고, 트랜잭션이 종료되면 영속성 컨텍스트가 종료된다. (중략)

## 개발

### 개발 환경 세팅

- **build.gradle**

```java
plugins {
	id 'org.springframework.boot' version '2.6.5'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
  // JPA와 Spring Data JPA 모두 다운로드 가능.
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'

	//H2 데이터베이스 추가
	runtimeOnly 'com.h2database:h2'

	//테스트에서 lombok 사용
	testCompileOnly 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

위 환경처럼 세팅해주면 JPA 뿐만 아니라 다른 웹이나 DB 관련 드라이버 문제가 발생하진 않을 것이다.

- **application.properties**

```java
# DB url setting
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
spring.datasource.password=1234

#JPA log
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

위는 DB url 세팅인데, 이거 안 해주면 DB 테스트 자체가 불가능하다. 일단 저거 먼저 해준다.

그 다음은 JPA 로그 설정이다. 

- 첫번째 줄은 hibernate가 자동으로 생성하고 실행하는 SQL을 로그창에서 확인할 수 있게 해준다.
- 두번째 줄은 그 SQL에 바인딩되는 파라미터들을 확인할 수 있게 해준다.

### 개발 코드 예제

- **객체에 JPA 적용**

```java
@Data
@Entity  // 현재 객체에 JPA를 적용
@Table(name = "item")  // 실제 테이블과 이름 같으면 생략 가능.
public class Item {

    @Id  // PK로 지정
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // AUTO_INCREMENT 전략 사용
    private Long id;

    @Column(name = "item_name")  // 실제 속성과 이름이 같으면 생략 가능.
    private String itemName;
    private Integer price;
    private Integer quantity;

    // JPA 사용 시 객체에 기본 생성자는 필수로 있어야 한다.
    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

- **JpaItemRepository**

```java
@Slf4j
@Repository
@Transactional  // JPA는 트랜잭션 기능을 기반으로 한다. 일반적으로는 데이터 변경이 이뤄지는 서비스 계층에 트랜잭션을 걸어준다. 지금은 메모리 상에서 변경하므로 레포지토리에 트랜잭션을 걸었음.
public class JpaItemRepository implements ItemRepository {

    // 얘가 JPA의 본체이다. 이렇게 선언해두면 스프링에서 자동으로 의존성 주입 해준다.
    private final EntityManager em;

    public JpaItemRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Item save(Item item) {
        em.persist(item);  // JPA가 해당 엔티티를 알아서 DB에 저장해줌.(사용 전 JPA에 해당 객체를 등록해줘야 함.)
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = em.find(Item.class, itemId);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }  // 트랜잭션이 commit되는 시점에 변경된 엔티티들이 있으면, JPA가 알아서 UPDATE 쿼리를 만들어서 DB에 전송함. 참고로 트랜잭션은 메소드가 끝날 때 commit됨.

    @Override
    public Optional<Item> findById(Long id) {
        Item item = em.find(Item.class, id);  // 찾을 객체 타입과 PK를 파라미터로 받는다. JPA가 PK 기준으로 알아서 찾아줌.
        return Optional.ofNullable(item);
    }

    // JPA는 동적 쿼리문 작성이 문제임. JPQL을 조합해서 동적 쿼리문을 작성해야 하는데, 코드 작성하기 너무 번거로움.
    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String jpql = "select i from Item i";
        Integer maxPrice = cond.getMaxPrice();
        String itemName = cond.getItemName();
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            jpql += " where";
        }
        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            jpql += " i.itemName like concat('%',:itemName,'%')";
            andFlag = true;
        }
        if (maxPrice != null) { if (andFlag) {
            jpql += " and";
        }
            jpql += " i.price <= :maxPrice";
        }
        log.info("jpql={}", jpql);
        TypedQuery<Item> query = em.createQuery(jpql, Item.class);
        if (StringUtils.hasText(itemName)) {
            ((TypedQuery<?>) query).setParameter("itemName", itemName);
        }
        if (maxPrice != null) {
            query.setParameter("maxPrice", maxPrice);
        }
        return query.getResultList();
    }
}
```

JPQL 문법이 어색할 것이다. JPQL에 대해 간단히 설명하면 아래와 같다.

- 테이블을 대상으로 쿼리를 작성하는 SQL과 달리, 엔티티를 대상으로 쿼리를 작성한다.
    
    ```java
    SELECT member
    FROM Member member
    ```
    
    위 코드는 JPQL로 “Member” 엔티티의 모든 엔티티를 조회하는 코드이다.
    
- JPQL의 동적 쿼리문은 `?` 또는 `:placeholder`를 사용하여 쿼리의 일부를 동적으로 지정합니다.
    
    ```java
    SELECT member
    FROM Member member
    WHERE member.age > :age
    ```
    
    위 코드는 “Member” 엔티티의 나이가 “:age” 보다 큰 엔티티를 조회하는 코드이다. 여기서 “:age”는 실행 시점마다 그 값이 변하는 변수로 취급된다.(타임리프의 동적 변수와 비슷하게 작용함.)
    
- **JpaConfig**

```java
@Configuration
public class JpaConfig {
    private final EntityManager em;
    
    public JpaConfig(EntityManager em) {
        this.em = em;
    }
    
    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }
    
    @Bean
    public ItemRepository itemRepository() {
        return new JpaItemRepository(em);
    }
}
```

EntityManager 객체를 구성 클래스에서 생성한 후, 이 객체가 필요한 구현체에게 의존성을 주입한다. 이렇게 해야 `단일 책임 원칙` 을 지킬 수 있기 때문이다.

- 단일 책임 원칙이란, 하나의 클래스는 단 하나의 책임만 가져야 하는 것을 말한다. 예컨데, 체스 말을 이동시키는 동작과 체스 말을 잡는 동작은 서로 다른 동작이다. 이 두 기능을 서로 다른 클래스로 각각 구현했을 때 단일 책임 원칙을 지켰다고 할 수 있다. 이렇게 되면 체스 말일 잡는 동작에 대한 수정이 필요할 때, 체스 말에 대한 짬뽕 클래스가 아닌 체스 말의 잡는 동작에 대한 클래스만 수정하면 되기 때문에 유지보수가 매우 간단해진다는 장점이 있다. 하지만 그만큼 클래스의 개수가 늘어나기 때문에 구조 자체가 난잡해지는 문제점도 발생한다. 그러므로 단일 책임 원칙의 적용 범위를 어느 정도로 잡을지 잘 타협해서 개발해야 한다.
- 그럼 구성 클래스의 책임은 무엇인가? 바로 “객체를 생성하는 역할”이다. 구성 클래스에서 객체를 생성하고 이 객체를 필요로 하는 다른 클래스들에게 주입을 해주면, 모두가 동일한 객체를 공유하기 때문에 레지스터에서 서로 다른 값을 가져 프로그램 내부에서 동기화 에러가 발생하는 것을 막을 수 있다. 이렇게 어떤 책임에 대한 의존성을 단일화시키면 여러 가지 복합적인 문제를 피해갈 수 있기 때문에 원활한 개발이 가능해진다는 장점이 있다.
- **MainApplication**

```java
@Import(JpaConfig.class)  // 스프링 컨테이너에 적용할 구성 클래스를 직접 지정한다.
...
```

# 데이터 접근 기술 - 스프링 데이터 JPA

기존 JPA 개발의 자동화를 지원하는 툴. 인터페이스만 생성해놓고, 디폴트 도메인의 이름만 잘 정해주면 Spring 내부에서 알아서 구현체와 쿼리문을 뚝딱 만들어준다. 그래서 코드 작성 시간을 크게 줄일 수 있음. 그래서 실무에서 반드시 사용하는 기술이라고 한다. (하지만 그래도 기존 기술들의 이론에 대한 충분한 이해가 필요.) “@Query” 어노테이션을 통해 수동으로 JPQL 작성도 가능.

인터페이스를 통해 기본적인 CRUD는 전부 자동화가 되어 있으며, 그 외에도 공통화가 가능한 대부분의 기능들이 다 자동화가 되어 있다. 

### JpaRepository 사용법

```java
public interface ItemRepository extends JpaRepository<Item, Long> { ... }
```

- 우선 JpaRepository를 상속받을 인터페이스를 생성한다.
- 이때 <> 안에는 순서대로 <엔티티 타입, 엔티티Id(아마도 PK)의 타입>을 넣어주면 된다.

### 쿼리 메소드 기능 사용법

스프링 데이터 JPA는 메소드의 이름을 분석해서 JPQL을 자동으로 생성하고 실행한다. 단, 이 기능을 이용하기 위해서는 메소드 이름을 작성할 때 몇 가지 규칙을 지켜야 한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
   List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

- **헤더**
    - 조회: find…By , read…By , query…By , get…By
    예:) findHelloBy 처럼 ...에 식별하기 위한 내용(설명)이 들어가도 된다.
    - COUNT: count…By 반환타입 long
    - EXISTS: exists…By 반환타입 boolean
    - 삭제: delete…By , remove…By 반환타입 long
    - DISTINCT: findDistinct , findMemberDistinctBy
    - LIMIT: findFirst3 , findFirst , findTop , findTop3 (이건 거의 안 씀. 페이징으로 대체함.)
- **바디**
    
    일반적으로 By 뒤에 조건절이 붙는다고 생각하면 된다.
    

### JPQL 수동으로 정의하기

```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long>
{
   //쿼리 메서드 기능
   List<Item> findByItemNameLike(String itemName);

   //쿼리 직접 실행
   @Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
   List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
}
```

수동 JPQL을 적용할 메소드에 `@Query` 어노테이션을 달아서 직접 정의해주면 된다. 메소드 이름이 너무 괴랄해질 거 같을 때 사용한다. → 보통 동적 쿼리문을 작성해야 하는 경우 이렇게 된다. 그래서 스프링 데이터 JPA도 동적 쿼리문 문제를 해결해주지는 못한다.

## 개발 코드 예제

```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long> {

    // 간단한 CRUD는 디폴트로 SpringDataJPA에서 제공하므로 별도로 선언할 필요 없움.
    // List<Item> findAll

    List<Item> findByItemNameLike(String itemName);

    List<Item> findByPriceLessThanEqual(Integer price);

    // 복잡한 쿼리문을 Spring Data JPA를 이용해 만드는 것은 너무 번거로움.
    // List<Item> findByItemNameLikeAndPriceLessThanEqual(String itemName, Integer price);

    // 직접 JPQL 쿼리문을 작성하여 동적 쿼리문을 구현. 이때 "@Param" 어노테이션을 통해 파라미터를 JPQL과 매핑해줘야 함.
    @Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
    List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);

}
```

- **SpringDataJpaItemRepository**

```java
@Repository
@Transactional
@RequiredArgsConstructor
public class JpaItemRepositoryV2 implements ItemRepository {

    private final SpringDataJpaItemRepository repository;

    @Override
    public Item save(Item item){
        // Spring Data JPA에서 기본으로 제공.
        return repository.save(item);
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = repository.findById(itemId).orElseThrow();
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }  // 트랜잭션이 발생되면서 JPA가 DB에 쿼리를 보냄.

    @Override
    public Optional<Item> findById(Long id) {
        // Spring Data JPA에서 기본으로 제공.
        return repository.findById(id);
    }

    // Spring Data JPA도 복잡한 동적 쿼리문은 깔끔하게 처리하지 못함. 현업에서는 이렇게 분기문으로 처리하지 않는다. -> 나중에 QueryDSL로 깔끔하게 처리하는 방법을 배울 것임.
    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        if (StringUtils.hasText(itemName) && maxPrice != null) {
            // return repository.findByItemNameLikeAndPriceLessThanEqual("%" + itemName + "%", maxPrice);
            return repository.findItems("%" + itemName + "%", maxPrice);
        } else if (StringUtils.hasText(itemName)) {
            return repository.findByItemNameLike("%" + itemName + "%");
        } else if (maxPrice != null) {
            return repository.findByPriceLessThanEqual(maxPrice);
        } else {
            return repository.findAll();
        }
    }
}
```

의존성 주입의 장점을 활용하기 위해 ItemRepository 인터페이스의 구현체를 따로 만들었다. 나중에 새로운 구성 클래스에서 주입되는 구현체의 이름만 변경하고 메인 클래스에서 해당 구성 클래스를 반영시키면, 해당 구성 내용이 코드 전체에 변경점이 적용되기 때문이다.

- **SpringDataJpaConfig**

```java
@Configuration
@RequiredArgsConstructor
public class SpringDataJpaConfig {

    // 이 인터페이스는 Spring Data JPA를 상속하기 때문에, 스프링 빈에도 등록되며 프록시 기술도 적용된다.
    private final SpringDataJpaItemRepository repository;

    @Bean
    public ItemService itemService(){
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository(){
        return new JpaItemRepositoryV2(repository);
    }
}
```

구성 클래스에 대한 얘기는 위에서 다시 확인하고 오기 바란다. 매우 중요한 개념이니까.

- **메인 클래스**

```java
@Import(SpringDataJpaConfig.class)
...
```

그리고 메인 클래스에서 구성 클래스를 앞에서 만든 거로 변경해주면, 감쪽같이 JPA에서 Spring Data JPA로 탈바꿈할 수 있다!

아, 참고로 Spring Data JPA는 구현체를 자동 생성하면서 예외처리도 자동으로 싹 해준다. 그 점은 정말 메리트 있는 거 같다.

# 데이터 접근 기술 - Querydsl

QueryDSL은 쿼리를 자바 코드 작성하듯이 도와주는 프레임워크이다. 이 말은, 내가 쿼리문을 작성하고 있을 때 타입 에러나 오타가 있다면 컴파일 단계에서 다 잡아줘서 배포 시 문제를 크게 줄여줄 수 있다는 말이다. 또한 코드 어시스턴스 기능이 있어 쿼리를 작성할 때도 자바 코드 짜듯이 매우 편리하게 작성할 수 있다. 주로 JPA 쿼리인 JPQL에 사용한다. 

쉽게 말해, Type-Safe하게 개발할 수 있도록 도와준다. 

## 개발 환경 설정

build.gradle에서 디펜던시에 아래 코드를 추가한다.

```java
//Querydsl 추가
	implementation 'com.querydsl:querydsl-jpa'
	annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa"
	annotationProcessor "jakarta.annotation:jakarta.annotation-api"
	annotationProcessor "jakarta.persistence:jakarta.persistence-api"
```

그리고 build.gradle의 맨 끝에 줄에 아래 코드를 추가한다.

```java
//Querydsl 추가, 자동 생성된 Q클래스 gradle clean으로 제거
clean {
	delete file('src/main/generated')
}
```

### Q 타입 생성

환경설정에서 ‘Build and run’ 을 모두 Intelij로 변경한 후에 빌드를 하거나 run을 하면 자동으로 생성된다. src/main/generated 경로에 Q 타입 클래스가 생성되어 있어야 한다.

gradle로도 Q 타입을 생성하는 방법이 있지만, 이게 제일 편하다. (필요하면 구글링)

## 개발 코드 예제

- JpaItemRepositoryV3

```java
import static hello.itemservice.domain.QItem.*;

@Repository
@Transactional
public class JpaItemRepositoryV3 implements ItemRepository {

    private final EntityManager em;
    private final JPAQueryFactory query;

    public JpaItemRepositoryV3(EntityManager em) {
        this.em = em;
        this.query = new JPAQueryFactory(em);  // 구성 클래스에서 주입받는 형태로 구현해도 됨.
    }

    // 동일
    @Override
    public Item save(Item item) {
        em.persist(item);
        return item;
    }

    // 동일
    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = findById(itemId).orElseThrow();
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    // 동일
    @Override
    public Optional<Item> findById(Long id) {
        Item item = em.find(Item.class, id);
        return Optional.ofNullable(item);
    }

    // QueryDSL을 적용하여 코드 작성 V1
    public List<Item> findAllOld(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        QItem item = QItem.item;  // 자동으로 생성된 Qitem에서 item 객체를 불러옴. Static import 하는 것을 추천.

        BooleanBuilder builder = new BooleanBuilder();
        if(StringUtils.hasText(itemName)) {
            builder.and(item.itemName.like("%" + itemName + "%"));  // 동적 쿼리문 작성
        }
        if (maxPrice != null) {
            builder.and(item.price.loe(maxPrice));  // 동적 쿼리문 작성
        }

        List<Item> result = query
                .select(item)
                .from(item)
                .where(builder)
                .fetch();

        return result;
    }

    // QueryDSL을 적용하여 코드 작성 V2
    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        List<Item> result = query
                .select(item)
                .from(item)
                .where(likeItemName(itemName), maxPrice(maxPrice))
                .fetch();

        return result;
    }

    private BooleanExpression likeItemName(String itemName) {
        if (StringUtils.hasText(itemName)) {
            return item.itemName.like("%" + itemName + "%");
        }
        return null;
    }

    private BooleanExpression maxPrice(Integer maxPrice) {
        if (maxPrice != null) {
            return item.price.loe(maxPrice);
        }
        return null;
    }
}
```

이번에는 QueryDSL을 적용한 구현체를 만들어보았다. 윗 부분은 JPA를 그냥 사용하면 되니까 다를게 없다. 중요한 것은 밑에 복잡한 동적 쿼리문을 QueryDSL로 처리하는 방법을 알아보는 거지.

V1의 경우에는 무난한 형태의 복잡한 동적 쿼리 작성법이다. builder 객체를 활용하여 JPQL 문자열을 조건에 따라 쭉쭉 붙여넣는 그림.

V2는 각 조건들을 메소드화하여 표현한 방식이다. 이 방식으로 개발하면 디테일하게 유지보수하기가 좀 더 수월해진다.

- QuerydslConfig

```java
@Configuration
@RequiredArgsConstructor
public class QuerydslConfig {

    private final EntityManager em;  // JPA와 동일하게 EntityManager를 받아옴.

    @Bean
    public ItemService itemService(){
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository(){
        return new JpaItemRepositoryV3(em);
    }
}
```

역시나 구성 클래스를 생성해준다.

- 메인 클래스

```java
@Import(QuerydslConfig.class)
...
```

# 데이터 접근 기술 - 활용 방안

개발자는 다음 두 가지 중 하나를 고를 수 밖에 없는 딜레마에 빠지게 된다.

1. 향후 서비스 운영에 있어 유지보수에 용이하게 만들기 위해 추상화에 비용을 들인다.
2. 당장 빠르게 개발하기 위해 추상화를 하지 않는다.

추상화를 하지 않으면 나중에 리팩토링을 하는데 기술부채 비용이 더 많이 들 수 있다. 반면에 추상화가 불필요한 서비스였다면, 그건 그거대로 비용을 낭비한 셈이라고 볼 수 있다.

핵심은 언제 ‘추상화’를 할 것인가라고 볼 수 있다. 그것에 대해 간단히 답하자면 아래처럼 말할 수 있겠다.

- **서비스의 규모가 커질 예정이거나, 잦은 변경이 예상되는 경우 = 추상화 필수**
- **서비스 규모가 작거나, 변경될 가능성이 매우 낮은 경우 = 추상화 불필요**

세상에 나쁜 솔루션은 없다. 다 쓸모가 있는 것들이다. 그러나 솔루션을 적재적소에 맞게 적용하는 능력은 반드시 필요하다.

## 개발 코드 예제

어떤 쿼리를 사용하든 개발하기 쉽게 코드를 작성하려면, SpringDataJPA와 QueryDSL을 모두 사용하기 위해 각각의 Repository를 구현해야 한다. SpringDataJPA 같은 경우에는 지가 알아서 구현체를 만들어주기 때문에 인터페이스만 만들어서 의존성 주입을 하면 된다. (의존성 주입은 Spring 내에서 알아서 처리해줌.)

- **ItemRepositoryV2**

```java
public interface ItemRepositoryV2 extends JpaRepository<Item, Long> {
}
```

이 인터페이스는 Spring Data JPA를 사용하기 위해 생성했다. 부모 인터페이스인 JpaRepository 안에 기본적인 CRUD 코드들이 전부 마련되어 있고 Spring Data JPA가 알아서 해당 코드들을 구현까지 해주기 때문에, 간단한 CRUD 작업을 해야 하는 경우에는 요 인터페이스를 호출해서 작업하면 된다.

- **ItemQueryRepositoryV2**

```java
@Repository
public class ItemQueryRepositoryV2 {

    private final JPAQueryFactory query;

    public ItemQueryRepositoryV2(EntityManager em) {
        this.query = new JPAQueryFactory(em);  // Spring Data JPA도 EntityManager 객체를 의존성 주입받아 쓰기 때문에, QueryDSL 사용 시에 해당 객체를 같이 주입받아 사용하는 형태로 개발했음.
    }

    public List<Item> findAll (ItemSearchCond cond) {
        return query.select(item)
                .from(item)
                .where(
                        likeItemName(cond.getItemName()),
                        maxPrice(cond.getMaxPrice())
                )
                .fetch();
    }

    private BooleanExpression likeItemName(String itemName) {
        if (StringUtils.hasText(itemName)) {
            return item.itemName.like("%" + itemName + "%");
        }
        return null;
    }

    private BooleanExpression maxPrice(Integer maxPrice) {
        if (maxPrice != null) {
            return item.price.loe(maxPrice);
        }
        return null;
    }
}
```

이 레포지토리는 QueryDSL을 사용하기 위해 구현하였다. 현재는 이름 필터링 조회, 금액 상한선 조회 두 기능에 대한 동적 쿼리문 기능이 QueryDSL을 사용하여 구현되어 있다. 필요하다면, 여기서 다른 동적 쿼리문을 QueryDSL을 활용해 작성해주면 된다.

- **ItemServiceV2**

```java
@Service
@RequiredArgsConstructor
@Transactional
public class ItemServiceV2 implements ItemService{

    private final ItemRepositoryV2 itemRepositoryV2;
    private final ItemQueryRepositoryV2 itemQueryRepositoryV2;

    @Override
    public Item save(Item item) {
        return itemRepositoryV2.save(item);
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = itemRepositoryV2.findById(itemId).orElseThrow();
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    @Override
    public Optional<Item> findById(Long id) {  // 리턴 타입에서 < > 안의 타입을 갖는 엔티티를 기준으로 알아서 찾는다.
        return itemRepositoryV2.findById(id);
    }

    @Override
    public List<Item> findItems(ItemSearchCond cond) {
        return itemQueryRepositoryV2.findAll(cond);
    }
}
```

DB 쿼리 생성 코드가 매우 간단해진 모습! 이게 자바 코드인지 SQL인지 분간이 안 될 정도로 깔끔하게 작성되었다.

- **V2Config**

```java
@Configuration
@RequiredArgsConstructor
public class V2Config {
    private final EntityManager em;  // JPA에서 자동으로 의존성 주입 해준다.
    private final ItemRepositoryV2 itemRepositoryV2;  // Spring Data JPA에서 자동으로 의존성 주입 해준다.

    @Bean
    public ItemService itemService() {
        return new ItemServiceV2(itemRepositoryV2, itemQueryRepository());
    }

    @Bean
    public ItemQueryRepositoryV2 itemQueryRepository() {
        return new ItemQueryRepositoryV2(em);
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JpaItemRepositoryV3(em);
    }
}
```

각각의 구현체들에 대해서 구성 클래스에서 세팅해주고~

- 메인 클래스

```java
@Import(V2Config.class)
...
```

메인 클래스에서 구성 클래스만 변경하면 끝.

## 다양한 데이터 접근 기술 조합

누누히 얘기하지만, 개발에 정답은 없다. 하지만 개발 팁 같은 건 세울 수 있겠지.

데이터 접근 기술은 크게 두 종류로 나뉜다.

- SQLMapper : MyBatis, JdbcTamplate
- ORM : JPA, Spring Data JPA, QueryDSL

보통 실무에서는 95% ORM을 적용하고, 나머지 5% 정도만 SQLMapper를 사용하여 네이티브 SQL을 작성하는 편이다. 실무에서는 개발 생산성이 1순위이기도 하고 거의 대체로는 복잡한 쿼리 코드를 작성할 일이 없기 때문에, ORM을 사용하여 대부분의 개발을 진행할 수 있다. 하지만 복잡한 쿼리 코드가 요구되는 일부 기능에는 아무리 QueryDSL이라도 ORM으로는 한계가 있기 때문에 SQLMapper를 사용하여 직접 네이티브 SQL을 작성해서 개발을 마무리짓는 편이다.

하지만 만약 현재 개발 중인 프로젝트가 복잡한 통계 쿼리를 자주 사용해야 한다면 SQLMapper를 위주로 개발을 진행하는 것이 좋다. 그런 쿼리들은 ORM 기술로는 감당하기가 어렵기 때문이다.

그리고 현재 프로젝트 팀원들의 수준들도 고려해야 한다. SQLMapper는 SQL만 알면 누구나 사용할 수 있지만, ORM 기술의 경우 학습 허들이 높기 때문에 팀원들의 개발 능력이 낮은 경우 어쩔 수 없이 SQLMapper 기반으로 코드를 작성해야 할 수도 있다.