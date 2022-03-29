---
layout: page
title: 3개월 전의 내 포폴 내가 피드백하기 1탄
permalink: /etc/myself
position: etc
---

# 3개월 전 취준생 “나"의 포폴 피드백하기 1탄

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

# 시작

3개월 전 입사하기 전의 나는 풀스택으로 공부를 하고 있었다. 정확히 말하자면 프론트에 좀 더 집중해서 공부를 하고 있었는데 내 깃헙만 보더라도 자바스크립트 공부 흔적을 어렵지 않게 찾을 수 있다.

<br/>

![image](https://user-images.githubusercontent.com/84627144/160537012-118e9f52-5011-4dd8-86fb-a823c7736900.png)

백엔드는 프론트를 위해 필요한 api 정도만 만들었지 그 이상 따로 공부하고 있던 건 없었다. 다만 자바의 정석 스터디는 하고 있었다. 

3개월 동안 백엔드 개발자로 공부하면서 수도 없이 많은 피드백을 받았는데 그 부분을 상기하면서 과거 3개월 전의 취준생이었던 내가 했던 포트폴리오를 3개월 후의 내가 피드백을 해보자.

3개월 전에 만든 포폴 어디서 피드백 받아보지도 못했는데ㅋㅋㅋ큐ㅠㅠ 

이렇게 스스로 하는 것도 재밌을 것 같다. 이걸 바탕으로 기초적인 백엔드 CRUD 정도는 만들어볼까 생각도 하고 잇다.

<br/>

# 3개월 전 내 백엔드 포폴 깃헙

[https://github.com/inseo24/dev_cal_spring_v1.git](https://github.com/inseo24/dev_cal_spring_v1.git)

부끄럽지만 같은 이름의 리액트 쪽은 코드가 더 난잡하다. ㅎㅎ

여튼 하나하나 다시 돌아보자.

<br/>

# 수정 사항

## 1. @Autowired

<br/>


### 옛날에는

<br/>

UserController.java

```java
@Autowired
private UserService userService;

@Autowired
private UserRepository userRepo;

@Autowired
private TokenProvider tokenProvider;
```

<br/>

인텔리제이에서도 말리는 `@Autowired` (Field injection is not recommended)

당연히 이클립스 쓰던 나는 잘 몰랐음^ㅇ^


![image](https://user-images.githubusercontent.com/84627144/160542598-7ff3decc-163e-4db5-8fe4-e75ef005ade2.png)


**첫 피드백**에서 바로 받았던 내용이다. 

<br/>

👨‍🏫 : `@Autowired` 대신 생성자 주입을 쓰는게 좋습니다. 인텔리제이 쓰시면 거기 줄도 쳐지지 않나요?

👧 : 이클립스 써서 그런 거 없었는데 (...)

<br/>

Injection에는 보통 3가지 정도 방법이 있다.

1. 생성자를 사용하기
2. setter나 다른 메소드
3. 필드 주입하는 방식

`@Autowired` 는 위에서 3번에 해당한다. 

<br/>

스프링에서 **권장**하는 가이드라인은 이렇다.

- 필수적인 dependency나 불변성이 목적이면 생성자 주입을 사용할 것
- 선택할 수 있거나 바꿀 수 있는 dependency라면, setter 주입을 사용할 것
- 대부분의 경우에선 field 주입은 나중에 생각하는게 좋습니다.

<br/>

일단 `@Autowired` 는 알다시피 의존성 주입할 때 사용하는 어노테이션이다. 

내 기억에 **면접 때** `@Autowired` 에 대해 질문을 받았고, 나는 의존성 주입할 때 쓰는 어노테이션 아닌가요? 라고 했다가 어떻게 주입해주는지 아냐는 꼬리 질문을 받았다.

몰랐기 때문에 잘 모르겠다고 대답했던 기억이 있다.🙄

<br/>

`@Autowired` 객체를 IoC 컨테이너에서 생성해 주입한다. 의존성 주입(DI)이란 객체를 사용하는 곳이 아닌 외부에서 객체를 생성해 주입하는 것을 말한다. 즉, 의존 객체의 타입에 해당하는 빈을 DI 컨테이너가 찾아 주입한다. 

<br/>

내가 쓴 코드를 예시로 들자면,

UserController 인스턴스는 Spring IoC 컨테이너에서 생성하고, UserController 객체를 생성하는 시점에 UserRepository 인터페이스에 맞는 객체를 생성한 후 주입하게 된다. 

DI 컨테이너와 강하게 결합되서 외부에서 사용할 수 없다는 단점이 있다.

→ DI 프레임워크가 있어야만 된다는 것

→ 테스트 코드를 짤 때 필드 객체를 수정할 수 없다는 점 등 

여러 단점들로 인해 생성자 주입을 많이 사용한다. 

<br/>

### 수정해보자

<br/>


1번 방식

```java
private final UserService userService;
private final TokenProvider tokenProvider;

public UserController(UserService userService, TokenProvider tokenProvider) {
	this.userService = userService;
	this.tokenProvider = tokenProvider;
}
```

2번 방식

```java
@RestController
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;
    private final TokenProvider tokenProvider;

...
```

변수명도 정확하게 써주는게 좋다고 생각해서 길더라도 다 적고 있다.

전에 쓴 코드들을 보니 변수명이 repo, imgRepo 이런 식으로 적은 것도 있는데 ~~변수명 안습~~

final은 코틀린에서 val 쓰는 것처럼 쓰려고 하다 보니 그렇게 썼다. 

이게 하나 더 달라진 점인데 위에 `@Autowired` 로 필드 주입을 하면 final을 쓸 수가 없다. 초기화 후에 빈 객체가 변경될 수 있기 때문이다. 위에 잠깐 언급된 불변성과 관련된 점인데 필드 주입의 경우 불변 객체로 쓸 수가 없게 된다.

<br/>

### 정리

1. `@Autowired` 를 쓰지 말자
2. 왠만하면 final을 붙여 런타임에 변경되지 않는 불변 객체로 두자.(오류는 미연에 방지하자)

~~근데 글 쓰면서 지웠는데 컨트롤러 단에 userRepository는 왜 있는거냐 하...~~

<br/>

## 2. DTO는 필요한 데이터만 담자

<br/>

### 이 때 왜 이랬냐

회원가입을 하면 리턴되는 데이터다. 

```java
{
    "token": null,
    "email": "jnh022@naver.com",
    "name": "seoin",
    "password": null,
    "userId": "5aa6099c7fd47420017fd475ace50000",
    "mobileNum": null
}
```

~~대체 텅텅 빈 데이터는 왜 리턴하는 건데~~

<br/>

이름 정도는 필요할 수도 있겠다. 안녕하세요 서인님! 이런 거 정도 써주려면.

저 포트폴리오의 작성할 당시의 나로 다시 돌아가보자. 

나는 저 당시 DTO를 처음 써봤다. 전에도 얘기했듯이 프론트 위주로 공부했었고(~~그마저도 잘하진 못했지만~~)

트위터에서 여자 개발자 분이 리액트랑 자바 스프링으로 책을 출판하셨다길래 그 책을 샀고 거기서 처음 접했다.

그리고 저 개인 프로젝트에 처음 쓰게 된 것이다.

<br/>

그럼 학원에서 어떻게 했냐고요? ~~뭘 어떻게 해 그냥 엔티티 반환하고!@#$$#~~

학원에서도 프론트 위주로 학습했기 때문에 백엔드 쪽은 컨트롤러, 엔티티, 레포지터리 이 딱 3개로 투두리스트 만들었다. (실화다)

![image](https://user-images.githubusercontent.com/84627144/160554391-18d64558-6d6a-4a3c-bf4c-44ed24bad593.png)

이쯤되니 내가 왜 백엔드로 취업이 됬는지 의문이 드는데(대체 왜?)

<br/>

### 수정하자면

```java
{
	"code": 2000,
	"message" : "얄라리 얄라",
	"data" : [ ... ]
}
```

여튼 DTO는 **필요한 데이터만 담는게 좋다**. 

개인적으로는 메서드마다 나누는 게 더 좋다고 생각한다. 예를 들어서 유저를 생성할 때 필요한 DTO 파일 따로, 수정할 때는 수정에 필요한 데이터만 담긴 DTO 따로, 뭐 이런 식이다.

그리고 데이터를 리턴하는 형식을 만드는 것도 좋다. ~~그냥 보기 좋은 떡이 맛도 좋다고, 깔끔하잖아요.~~

<br/>

이건 회사마다 형식이 다를 거 같긴 한데 내가 새로운 개인 프로젝트를 만든다면 위처럼 code, message, data 로 만들고, 리턴하는 dto는 저 data 안에 담을 거 같다.

<br/>

### 고민되는 것

error를 response 하는 방식이 여러가지가 있겠지만, 현재는 enum 클래스를 쓰고 있다. 아마 다른 회사에서도 enum class를 쓰는 경우가 여럿 있겠지?

근데 개인적으로 **error를 enum으로 만드는게 과연 좋은 건가?** 라는 생각이 있다. 

errorResponse enum class 데이터 구조가 바뀌면 그거 참조하는 부분 다 변경해야 할 거고, 이 클래스 쓰는 다른 클래스들도 전부 다시 컴파일해야 하는 거 아닐까. 기존 오류 코드를 그냥 재활용하는게 더 좋지 않을까 라는 생각이 들긴 하지만. 나에겐 더 뾰족한 생각이 없기 때문에 ^ㅇ^

<br/>


## 3. 롬복의 `@Data` 는 조심하자 - Setter, toString() ...

<br/>

코틀린에서 data class를 최소화해서 사용하는 것과 비슷한 이유다.

너무 편한건 오히려 독이 된달까.

<br/>

### 과거

```java
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Data
@Entity
@Table(name = "Board")
public class BoardEntity {
 ...
}
```

저 다닥다닥 붙어 있는 어노테이션들 😅

코틀린에서 JPA Entity에 data class 대신 class를 쓰는 데엔 여러 이유가 있다. 

실제 [스프링의 오피셜 튜토리얼](https://stackoverflow.com/questions/58127353/should-i-use-kotlin-data-class-as-jpa-entity)에도 jpa에 data class를 쓰지 말라고 언급하고 있다.

<br/> 

> 
Here we don’t use `[data` classes](https://kotlinlang.org/docs/reference/data-classes.html) with `val` properties because JPA is not designed to work with immutable classes or the methods generated automatically by `data` classes. 
If you are using other Spring Data flavor, most of them are designed to support such constructs so you should use classes like `data class User(val login: String, …)` when using Spring Data MongoDB, Spring Data JDBC, etc.
> 

val 프로퍼티를 담은 data class를 사용하지 않습니다. 

왜냐면 JPA는 불변 클래스나 data class가 자동 생성한 메서드와 함께 사용할 걸 생각하고 만든 것이 아니니까요.

라고 쓰여 있다.

<br/>

위에 언급된 것 처럼 Hibernate는 data class가 자동 생성하는 equals, hashcode, toString 과 잘 맞지 않는다. 내가 실제 겪었던 양방향 관계에서 발생하는 순환참조 문제부터(toString() 때문에 발생하는 StackOverflow), 코틀린이 기본 클래스가 final인 것 때문에 allopen 플러그인을 추가해야하는 점 등 ..

<br/>

-----

이건 잠시 이야기가 새긴 했는데 다시 롬복의 `@Data` 로 돌아오면

위에 열심히 쓴 글의 요지는 쉽게 가면 코가 크게 깨진다는 거다.

<br/>

`@Data` 는 Getter(all fileds), Setter(on all non-final fields), RequiredArgsConstructor, ToString, EqualsAndHashCode 를 생성한다. 

이 엄청난 어노테이션은 바닐라 자바로 코딩할 때에 비해 굉장히 많은 코드를 줄여준다.

쓰지 말자는 이유는 여러 가지 있지만 2가지만 얘기해보면 아래와 같다.

1. 아까 위에 말한 data class의 toString()의 문제와 동일한 순환 참조 이슈가 발생한다.
2. Setter를 사용할 때의 문제점.


-----


일단 자바에서도 자주 언급되지만 `@Setter` 사용은 지양되고 있다. 특히 나처럼 무분별하게 사용한 Setter라면 더더욱 바람직하지 않다.(~~심지어 나처럼 한 클래스가 도메인에 엔티티로 여러 개를 다 하고 있으면 더더더!!)~~

Setter를 지양하자는 의견에는 다음과 같은 이유가 있다.

1. setter가 있으면 아무데서나 객체를 변경할 수 있게 되서 일관성 유지가 힘들다.
2. 실제 의도를 파악하기가 어렵다. (왜 set하는 건데?)

<br/>

어찌됐든 실수를 덜 하고 코드 상 더 알아보기 쉬운 코드를 만들기 위한 것의 일환이다.

setter보다 실제 의도를 코드상에서 확인할 수 있게 메소드를 만드는게 낫다.

<br/>


예를 들어, 유저 정보를 업데이트 한다고 할 때 아래 내가 3개월 전에 작성한 코드를 보자.

```java
public UserEntity update( final String userId, final UserEntity user) {
		
	final Optional<UserEntity> optUserEntity = userRepo.findById(userId);
	
	UserEntity userEntity = optUserEntity.get();
	
	userEntity.setName(user.getName());
	userEntity.setPassword(passwordEncoder.encode(user.getPassword()));
	userEntity.setModifiedTime(LocalDateTime.now());

	return userEntity;
	
}
```

세상에 옵셔널 유저 엔티티를 optUserEntity라고 써놨네 ~~진짜 끔찍하다~~

와 지금 봤는데 저저저저 유저 엔티티 업데이트하는거 저거저거 컨트롤러 단에 updatePassword라고 써두고 서비스 단에는 update라고 쓰고 난리도 아니네.

<br/>

### 수정

<br/>

여튼 조금 수정을 해보자면 지금은 도메인이라고 쓰고 JPA UserEntity 밖에 없는데 User 도메인이 분리되어 있고 그 유저 도메인에 아래와 같이 메서드를 하나 만든다.

```java
public void updatePassword(String password) {
		this.password = password;
}
```
<br/>

그리고 서비스 단에서는 이렇게 패스워드 업데이트하게 넘겨주고 저기에 `@Transactional` 써서 자동으로 더티체킹해서 업데이트해도 되고, 아니면 그냥 userRepository.save(userEntity) 해서 저장해도 된다.

```java
// 1번
@Transactional
public void updatePassword(final String userId, final UserUpdateDTO userUpdateDto) {
	final UserEntity userEntity = userRepository.findById(userId).get();
	userEntity.updatePassword(userUpdateDto.getPassword());
}

------

// 2번
@Transactional
public void updatePassword(final String userId, final UserUpdateDTO userUpdateDto) {
	final UserEntity userEntity = userRepository.findById(userId).get();
	userEntity.updatePassword(userUpdateDto.getPassword());
	userRepository.save(userEntity);
}
```

나는 개인적으로 후자가 더 좋다. 물론 안써도 되겠지만 **뭔가 내 마음이 더 편해!**(JPA 의존적인 게 좋은 건 아니잖아?)

Optitonal이 있어서 못 찾으면 알아서 RuntimeException 날 거고 그럼 알아서 스프링 트랜잭션 기본 정책 상 롤백도 될 거고. 

여튼 `@Data` 를 쓰지 말자 교훈 

→ Setter 대신 해당 데이터를 변경하는 메서드를 도메인에 만들고 거기에 서비스 단은 해당 도메인을 찾아 그 데이터를 넘겨주자.

<br/>

위를 토대로 `@Data` 대신 아래처럼 수정해볼 수 있겠다.

```java
@Builder
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@ToString(exclude = "관계가 있는 테이블(OneToMany etc ...)")
@Getter
@Entity
@Table(name = "Board")
@EqualsAndHashCode(of = {"id"})
public class BoardEntity {
 ...
}
```

JPA에서 프록시를 생성하려면 기본 생성자가 반드시 1개는 필요하다고 한다. JPA 강의 들을 때마다 영한님이 그 소리 자꾸 하시는데 (여러분 JPA에서 기본 생성자는 하나는 꼭 있어야 합니다!)

여튼 만들더라도 굳이 외부에 생성을 열어둘 필요는 없다. 그래서 위처럼 접근 권한을 설정할 수 있다. 

<br/>

위에 달린 `@AllArgsConstructor` 는 아마 `@Builder` 때문에 달았던 걸로 기억하는데 저것도 뭐 필드 다 안 넣으면 컴파일 에러 나고 그랬었다. 근데 3개월 동안 빌더 관련해서 생각해본 적이 없어서 일단은 넣어뒀는데 나중에 자바 공부 좀 더 하고 수정해봐야 겠다.

아 더 봐야 하는데 글 쓰면서 하니 시간이 더 걸리는 거 같다. 

오늘은 그럼 20000!
