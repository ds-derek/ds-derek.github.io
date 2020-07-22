---
layout: post
title: OneToMany 관계 한번에 Insert
tags: [JAVA, Spring, JPA]
categories:
    - Java
    - Spring
    - JPA
    - Hibernate
permalink: jpa-cascade_type.html
createat: 2020-07-22
---

이번 포스팅에서는 JpaRepository 를 이용 해서 @OneToMany 관계의 Parent 와 Child 를 함께 Insert 하는 방법을 다루겠습니다.
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

     @OneToMany(fetch = FetchType.LAZY, optional = false)
     @JoinColumn("parent_id")   
     private Parent parent;
}
```

