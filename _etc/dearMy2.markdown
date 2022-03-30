---
layout: post
title: 3개월 전의 내 포폴 내가 피드백하기 2탄
permalink: /etc/myself/2
position: etc
---

# 3개월 전 취준생 “나"의 포폴 피드백하기 2탄

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

<br/>

저번에는 3가지를 봤다.

- `@Autowired`
- DTO에 담을 데이터
- 롬복의 `@Data`

말할 게 한 바가지라 포스팅 하나로 끝나지 않는다 ^ㅁ^

<br/>

## DTO mapper 얘기를 쓰려 했는데...

<br/>

이건 사실 내가 잘못 했다고는 생각하지 않는데 3개월 간 다양한 방식을 접하면서 오히려 좀 더 생각이 **복잡해졌다**.

일단, 내가 3개월 전에 처음으로 dto를 썼다는 건 미리 알렸고, 

그 당시에 dto ↔ entity 간에 매핑은 dto 안에 toEntity라는 static 메서드를 만들어서 했다.

**아니 근데 지금 보니 내 코드를 이해하기가 힘드네**

mapper 얘기를 쓰려 했는데 일단 잠시 코드부터 보자.

<br/>

```java
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Data
@Entity
@Table(name = "User", uniqueConstraints = {@UniqueConstraint(columnNames = "email")})
public class UserEntity {

	@Id
	@GeneratedValue(generator="system-uuid")
	@GenericGenerator(name="system-uuid", strategy="uuid")
	private String userId;
	
	@Column(nullable = false)
	private String name;
	
	@Column(nullable = false, unique = true)
	private String email;
	
	@Column(nullable = false)
	@JsonIgnore
	private String password;
	
	private String mobileNum;
	
	@Column(nullable = false)
	private LocalDateTime createdTime;
	
	private LocalDateTime modifiedTime;
	
	@PrePersist
	public void createdTime() {
		this.createdTime = LocalDateTime.now();
	}

	public void updatePassword(String password) {
		this.password = password;
	}
}
```

위에 어노테이션에 `@Builder` 가 붙어 있다.

빌더 패턴을 쓴 건데 일단 빌더 패턴 1도 모르고 썼다. 그냥 매개변수 많을 때 쓰기 편하다더라~ 정도의 배경 지식으로 썼다. ~~반성합니다.~~

<br/>

### Builder 패턴

<br/>

[여기 글 출처](https://stackoverflow.com/questions/328496/when-would-you-use-the-builder-pattern)

이펙티브 자바에 빌더 패턴이 소개되어 있다.(난 읽은 적 없지만)

```java
The builder pattern is a good choice when designing classes 
whose constructors or static factories 
would have more than a handful of parameters.
```

<br/>

생성자나 static 팩토리가 한 줌 이상의 매개변수를 가져야할 때 좋은 선택이라고 합니다.

가령 이럴 때 말이죠. 아래처럼 생성자 리스트를 마주칠 때가 있죠?

```java
Pizza(int size) { ... }        
Pizza(int size, boolean cheese) { ... }    
Pizza(int size, boolean cheese, boolean pepperoni) { ... }    
Pizza(int size, boolean cheese, boolean pepperoni, boolean bacon) { ... }
```

<br/>

이런 걸 telescoping 생성자 패턴이라고 한답니다. 생성자가 4개, 5개가 넘어가면 매개변수의 순서나 상황에 따라 필요한 특정 생성자를 기억하기 어려워집니다. 가령 저기서 페퍼로니를 먼저 넣을지 베이컨을 먼저 넣어야할 지 기억하기가 힘들어진다는 점이죠.

여기서 한 가지 대안으로 나온게 Java Beans 패턴인데 그냥 new 생성자로 피자 만들어서 치즈, 페퍼로니 등을 하나하나 set 해주는 그걸 말합니다.(~~노가다 패턴~~) 

---

그래서 나온 다음 대안이 빌더 패턴입니다.

두둥!

```java
public class Pizza {
  private int size;
  private boolean cheese;
  private boolean pepperoni;
  private boolean bacon;

  public static class Builder {
    //required
    private final int size;

    //optional
    private boolean cheese = false;
    private boolean pepperoni = false;
    private boolean bacon = false;

    public Builder(int size) {
      this.size = size;
    }

    public Builder cheese(boolean value) {
      cheese = value;
      return this;
    }

    public Builder pepperoni(boolean value) {
      pepperoni = value;
      return this;
    }

    public Builder bacon(boolean value) {
      bacon = value;
      return this;
    }

    public Pizza build() {
      return new Pizza(this);
    }
  }

  private Pizza(Builder builder) {
    size = builder.size;
    cheese = builder.cheese;
    pepperoni = builder.pepperoni;
    bacon = builder.bacon;
  }
}
```

<br/>

이렇게 써서 아래처럼 빌더의 setter 메서드를 이용해 빌더 객체를 연결한 형태로 리턴합니다.

<br/>

```java
Pizza pizza = new Pizza.Builder(12)
                       .cheese(true)
                       .pepperoni(true)
                       .bacon(true)
                       .build();
```

이렇게 쓰면 읽기 쉽고 이해하기 쉽다~ 그래서 매개변수가 미래에 많아질 클래스가 있다면 고려해 봐라~

라는게 빌더 패턴을 쓰는 이유 였습니다.

~~에휴~~

---

<br/>

빌더 패턴이 유연하다는 점을 방금 글 쓰면서 이해했다. 대충 아래 같이 빌더 객체 만들 때 저 안에 데이터를 필수적인 데이터를 지정할 수 있고, 옵셔널하게 매개변수 몇 개 빼고도 객체 생성이 가능하구나. 

```java
UserEntity user = UserEntity.builder().email(userDTO.getEmail()).name(userDTO.getName())
                .mobileNum(userDTO.getMobileNum()).password(passwordEncoder.encode(userDTO.getPassword())).build();

UserEntity user = UserEntity.builder().email(userDTO.getEmail())
                .password(passwordEncoder.encode(userDTO.getPassword())).build();
```

와 나 정말 알고 쓴 게 하나도 없구나.

<br/>

---

<br/>


### Lombok의 `@Builder`

<br/>

출처 : Builder.java

그럼 다음으로 lombok의 `@Builder` 를 보자. 

```java
Before:
   @Builder
   class Example<T> {
   	private T foo;
   	private final String bar;
   }
   
After:
   class Example<T> {
   	private T foo;
   	private final String bar;
   	
   	private Example(T foo, String bar) {
   		this.foo = foo;
   		this.bar = bar;
   	}
   	
   	public static <T> ExampleBuilder<T> builder() {
   		return new ExampleBuilder<T>();
   	}
   	
   	public static class ExampleBuilder<T> {
   		private T foo;
   		private String bar;
   		
   		private ExampleBuilder() {}
   		
   		public ExampleBuilder foo(T foo) {
   			this.foo = foo;
   			return this;
   		}
   		
   		public ExampleBuilder bar(String bar) {
   			this.bar = bar;
				return this;
   		}
   		
   		@java.lang.Override public String toString() {
   			return "ExampleBuilder(foo = " + foo + ", bar = " + bar + ")";
   		}
   		
   		public Example build() {
   			return new Example(foo, bar);
   		}
   	}
   }
```
<br/>

Example 클래스에 package-private한 생성자가 생기고, ExampleBuilder이라는 빌더 클래스가 필드 이름마다 1개씩 생성된다. 뉴 빌더로 빌더 인스턴스 만들고 저기 build() 메서드로 실제 생성자를 호출해서 객체를 생성하나 보다.

근데 경고 문구 하나가 눈에 띄는데, 생성자의 경우, 생성자를 작성하지 않았으면서 명시적으로 `@XArgsConstructor` 어노테이션도 없을 때만 생성된다고 한다. 

<br/>

그럼 나는 저기 왜 `@NoArgs, @AllArgs~` 를 붙였을까. 

NoArgs 야 jpa entity니까 붙였을 거고, AllArgs는 빌더 때문에 같이 붙였나 보다. 

밑에 읽어보니 lombok이 AllArgs를 가정하고 사용하는 코드를 생성한다고 하네. 저기서 AllArgs 어노테이션이 없었다면 컴파일 에러가 났을 수도 있겠다.

흐어 그래도 왜 쓴지 약간은 알 것도 같아진 게 발전이라면 발전이다.

<br/>

### 그래서 mapper는?

<br/>

돌고 돌아 매퍼로 돌아왔다.

mapper를 쓰는 방식이 1가지에서 3가지로 늘었다.

1. 서비스 계층에서 Entity Builder로 DTO 값을 하나하나 넣어주는 방식
2. model mapper 등의 라이브러리를 이용하는 방식
3. 엔티티 생성자나 빌더에 dto를 넘기고 그 안에서 값을 넣어주는 방식

어떤게 좋은 방식일까? **뭔가 정해진 답은 없는 것 같다.**

<br/>

model mapper 처음 쓸 때만 해도 세상 이렇게 편한게 있나! 하고 좋아했었는데 DTO랑 도메인 이름이 달라지니 결국 서비스에서 하나하나 넣어줘야 해서 진짜 이런 계륵 같은게 있나 하고 씹어대다가

아니 그럼 이걸 별도의 static 메서드를 만들어서 하자!! 하고 생각했었는데(코틀린으로 companion object를 생각했었음) 또 그건 별로인 거 같다고 얘기를 듣기도 하고.

<br/>

요즘은 클린 코드 같은 책을 읽다 보니 **그냥 쉬운게 최고인 거 같다.** 진짜 대충 살고 싶어.

라이브러리 쓰지 말고 mapper 클래스 따로 만드는 것도 좋은 선택인 거 같다.(어차피 이름 다르면 제대로 매핑도 안되는 거, 라이브러리 쓴다고 거기에 얽매이게 되는 것도 뭔가 의존적이게 되는 거 같고)

그냥 엔티티 생성자 쓰자~!!(DTO 파라미터 갖게 하는건 말고! 그것도 의존적이니까...) 이게 포조인가?!

<br/>

### 그 외 짜투리 생각

전에 프로젝트 보니까 dto → entity 매핑을 컨트롤러에서 수행하고 있다. 

별 고민 없이 선택한 것이긴 한데 현재는 dto를 서비스로 넘기고 변환은 서비스 계층에서 모두 처리하고 있다. 

근데 이것도 하다 보며 느낀 건데 controller랑 서비스랑 분리되는 거 같지 않다. ~~컨트롤러 수정하면 서비스 결국 수정해야 하고 서비스 수정하면 또 컨트롤러 수정하고 있고 대체 분리라는 건 어떻게 하는 거야.~~

~~그리고 그렇게 dto랑 도메인을 분리하는 건 정말 매핑하는 cost를 무시할만큼 좋은 것일까 흠믐믐~~

<br/>

외국 개발자들 깃헙 타고 들어가보면 dto도 인터페이스를 따로 만들고 뭐 responseBase 이런거 extends 하면서 인터페이스를 implements 해서 쓰고 그러던데. 

뭐 예를 들어서, 유저 생성하는 인터페이스를 아래처럼 만들고.

```tsx
export interface CreateUser {
  email: string;
  country: string;
  postalCode: string;
  street: string;
}
```

<br/>

실제 유저 생성할 때 요청하는 dto는 이렇게 쓴다.

```tsx
export class CreateUserRequest implements CreateUser {
  @ApiProperty({
    example: 'john@gmail.com',
    description: 'User email address',
  })
  @MaxLength(320)
  @MinLength(5)
  @IsEmail()
  readonly email: string;

	@ApiProperty({ example: 'France', description: 'Country of residence' })
  @MaxLength(50)
  @MinLength(4)
  @IsString()
  @Matches(/^[a-zA-Z ]*$/)
  readonly country: string;

	... 생략

}

export class CreateUserHttpRequest extends CreateUserRequest
  implements CreateUser {}

export class CreateUserMessageRequest extends CreateUserRequest
  implements CreateUser {}
```

<br/>

위는 타입스크립트 예시긴 한데 이거 뭐 이렇게 쓰니 자바스크립트랑 타입스크립트랑 뭐가 다른가 싶기도 하네.

여튼 아직 사실 어디에서 dto를 매핑해야 하는게 옳은지는 잘 모르겠다.~~(매핑 하는 것도 나만 귀찮나??)~~

나란 쪼랩 생각만 많고 해결책은 1도 모르는 쪼랩...

<br/>

### 개인적인 결론

<br/>

**정답 아니고 정말 개인적인 생각이다.**

- 단순한게 좋다.
- controller에서 service로 넘길 때 그냥 값만 단순히 던져주는게 좋은거 같다.
- 근데 dto에 담긴게 많으면 너무 길어져서 그 때는 dto를 매퍼로 바꾸긴 해야 할 거 같은데
    - 그 작업을 컨트롤러에서 해야하는지 서비스에서 해야하는지는 잘 모르겠다.
    - 컨트롤러가 데이터 받는 곳이니 매핑도 해줄 수 있는 거 아닐까? 싶기도 하고.
    - 서비스는 비즈니스 로직만 담는다는 생각을 하면 여기서 왜 매핑을 하고 있는지 모르겠고.
    - 그렇다고 그걸 레포지터리에서 한다? 그건 진짜 모르겠다.
- 매핑하는 것도 귀찮다 휴
- 하지만 뾰족한 대안은 모르겠다. 나중에 알게되면 이 글에 업데이트 해야겠다.

<br/>

## 소소하게 변경한 거

<br/>

`@CreationTimestamp` , `@UpdateTimestamp`

하이버네이트에서 제공하는 어노테이션이다. 엔티티 객체가 INSERT, UPDATE 될 때 알아서 현재 시간을 자동으로 설정해 준다.

전에 코드에선 `@PrePersist` 를 썼었는데 위의 두 개는 JPA 관련이라 음.

<br/>

시간 관련해서도 사용하는 방법은 가지각색인 것 같다.

**Hibernate**

- `@CreationTimestamp`
- `@UpdateTimestamp`

<br/>

**Spring Data JPA** 

- `@CreatedDate`
- `@LastModifiedDate`

<br/>

**PrePersist, PreUpdate, PreRemove**

```java
@PrePersist
protected void prePersist() {
    if (this.dataChangeCreatedTime == null) dataChangeCreatedTime = new Date();
    if (this.dataChangeLastModifiedTime == null) dataChangeLastModifiedTime = new Date();
}

@PreUpdate
protected void preUpdate() {
    this.dataChangeLastModifiedTime = new Date();
}

@PreRemove
protected void preRemove() {
    this.dataChangeLastModifiedTime = new Date();
}
```

<br/>

최근에는 Instant를 이용해서 쓰는 것도 봤다.

```kotlin
internal data class Model(
    override val uuid: UUID,
    override var nickname: String,
    override var profileImageUrl: String,
    override var deletedAt: Instant?,
    override val createdAt: Instant,
    override var updatedAt: Instant,
    override val version: Long
) : Editor

@Suppress("LongParameterList")
fun create(
    uuid: UUID? = null,
    nickname: String,
    profileImageUrl: String,
    deletedAt: Instant? = null,
    createdAt: Instant? = null,
    updatedAt: Instant? = null,
    version: Long? = null
): User {
    val now = Instant.now()

    return Model(
        uuid = uuid ?: UUID.randomUUID(),
        nickname = nickname,
        profileImageUrl = profileImageUrl,
        deletedAt = deletedAt,
        createdAt = createdAt ?: now,
        updatedAt = updatedAt ?: now,
        version = version ?: Versioned.DEFAULT_LONG_INT
    )
}
```

<br/>

근데 저기 생성자에 ? = null 넣는 거 좀 그런 거 같다. 음 ... 하지만 다른 해결책은 잘 모르겠군. ㅠ

근데 여기 Model을 Editor 인터페이스를 상속해서 internal data class를 만드네. 

Editor 인터페이스를 User 인터페이스를 상속하고. 와 이거 구조 왜 이러지. (이건 다음 설로인 2탄에서...)

<br/>

에휴 다른 곳들도 진짜 미치겠지만 User 하나만 보기도 이렇게 힘들다니. 

아 그리고 이럴 거면 왜 ControllerExceptionHandler 만든지 모르겠지만 제대로 쓰지도 않는 예외 처리 코드들도 내일 이어서 수정해봐야 겠다.

<br/>

## 다음 이야기

<br/>

- 예외 처리
- 한글 깨지는 현상 → 이건 그냥 영어로 쓰자!
    
    ```kotlin
    @NotBlank(message="��й�ȣ�� �����Դϴ�.")	
    @Pattern(regexp="(?=.*[0-9])(?=.*[a-zA-Z])(?=.*\\W)(?=\\S+$).{8,20}",
    message = "��й�ȣ�� ���� ��,�ҹ��ڿ� ����, Ư����ȣ�� ��� 1�� �̻� ���Ե� 8�� ~ 20���� ��й�ȣ���� �մϴ�.")
    
    @NotBlank(message="�̸����� �����Դϴ�.")	
    @Email(message = "�̸��� ���Ŀ� ���� �ʽ��ϴ�.")
    ```
    
- Repository 수정 - Native 쿼리는 피하자
- 하 다른 건 내일 생각하자. 머리 아프다.


<br/>

엉엉 내가 쓴 코드 너무 보기 싫다 엉엉엉

뭘 알고 쓴 게 아니니까 이걸 이 때 왜 이렇게 썼는지를 다시 공부해야 하네 엉엉

이래서 알고 쓰라고 뭐라고 하시는 건가보다 엉엉😭😭