---
date: 2024-05-28
author: hyebeenbyeon
---

> 💻 참고한 사이트
[javax/persistence/EntityManager - docs](https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html)
> 📌 참고 하면 좋을
[jakarta/persistence/EntityManager - docs](https://jakartaee.github.io/persistence/latest/api/jakarta.persistence/jakarta/persistence/EntityManager.html)

## Entity Manager

In terms of **Persistence Context**
- 영속성 컨텍스트(_persistence context_)와 상호작용 하는데 사용되는 인터페이스이다.
- `EntityManager`의 인스턴스는 persistence context와 연결된다.

In terms of **Persistent Entity**
- persistence context는 entity 인스턴스의 집합으로, 모든 _persistent entity_(영속성 엔티티) ID에 대해 고유한 엔티티 인스턴스가 존재한다.
- persistence context 내에서 엔티티 인스턴스와 그 수명 주기가 관리된다.
- `EntityManager` API는 영속성 엔티티 인스턴스(persistent entity instance)를 생성 및 삭제, 기본키오 엔티티 찾기, 엔티티에 대한 쿼리에 사용된다.

In terms of **Persistence Unit**
- 주어진 `EntityManager` 인스턴스가 관리할 수 있는 엔티티 집합은 _persistence unit_(영속성 단위) 로 정의된다. 
- persistence unit은 어플리케이션에 의해 관련되거나 그룹화되고 단일 데이터베이스에 대한 매핑에 배치되어야 하는 모든 클래스의 집합을 정의한다.

### Entity Transaction
Interface used to control transaction on resource-local entity manager.
The `EntityManager - getTransaction` method returns the `EntityTransaction` interface.

