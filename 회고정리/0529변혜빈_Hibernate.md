---
date: 2024-05-28
author: hyebeenbyeon
---

> 💻 참고한 사이트
[Hibernate 6 Documentation](https://docs.jboss.org/hibernate/orm/6.5/introduction/html_single/Hibernate_Introduction.html#entity-clases)

## 개요
_ORM_ (Object Relational Mapping)
- 객체 지향 언어에서 관계형 데이터베이스 작업을 단순화하기 위한 추상화
- 객체가 테이블이 되도록 매핑시켜 주는 것
- SQL을 직접 사용하지 않고 데이터를 CRUD 할 수 있는 클래스 및 매소드 

_JPA_ (Java Persistence API)
- 자바 ORM 기술에 대한 표준 명세로, JAVA에서 제공하는 API이다.
- 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스다.

_Hibernate_ 
- JPA([Java Persistence API](https://github.com/jakartaee/persistence/tree/master/api/src/main/java/jakarta/persistence))의 구현체의 한 종류이다.

_Spring Data JPA_ 
- 이러한 구현체 위에 추상화 계층을 제공하여 개발자들이 더 편히라게 데이터 접근을 할 수 있도록 돕는 프레임워크이다.

## Spring Data JPA
### Spring Data JPA의 역할
> written by gpt-4o

Spring Data JPA는 Spring Framework에서 JPA(Java Persistence API)를 보다 쉽게 사용할 수 있도록 도와주는 모듈로, 주로 다음과 같은 역할을 합니다:

1. **리포지토리 추상화 제공**: Spring Data JPA는 데이터 액세스 레이어를 추상화하여, 개발자가 복잡한 JDBC 코드를 작성하지 않고도 데이터베이스와 상호 작용할 수 있도록 합니다. `JpaRepository`, `CrudRepository` 등의 인터페이스를 통해 기본적인 CRUD (Create, Read, Update, Delete) 작업을 쉽게 구현할 수 있습니다.

   ```java
   public interface UserRepository extends JpaRepository<User, Long> {
       // 기본적인 CRUD 메서드는 JpaRepository가 제공
   }
   ```

2. **쿼리 메서드 자동 생성**: 인터페이스에 메서드를 정의하기만 하면, 메서드 이름을 기반으로 JPA 쿼리를 자동으로 생성해줍니다. 예를 들어, `findByUsername(String username)`이라는 메서드를 선언하면, Spring Data JPA는 자동으로 `username` 필드로 검색하는 SQL 쿼리를 생성합니다.

   ```java
   public interface UserRepository extends JpaRepository<User, Long> {
       List<User> findByLastName(String lastName);
   }
   ```

3. **커스텀 쿼리 지원**: 복잡한 쿼리가 필요한 경우, JPQL(Java Persistence Query Language) 또는 네이티브 SQL을 사용하여 커스텀 쿼리를 작성할 수 있습니다.

   ```java
   @Query("SELECT u FROM User u WHERE u.email = ?1")
   User findByEmailAddress(String emailAddress);
   ```

4. **페이징 및 정렬 지원**: 대용량 데이터를 다룰 때 유용한 페이징 및 정렬 기능을 쉽게 구현할 수 있도록 지원합니다. `Pageable` 인터페이스를 사용하여 페이징된 결과를 반환할 수 있습니다.

   ```java
   Page<User> findAll(Pageable pageable);
   ```

5. **트랜잭션 관리**: Spring Data JPA는 Spring의 트랜잭션 관리 기능과 통합되어, 애플리케이션의 데이터베이스 트랜잭션을 쉽게 관리할 수 있습니다. `@Transactional` 어노테이션을 사용하여 메서드 또는 클래스 수준에서 트랜잭션을 제어할 수 있습니다.

   ```java
   @Transactional
   public void someTransactionalMethod() {
       // 데이터베이스 트랜잭션이 필요할 경우 사용
   }
   ```

6. **자동 리포지토리 구현**: 기본적인 CRUD 메서드 외에도, Spring Data JPA는 필요한 경우 리포지토리 인터페이스의 구현을 자동으로 생성해줍니다. 개발자는 인터페이스만 정의하면 되며, 나머지 구현은 Spring Data JPA가 처리합니다.

Spring Data JPA의 주요 역할은 이러한 기능들을 통해 데이터 액세스 계층을 단순화하고, 반복적인 코드 작성을 줄이며, 유지보수성을 높이는 것입니다. 이를 통해 개발자는 비즈니스 로직에 더 집중할 수 있게 됩니다.


## Hibernate
### JPA 구현체의 역할
> written by gpt-3.5

Hibernate와 같은 JPA 구현체는 다음과 같은 주요 역할을 수행합니다:

1. **JPA 구현체 제공**: JPA(Java Persistence API)를 실제로 구현하는 라이브러리로, JPA 명세에 정의된 기능들을 구현하여 제공합니다. 이를 통해 개발자는 데이터베이스와 상호 작용하기 위해 표준화된 방법을 사용할 수 있습니다.

2. **ORM (Object-Relational Mapping) 기능 제공**: 객체 지향 프로그래밍 언어인 자바와 관계형 데이터베이스 간의 매핑을 지원합니다. 즉, 객체와 테이블 간의 매핑을 자동으로 처리하여, 객체를 데이터베이스에 저장하고 조회할 수 있도록 합니다.

3. **SQL 생성 및 실행**: 객체 지향 쿼리를 사용하여 개발자가 SQL을 직접 작성하는 번거로움을 줄여줍니다. Hibernate는 개발자가 JPQL(Java Persistence Query Language) 또는 Criteria API를 사용하여 객체를 쿼리하고, 이를 SQL로 변환하여 실행합니다.

4. **트랜잭션 관리**: 데이터베이스 트랜잭션을 관리하고, ACID(원자성, 일관성, 고립성, 지속성) 속성을 보장합니다. 이를 통해 데이터베이스에서 데이터의 일관성과 무결성을 유지할 수 있습니다.

5. **캐시 지원**: 성능 향상을 위해 쿼리와 엔티티를 캐시하여 반복적인 데이터베이스 액세스를 줄입니다. Hibernate는 다양한 종류의 캐시를 제공하며, 개발자는 설정을 통해 적절한 캐시 전략을 선택할 수 있습니다.

6. **지연 로딩 및 즉시 로딩**: 객체 간의 관계를 관리하고, 지연 로딩 또는 즉시 로딩을 통해 필요한 경우에만 연관된 객체를 로드합니다. 이를 통해 성능을 최적화하고, 불필요한 데이터베이스 액세스를 방지할 수 있습니다.

7. **유지보수 및 확장성**: Hibernate는 다양한 설정 옵션과 확장 포인트를 제공하여, 개발자가 애플리케이션의 요구 사항에 맞게 커스터마이징할 수 있습니다. 또한, 자동으로 SQL을 생성하므로 개발자는 데이터베이스에 대한 세부 사항을 신경 쓸 필요가 없습니다.

이러한 기능들을 통해 Hibernate와 같은 JPA 구현체는 개발자가 데이터 액세스 계층을 보다 쉽고 효율적으로 구현할 수 있도록 도와줍니다.

### We can think the API of Hibernate in terms of three basic elements
1. an implementation of the JPA-defined APIs, most importantly, of the interfaces `EntityManagerFactory` and `EntityManager`, and of the JPA-defined O/R mapping annotations,
2. a _native API_ exposing the full set of available functionality, centered around the interfaces [`SessionFactory`](https://docs.jboss.org/hibernate/orm/6.5/javadocs/org/hibernate/SessionFactory.html), which extends `EntityManagerFactory`, and [`Session`](https://docs.jboss.org/hibernate/orm/6.5/javadocs/org/hibernate/Session.html), which extends `EntityManager`, and
	()
3. a set of [_mapping annotations_](https://docs.jboss.org/hibernate/orm/6.5/javadocs/org/hibernate/annotations/package-summary.html) which augment the O/R mapping annotations defined by JPA, and which may be used with the JPA-defined interfaces, or with the native API.

![[Pasted image 20240529125810.png]]

 정리 (번역)
1. `EntityMangerFactory`와 `EntityManager`를 구현하고 Object-Relation 어노테이션 구현
2. `SessionFactory`(extends `EntityManagerFactory`)와 `Session`(extends `EntityManager`)을 중심으로 사용 가능한 모든 기능을 노출하는 네이티브 API
3. JPA에 의해 정의된 O/R 매핑 어노테이션을 확장하는 매핑 어노테이션 집합

Native API
: 특정 플랫폼에 고유하게 제공되는 API (특정 플랫폼에 최적화, 종속됨)


