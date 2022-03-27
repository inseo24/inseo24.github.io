---
layout: page
title: 🍖 뜯어보기 1일차 - build.gradle(feat. Entity vs Domain)
permalink: /kotlin/study-plan/sirloin/1
position: kotlin
nav_order: 3
parent: Kotlin study plan
---


# 🍖 뜯어보기 1일차 - 시작은 build.gradle 부터?(feat. Entity vs Domain)

### 시작

일단 트위터에 오픈 소스 뜯어보겠습니다! 설로인부터 할 거에요! 라고 질러서 오늘 설로인 소스 코드를 다운 받았다. 멍청하게 서버 파일 인텔리에 한 꺼번에 올려서 전체 다 다운 받는데만 20분 걸렸다.

**~~내 노트북이 문제인가?~~**

여튼 모르는 코드를 볼 때 build.gradle 부터 보는게 습관이라 여기부터 시작했다.

시작부터 `Distributed under CC BY-NC-SA` 라는 문구가 눈에 띄었는데 당연히 몰랐기 때문에 검색을 해봤더니 라이센스 관련된 문구였다. 오 뭔가 오픈 소스는 이렇게 꼼꼼해야 하는 건가 싶었다.

- This license lets others remix, tweak, and build upon your work non-commercially, as long as they credit you and license their new creations under the identical terms.
- 이 라이센스는 상업 목적이 아니면 다른 사람이 수정해도 상관 없다~ 라는 거 같다. 동일 조건이면 새로운 창작물에도 라이센스를 따로 부여할 수 있다는 건가? 여튼 [출처](https://guides.lib.umich.edu/creativecommons/licenses) 에서 더 자세한 건 보세요.

 

<img width="161" alt="image" src="[https://user-images.githubusercontent.com/84627144/160265514-d58187c6-aaad-4ca2-b7d8-5225cb38e1af.png](https://user-images.githubusercontent.com/84627144/160265514-d58187c6-aaad-4ca2-b7d8-5225cb38e1af.png)">

---

그다음 보게 된 것은 jvm 17과 kotlin version 1.6.10 이라는 건데 1.6?

갑자기 나는 몇 버전 쓰지 하고 궁금해져서 내 코드 보다가 젯브레인 코틀린 블로그를 갔는데 1.6.0 릴리즈가 2021년 12월 중순이다. 아직 3개월 밖에 안지난 거 같은데 엄청 최신 버전 쓰네? 싶었다. 그리고 좀 더 생각해보니 아 이거 인텔리제이 기본으로 만들면 코틀린 최신 버전으로 되고 java 17 설정해서 이렇게 나왔나 싶었다.(왜냐면 내가 최근에 만든게 

근데 검색해보면서 젯브레인 블로그 처음 들어가봤는데 이것도 각잡고 정리하면서 공부하는 글 쓰면 엄청난 공부가 될 거 같다. 코틀린 정보는 다 여기 있었구나!!

1.6.10 에서 다른 건 잘 모르겠고 Kover라는게 생겼다고 한다. 코드 커버리지를 측정하는 gradle plugin이라는데 기존에 jacoco가 gradle toolchain, 멀티 플랫폼 프로젝트에서는 완전히 통합되지 않은 이슈를 픽스하기 위해서 만들었다고 한다. 아직 개발 초기 단계구나. 

Kotlin + Coverage = Kover 인건가!

---

이거 2개도 처음 본다. 

`version_javax_inject = "1"`

얘는 maven repository 공식 홈페이지 가보니 구글에서 만든거 같고 자동으로 의존성 주입해주는 건가 보다. 회사 코드 어디에 쓰였는지 나중에 봐봐야지. 내가 검색해서 알아내는 것 보다 빠르겠지 

나는 처음 뵙는 라이브러리인데 구아바, 스프링 빈즈에 쓰이신 건지 Artifacts using Javax Inject 치니 여러가지가 나오는 구만

> JSR-330: Dependency Injection for Java
Welcome to the home of JSR-330: Dependency Injection for Java.
> 

<img width="255" alt="image" src="[https://user-images.githubusercontent.com/84627144/160266345-31d4b1de-81dc-4832-9024-643823df9803.png](https://user-images.githubusercontent.com/84627144/160266345-31d4b1de-81dc-4832-9024-643823df9803.png)">

**음 뭔가 단출해서(?) 좋군요!**

`version_jsr305 = "3.0.2"`

jsr305도 처음 보는데 컴파일 시점에 정적으로 버그 잡아 준다고 하는데 이거 어디서 쓰였을까 훔훔

<img width="348" alt="image" src="[https://user-images.githubusercontent.com/84627144/160266378-744b65ad-0dc9-461b-8b03-5851945e4dd4.png](https://user-images.githubusercontent.com/84627144/160266378-744b65ad-0dc9-461b-8b03-5851945e4dd4.png)">

오호 어노테이션이 javax.validation에서 보던 것과 비슷한게 보이는데 NonNull 같은 거.

이거 컴파일 할 때 저기 어노테이션 붙은 것들 데이터 의도한대로 담고 있는지 다 체크해주나 보다. 이거는 대충 봐도 꽤 좋은 거 같다!! 나중에 사용하는 예시 좀 보고 개인 프로젝트에 써봐야지.

---

오늘은 여기까지 하고 다른 거 좀 뒤져볼까~? 하다가 재밌는 걸 발견했다.

재밌게 이런 질문이 있네!!!

```
// POINT: 이 클래스를 data class 로 선언하고, equals / hashCode 를 삭제해도 문제가 없을까요?
// POINT: Entity 와 Domain Model 의 차이란 뭘까요?
```

api-core-infra-impl 쪽 domain.user 디렉토리 안에 UserEntity 위에 달린 재밌는 주석이 달려 있었다!

infra라고 써있는게 JPA 같은거나 외부 서비스, message 관련인가 싶긴 했는데 domain.user 밑 repository가 있었다. 디렉토리 구조가 뭔가 신기한데 저렇게 도메인별로 구분하는 건가? 

일단 첫 번째 포인트에 대한 생각을 적어보자면(코틀린 3개월차라 틀릴 수 있음)

1. data class 생성하면 알아서 equals, hashCode 같은거 생성되서 상관은 없는데
2. 문제가 저게 JPA랑 같이 쓸 때 에러가 꽤 뿜뿜 했다. 양방향 맵핑 했을 때 toString() 때문에 StackOverFlow 도 뜨고, 뭐 이런저런...
3. 결론은 삭제해도 문제는 없으나 data class를 엔티티로 쓸 때 발생할 수 있는 side effect가 있어서 꼭 적합한지는 잘 모르겠다!
4. 근데 지금 보니까 이 클래스 JPA로 만든 Entity가 아니구나!
5. equals랑 hashCode 자동 생성해서 비교해볼까.

```kotlin
// 이게 원래 있던 코드
override fun equals(other: Any?): Boolean =
  if (other !is UserEntity) {
      false
  } else {
      Objects.equals(id, other.id) &&
      Objects.equals(uuid, other.uuid) &&
      Objects.equals(nickname, other.nickname) &&
      Objects.equals(profileImageUrl, other.profileImageUrl) &&
      Objects.equals(deletedAt, other.deletedAt) &&
      Objects.equals(createdAt, other.createdAt) &&
      Objects.equals(updatedAt, other.updatedAt) &&
      Objects.equals(version, other.version)
  }
        
// 이게 인텔리제이 기능으로 만든 거!
override fun equals(other: Any?): Boolean {
    if (this === other) return true
    if (javaClass != other?.javaClass) return false

    other as UserEntity

    if (id != other.id) return false
    if (uuid != other.uuid) return false
    if (deletedAt != other.deletedAt) return false
    if (createdAt != other.createdAt) return false
    if (version != other.version) return false
    if (nickname != other.nickname) return false
    if (profileImageUrl != other.profileImageUrl) return false
    if (updatedAt != other.updatedAt) return false

    return true
}
```

음 자동 생성해서 만든 코드가 아니었네. !is 를 쓴 코드 처음 본다.

Object의 equals를 써서 값을 하나하나 비교하는구나.

Object 안에 equals 타고 들어가니 아래 코드가 나오는데 이렇게 보니까 같은 코드를 가독성 있게 다시 썼구나 싶다.

```java
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```

```kotlin
// 작성되어 있던 코드
override fun hashCode(): Int {
        return Objects.hash(
            this.id,
            this.uuid,
            this.nickname,
            this.profileImageUrl,
            this.deletedAt,
            this.createdAt,
            this.updatedAt,
            this.version
        )
    }

// 인텔리 제이 기능으로 자동 생성한 코드
override fun hashCode(): Int {
    var result = id?.hashCode() ?: 0
    result = 31 * result + uuid.hashCode()
    result = 31 * result + (deletedAt?.hashCode() ?: 0)
    result = 31 * result + createdAt.hashCode()
    result = 31 * result + version.hashCode()
    result = 31 * result + nickname.hashCode()
    result = 31 * result + profileImageUrl.hashCode()
    result = 31 * result + updatedAt.hashCode()
    return result
}
```

hashCode 도 그렇네! 얘도 Object의 hash를 써서 따로 작성한 코드였다.

자동생성된 코드 result 따닥따닥 써 있어서 진짜 꼴보기 싫네

Object.hash로 한 번에 묶으니까 엄청 깔끔하다. 워후

실제 Object 안에 있는 hash()의 hashCode()를 타고 들어가니 result 똑같이 써주는 거 보니 기능은 동일하구나. 나 hashCode 구현해본 적 없는데 흠믐

일단 생각에는 data class는 hashCode(), equals(), toString() 등을 자동으로 생성해주니 문제는 없지 않을까 싶다. 

---

2번째 포인트는 엔티티와 도메인의 차이에 대한 점인데.

헥사고날 아키텍처를 학습하면서 공부했던 부분이라 음 정리해보자면.

도메인은 순수 비즈니스 로직만을 표현하고 캡슐화되어서 기능적인 요구사항에 맞게 설계를 한다고 보면 되고

엔티티는 외부의 인프라에 해당해서 내부 영역인 도메인과 분리해서 구성한다. 

최근에 끝난 운영체제 스터디에서 나보다 선배 개발자분들이 JPA 쓰기 싫다 꼴도 보기 싫다 이런 얘기 엄청 하셨는데 결국 DB에 의존적이게 되고, JPA에 의존적이게 되서 그 부분에서 발생하는 이슈에 진저리(?)가 나신 거 같았다.

여튼 헥사고날은 내부와 외부를 나눠서 비즈니스 로직이 표현 로직이나 데이터 접근에 의존하지 않게 만드는게 포인트라고 할 수 있는데 여기서 내부에 해당하는게 도메인! 외부에 해당하는게 엔티티라고 보면 될 거 같다!

---

아 User 엔티티 Editor 타고 갔다가 또 질문 발견했다. 이건 내일 하는 걸로...

`// POINT: 왜 User 도메인 모델을 mutable 로 설계하지 않고, 이런 타입을 별도로 만들었을까요?`

### 오늘 알 게 된 것

1. 새로 본 라이브러리들!
    1. javax_inject
    2. jsr305
    3. kover(이건 코틀린 블로그에서...)
2. 라이선스 정책
    
    `CC BY-NC-SA` - 나중에 내가 이걸 직접 타이핑할 날이 올까?
    
3. 설로인 코드에는 질문이 산다!(그것도 엄청 많이!!)
    
    

### 다음에 할 것

1. User Domain 모델을 뮤터블하게 설계하면 당연히 아무데서나 아무 개발자가 이 클래스를 참조해서 바꿀 수 있으니 코드가 개판될 가능성이 높아지겠지? 그니까 도메인에서 fun을 따로 만들어서 필드를 수정하거나 아니면 여기 있는 코드처럼 인터페이스를 만들 수도 있겠다. 이렇게 인터페이스 만든거 처음 보는데 인터페이스를 통해서 수정하게 만드는 것도 코드 분리하는 면에서도 좋고 명세로서도 좋을 거 같긴 하다. 

1. 아 그리고 설로인에 jvm lib가 따로 있어서 그게 궁금해서 내일 볼까 싶다. 흠믐 
    
    원래 오늘 그거 보려했는데 뜬금 없이 질문 발견해서 시간이 많이 지나버렸네.
    

### 결론

오늘도 재미가 있었다~!

(real mysql 1권 읽기 싫다!!)