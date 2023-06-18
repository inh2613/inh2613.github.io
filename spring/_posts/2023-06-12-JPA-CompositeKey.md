---
layout: post
title: JPA 복합키 매핑
description: >
  REPL-Note 토이 프로젝트 당시 경험했던 JPA 복합키 매핑 관련 내용입니다. 
sitemap: false
hide_last_modified: true
---

---

## 배경지식

 데이터베이스 테이블 관계는 **외래키가 기본키에 포함되는지 여부에** 따라 ```식별 관계```와 ```비식별 관계```로 구분한다. 

### 식별관계
- 부모 테이블의 기본키를 내려받아서 자식 테이블의 기본키+외래키로 사용하는 관계

### 비식별 관계
- 부모 테이블의 기본키를 받아서 자식 테이블의 외래키로만 사용하는 관계
- 외래키에 NULL을 허용하는지에 따라 ```필수적 비식별 관계```와 ```선택적 비식별 관계``` 나눈다. 

    - 필수적 비식별 관계
      - 외래키에 NULL을 허용하지 않는다.(연관관계를 필수적으로 맺어야 한다. )
    - 선택적 비식별 관계
      - 외래키에 NULL을 허용한다. 연관관계를 맺을지 말지 선택할 수 있다.

DB 설계시 식별관계나 비식별 관계 중 하나를 선택해야한다. 최근에는 비식별관계를 주로 사용하고 꼭 필요한 곳에서만 식별관계를 사용하는 추세라고 한다. JPA는 두 관계를 모두 지원한다.

## JPA 복합키 매핑

식별자 필드가 하나일 때는, 보통의 자바의 기본 타입을 이용해서 문제가 없다.

하지만 식별자 필드가 2개이상이면 별도의 식별자 클래스를 만들고 그곳에 ```equals```와 ```hashcode```를 구현해야 한다.

JPA는 복합키를 지원하기 위해 ```@IdClass```(RDB에 가까운 방법)과 ```@EmbeddedId```(객체지향에 더 가까운 방법)을 제공한다.

### 1. IdClass

```

```


### 2. EmbeddedId
```
@Data
@Embeddable
public class TodoId implements Serializable {

    @Column(name = "todo_id")
    private String todoId;

    @Column(name = "room_id")
    private Long roomId;


    public String getTodoId(){
        return todoId;
    }
    public Long getRoomId(){return roomId;}
    
}
```

```
@Data
@Entity
@Table(name = "todo")
@EntityListeners(AuditingEntityListener.class)
public class Todo implements Serializable {

    @EmbeddedId
//    @Id -> 주석처리를 꼭 해줬어야 에러가 안났음
    @Column(name = "todo_id")
    private TodoId todoId;


    @Column(name = "is_finished", columnDefinition = "TINYINT", length = 1)
    @ColumnDefault("0")
    private int isFinished;

    @Column(name = "isShared", columnDefinition = "TINYINT", length = 1)
    @ColumnDefault("0")
    private int isShared;

    @CreatedDate
    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;


    public TodoId getTodoId(){
        return todoId;
    }
}
```


## 참고
- https://www.nowwatersblog.com/jpa/ch7/7-3
- https://steady-hello.tistory.com/106





    



