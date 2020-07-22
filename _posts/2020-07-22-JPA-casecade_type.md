---
layout: post
title: JPA Entity OneToMany 관계 한번에 Insert
tags: [JAVA, Spring, JPA]
categories:
    - Java
    - Spring
    - JPA
    - Hibernate
permalink: jpa-cascade_type.html
createat: 2020-07-22
---

이번 포스팅에서는 JpaRepository 를 이용 해서 @OneToMany 관계의 Parent 와 Child 를 함께 Insert 하는 방법을 다루어 보겠습니다.  

```java
@Entity
@Getter
@Setter
public class Parent{
    @Id
    @GeneratedValue (generator = "uuid2")
    @GenericGenerator(name="uuid2", strategy = "uuid2")
    @Column(name="id", columnDefinition = "BINARY(16)", updatable = false)
    private UUID id;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    private List<Child> childList = new ArrayList<>();
    
    public void addChild(Child child){
        child.setParent(this);
        this.childList.add(child);
    }
}

@Entity
@Getter
@Setter
public class Child{
     @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
     private Integer id;

     @ManyToOne(fetch = FetchType.LAZY, optional = false)
     @JoinColumn("parent_id")   
     private Parent parent;
}
```

양방항 매핑 관계를 가지고 있 부모 엔티티와 자식 엔티티의 구조입니다. 
Entity Manager 를 통해 부모 Entity 를 Persist 할 때 자식 Entity 와 함께 Insert 하려면 아래와 같은 사항을 체크해 보는 것이 좋습니다.
* CascadeType.PERSIST : @OneToMany 부분에 cascade 방식을 지정해 주는데 CascadeType.PERSIST 또는 이를 포함하는 CascadeType.ALL 로 선언해야 합니다.
* optional = false : @ManyToOne 어노테이션 에는 optional=false 를 지정해 주는데, 객체가 Null 이 들어갈 수 없음을 알려줍니다.
* addChild() 메서드 : 양방향 객체 바인딩을 해주는 메서드로 양쪽 Entity 에 각각 연관되는 Entity를 넣어 줍니다. 이 부분이 매우 중요한데, 이 연관관계의 주인(foreign key가 있는 엔티티)은 Child 엔티티 이기 때문에 Child 에게 Parent 가 무엇인지 정확히 알려주어야 정상적으로 양방향 매핑이 이루어 집니다. 

```java
@Service
public class Service {
    
    private ParentRepository parentRepository;

    public Service(ParentRepository parentRepository){this.parentRepository = parentRepository;}
    
    @Transactional
    public void save(){
        Parent parent = new Parent();
        Child child1 = new Child();
        Child child2 = new Child();
        
        parent.addChild(child1);
        parent.addChild(child2);
        parentRepository.save(parent);
    }
}

```  
그리고 위와 같이 Parent 를 save 하기 전에 Child를 addChild() 메서드로 양방향 관계를 성립시킨 뒤 Repository를 통해 save 해주면  
부모 엔티티와 자식 엔티티에 대한 Insert 쿼리가 동시에 나가는 것을 볼 수 있습니다.

엔티티의 양방향 바인딩 방법은 몇가지 더 있지만 다른 경우에는 JoinTable 을 생성한다던가, Insert 후 Update 를 한다던가 하는 예상과 다른 동작이 발생하는 것에 비해   
이 경우가 논리적으로 가장 정확한 방법이라고 할 수 있습니다. 

다른 경우와 관련된 내용을 원하시 hibernate 의 기본 매핑 전략에 대해 찾아보시는 것을 추천드립니다.

또 CascadeType.PERSIST 와 CascadeType.ALL 에 관한 차이와 다른 옵션에 대해서는 이 포스팅에서 다루지 않았지만  
전반적으로 이해하고 사용하시는 것을 추천드립니다.  