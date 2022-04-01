---
layout: post
title: 3개월 전의 내 포폴 내가 피드백하기 3탄
permalink: /etc/myself/3
position: etc
parent: etc
nav_order: 5
---

# 3개월 전 취준생 “나"의 포폴 피드백하기 3탄

<br />

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

<br />

## 이어서

<br />

- 예외 처리
- Repository 수정 - Native 쿼리는 피하자

예외도 예외인데 어제 그 3개원 전 Entity 클래스를 좀 더 개선해보자.

<br />

### 1. `@Builder` 를 개선해보자.

<br />

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "User", uniqueConstraints = {@UniqueConstraint(columnNames = "email")})
public class UserEntity {

    @Id @GeneratedValue(generator = "system-uuid")
    @GenericGenerator(name = "system-uuid", strategy = "uuid")
    private String userId;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password;

    private String mobileNumber;

    @CreationTimestamp
    private LocalDateTime createdTime;

    @UpdateTimestamp
    private LocalDateTime modifiedTime;

    @Builder
    public UserEntity(String name, String email, String password, String mobileNumber) {
        this.name = name;
        this.email = email;
        this.password = password;
        this.mobileNumber = mobileNumber;
    }

    public void updatePassword(String password) {
        this.password = password;
    }
}
```

<br />

일단 `@Builder` 를 이렇게 써줘도 깔끔하고 괜찮을 거 같다.

근데 password 업데이트하는 곳에 문제가 하나 있다. 다들 알고 계셨을까?

는 당연히 모르셨겠지? 내가 controller 단에서 비밀번호 인코딩을 안하고 넘겨줘서 문자열 그대로 저장된다. 

후후 

<br />

```java
@PutMapping("/auth/update")
public ResponseEntity<HttpStatus> updatePassword(@AuthenticationPrincipal String userId,
                                                 @RequestBody @Valid UserUpdateDTO userDTO) {
    userService.updatePassword(userId, passwordEncoder.encode(userDTO.getPassword()));
    return ResponseEntity.ok(HttpStatus.OK);
}
```

<br />

이건 첫 포폴 제출할 때 이미 알고 있었던 문제인데 나란 인간 누가 고치라고 안하면 안 고치는 사람이라. 

그냥 아 이거 문자열 그대로 변경되네 하하 하고 넘겼던 걸로 기억한다.

왜 그렇게 살았냐고요? 백엔드 쥐뿔도 몰라서 그랬습니다.


<br />

### 2. Native Query는 피합시다.

<br />

JPA를 사용하다가 조금 복잡한 쿼리가 나오면 query method 를 찾기 보다 무지성으로 `@Query` 를 썼었다.

왜 썼냐? 쉬우니까!

<br />

하지만 JPA를 쓰면서 네이티브 쿼리를 쓴다면 사실 JPA가 주는 장점을 누리기 힘들어진다.

JPA는 디비가 바뀌더라도 상관 없이 알아서 쿼리를 생성해서 전달하기 때문에 디비 방언(dialect)를 알 필요가 없어지는 장점이 있는데 이걸 그냥 네이티브 쿼리를 쓴다?

그냥 다시 DB 종속적이게 되는 거다. 변경할 때 네이티브 쿼리 쓴 거 다 뒤져봐야 한다.

<br />

아래는 내가 3개월 전에 쓴 네이티브 쿼리다.

지금 보니 굳이 쓸 필요 없는 것도 썼다. 하하.

<br />

```java
// 1번
@Query(value = "SELECT * FROM comment WHERE board_id = :boardId", nativeQuery = true)
List<CommentEntity> retrieveComment(@Param("boardId") String boardId);
	

// 2번
@Modifying
@Query(value = "DELETE FROM comment WHERE id = :id AND user_id = :userId", nativeQuery = true)
void delete(@Param("id") int id, @Param("userId") String userId);
```

<br />

위에 있는 1번 쿼리는 그냥 이렇게 작성해도 잘 된다.

```java
List<CommentEntity> findAllByBoardId(String boardId);
```

<br />

2번 쿼리는 변경되는 쿼리인데 저런건 그냥 조회하는 코드로 바꾼 후 서비스 단에서 삭제하는 로직으로 연결해주자.

아래처럼 레포지터리에서 찾고 서비스 단으로 넘겨서 삭제하는 로직으로!

<br />

```java
CommentEntity findByIdAndUserId(int id, String userId);
```

<br />

근데 삭제보단 그냥 삭제 표시를 하는 컬럼을 하나 만들어서 update 로 사용하지 않게 바꿔주는게 더 좋다.

생각보다 실제 db 데이터를 삭제하는 경우는 거의 없는 거 같다.

뭐 그런걸 알고 만들었냐만은.

<br />

이렇게 찾아서 넘겨주는게 좋긴 한데 삭제보다는 → 삭제 컬럼을 만들어서 표시하는 걸로!

아래와 같이 CommentEntity에 isDeleted를 만들어주고, 음 생각해보니 삭제 시간도 만들어주면 좋겠다.

이런거는 도메인 안에 메서드로 묶어주면 좋을 거 같군!

<br />

```java
private boolean isDeleted;
private LocalDateTime deletedAt;

public void delete() {
		this.isDeleted = true;
		this.deletedAt = LocalDateTime.now();
}

public void update(String comment) {
		this.comment = comment;
}
```

<br />

시간은 LocalDateTime을 쓰기도 하고 Instant를 쓰기도 하고. 다양한 거 같다. 

여튼 저런 메서드를 이제 서비스에서 전달만 해주면 된다.

<br />

```java
@Transactional
public void delete(int id, String userId) {
    final CommentEntity entity = commentRepository.findById(id).orElseThrow();
    if (!Objects.equals(entity.getUserId(), userId)) throw new CustomApiException("not same user");
    entity.delete();
}

@Transactional
public void update(CommentDTO commentDTO, Integer id) {
    final CommentEntity comment = commentRepository.findById(id).orElseThrow();
    comment.update(commentDTO.getComment());
}
```

<br />

저긴 이제 단순하게 바로 엔티티에서 업데이트하고 삭제하는 거긴 한데 도메인과 엔티티를 분리해서 하는게 더 좋긴 하다. 도메인이랑 엔티티 분리해서 하는 건 하고 있긴 한데 최근에 본 코드는(설로인임) 거기서 한 번 더 분리해서 인터페이스 만들어서 수정할 컬럼이랑 수정 안할 컬럼 나누고, Editor() 인터페이스 만들어서 쓰긴 하던데, 아직 거기까진 직접 개발해본 게 아니라서 잘 모르겠다. 

인터페이스 분리하는게 코드가 많아지고 귀찮긴 한데 애플리케이션 사이즈가 커지면 커질수록 더 잘 분리된 코드일수록 좋다고 하긴 하더라. 난 아직 잘 모르겠다.

<br />

위에 말한 최근에 본 코드라는게 이 부분인데(출처 : 설로인 샌드박스)

<br />

```kotlin
// POINT: 왜 User 도메인 모델을 mutable 로 설계하지 않고, 이런 타입을 별도로 만들었을까요?
interface Editor : User {
    override var nickname: String

    override var profileImageUrl: String

    /**
     * 이 필드에 직접 assign 하지 마시고, delete 메소드를 활용하세요.
     * 이 필드에 직접 assign 할 경우 문제가 생길 수 있습니다.
     */
    override var deletedAt: Instant?

    override var updatedAt: Instant

    override fun edit(): Editor = this

    fun delete(instant: Instant = Instant.now()): Editor {
        this.updatedAt = instant
        this.deletedAt = instant
        return this
    }
}
```

<br />

User 도메인도 인터페이스로 만들고 val로 수정 안 할 컬럼은 위에 다 빼놓고 수정할 컬럼만 Editor에 상속 받아서 담고 있었다. 여기서도 필드에 직접 assign 하지 말고 메소드를 이용하세요~ 라고 친절히 말 해주고 있네.

여기서 이런 타입을 따로 만들고 위에 직접 assign 할 때 문제가 생길 수 있다고 경고문을 써준 이유는 아마도 직접 data를 set하지 못하게 막고 수정할 수 있는 컬럼만 따로 빼둔 거 위해서 같다.

좀 궁금한 점은 이건데.

<br />

```kotlin
val isDeleted: Boolean
        get() = deletedAt != null
```

<br />

삭제 표시하는 컬럼을 val로 막아두고 deleteAt이 null일 때는 get을 못하게 막아뒀다. 

근데 생성될 때는 저기 기본값으로 false가 들어갈 거 같은데 실제 delete 메서드 쪽에서는 isDeleted를 true로 바꿔주는 로직이 안보이는데 테스트에서 저게 true인 걸 검증하는 로직이 있다.

테스트를 실행해보고 싶긴 한데 java 버전이 안맞아서 아직 실행을 못시켜봤다. 나중에 해봐야지.

<br />

여튼 여러모로 좋은 코틀린 공부 소스다. ~~머리에 자극이 너무 많이 와~~

<br />

### 3. 예외 처리도 써야 하는데 ... 이건 진짜 내일!

이건 진짜 내일 쓰겠다. 

이 글도 어제부터 쓴 건데 build.gradle만 올리고 끝냈네.

근데 사실 예외 처리는 아직도 잘 모르겠다. 어려워. 웹 플럭스는 예외 처리 안한다매요? 

배보다 배꼽이 더 크게 공부해야할 거 같긴 한데 어우어 오늘은 모나드 2탄도 부디 글을 쪄보겠습니다😂
