---
date: 2024-05-28
author: hyebeenbyeon
---

> 💻 참고한 사이트
[mappedby 필요한 이유 - velog](https://velog.io/@wogh126/JPA-%EC%96%91%EB%B0%A9%ED%96%A5-%EB%A7%A4%ED%95%91%EC%97%90%EC%84%9C-MappedBy%EA%B0%80-%ED%95%84%EC%9A%94%ED%95%9C-%EC%9D%B4%EC%9C%A0)

## 양방향 매핑
양방향 매핑은 두 객체가 서로 참조해야 하는 상황에서 정의하는 연관관계 방식이다. 실제로는 각각의 단방향 매핑이 존재하는 것이며, 이를 합쳐 양방향을 의미한다.

양방향 매핑을 할 때네는 반드시 한쪽 객체에 `mappedBy` 옵션을 설정해야 한다.

## Code
> Member.java
```java
@Entity
public class Member { 

	@Id @GeneratedValue
	private Long id;

	@Column(name = "USERNAME")
	private String name;
	private int age;

	@ManyToOne
	@JoinColumn(name = "TEAM_ID")
	private Team team;
    
	… Getter, Setter 생략
}
```

> Team.java
```java
 @Entity
 public class Team {
 
   @Id @GeneratedValue
   private Long id;
   private String name;
   
   @OneToMany(mappedBy = "team")
   List<Member> members = new ArrayList<Member>();
   
   … Getter, Setter 생략
 }
```

## mappedBy 사용하는 이유
![[Pasted image 20240529113056.png]]

만약 양방향 매핑을 적용한다면 Member 객체뿐만 아니라 Team 객체도 members 필드를 통해 MEMBER 테이블에 접근이 가능해지기 때문에 혼란이 생길 수 있다.

만약 Member 객체와 Team 객체가 서로 MEMBER 테이블의 데이터 값을 변경하려 한다면 무결성을 해칠 가능성도 생길 것이다.

그렇기 때문에 **두 객체중 하나의 객체만 테이블을 관리할 수 있도록 정하는 것**이 MappedBy 옵션인 것이다.

```java
@OneToMany(mappedBy = "team")
List<Member> members = new ArrayList<Member>();
```

예제에서는 Team 객체에 MappedBy가 적용되어 있는 것을 볼 수 있다.

`Team` 객체는 `MEMBER` 테이블을 관리할 수 없고, 
`Member` 객체만이 권한을 받고 `Member` 클래스의 `team` 필드를 통해 `team` 객체를 관리할 수 있다. 

## Owner
- `Team` 클래스의 `members` 필드
	- Owner
	- `mappedBy`가 설정된 필드를 가진 클래스는 주인이 아닌 쪽이다.
	- 읽기(조회)만 가능해진다. 
- `Mamber` 클래스의 `team` 필드
	- Inverse Side
	- `MappedBy`에 의해 설정된 `team` 필드가 주인이 된다.
	- (`team` 필드는 `members` 필드의 요소인 `Member`의 `team` 필드를 말한다.)

정의
- 주인 (Owner) 
	- 엔티티 관계에서 외래키를 실제로 관리하는 측을 의미한다.
	- `@JoinColumn` 어노테이션을 통해 매핑 정보를 정의한다.
	- 주인은 관계의 상태를 변경할 수 있으며, 데이터베이스에 반영된다.
- 비주인 (Inverse Side)
	- 주인이 아닌 다른 쪽의 엔티티 관계를 정의한다.
	- 외래키를 직접 관리하지 않는다.
	- `mappedBy` 어노테이션을 사용하여 주인을 지정한다.

일반적으로 외래키를 가진 객체를 주인으로 정의하는 것이 좋다.(예제도 외래키를 가진 Memeber 객체가 주인이 된 것이다.)