---
layout: page
title: 모나드야 모나드야! (Monad, Functor) - 1
permalink: /etc/monad
position: etc
---


# Monad 모나드야 모나드야! 1탄

<br />

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

<br/>

### 시작
<br />

시작은 이랬습니다. 운영체제 스터디를 하던 중 한 분이 모나드를 이해한 거 같다면서 얘기를 했습니다.

그 때 다들 반응이 모나드가 뭐야?👀👀 웅성웅성한 느낌이었는데요.

설명을 듣고도 저는 이해를 못했고 회사 와서 사수님한테 물어봤을 때도 잘 이해를 못했습니다.

그러던 중 스터디가 끝나고 몇 일 후...

카톡 방에 전달된 한 유튜브 [영상](https://www.youtube.com/watch?v=jI4aMyqvpfQ) !!

**Monad란 무엇인가?** 라는 제목의 영상이었습니다. 

이 영상을 보고 제가 지금까지 열심히 쓰고 있던 map()에 대해 완전히 잘못 알고 있었다는 것을 깨달았습니다.

<br />


### 이 영상을 보기 전에
<br />

저는 함수형 프로그래밍을 모르는 사람입니다. 알려고 노력도 아직 안해봤습니다.(~~객체지향도 이해 못했다구욧!!~~)

map에 대한 개념이라고는, 배열을 넣고 그 배열에 조작을 한 후 새로운 배열로 리턴하는구나! 정도의 생각만 갖고 있었습니다. 

영상 속에 언급된 것처럼 ~~아! for 문을 옆으로 쓰는 구나!~~ 급의 이해도를 갖고 있었습니다.

~~자매품: flatMap은 map을 flat하게 만들어주는구나!~~


<br />

### 영상 속에서는
<br />

생각치도 못한 Promise 개념이 나와서 놀랬습니다. Optional이 모나드 개념에서 파생됬다는 건 스터디원님에게 들어서 알았던 것이지만 자바스크립트의 그 프로미스?!!도 관련된 개념이었어?

생각보다 모나드라는 개념은 이미 많은 곳에 적용되어 있었습니다. 

영상에선 이렇게 설명합니다.

Monad는 값을 담는 컨테이너의 일종이며 Functor를 기반으로 구현되어 있다.

flatMap() 메소드를 제공하고 Monad Laws를 만족시키는 구현체를 말한다.

여기까진 **엥?** 할 수 있지만 영상을 읽고 제가 이해한대로 글을 한 번 쪄보겠습니다.

<br />


### 일단은 Functor가 뭔데?

```java
import java.util.function.Function;

interface Functor<T> {
    <R> Functor<R> map(Function<T, R> f);
}
```

Functor는 위와 같이 생긴 인터페이스입니다.

- Functor는 항상 불변형의 값을 담은 컨테이너입니다. → 그래서 map은 원본 객체를 절대 변경하지 않습니다
- 함수를 매개변수로 받는 map 메소드만 갖고 있습니다.
- 이 map의 메소드 매개변수로 함수를 받는데, 해당 함수는 T 타입을 받고 R 타입을 반환합니다.
- Functor는 map 메소드의 매개변수로 전달한 함수의 결과를 새로운 Functor 객체에 감싸서 반환합니다.

<br />

결국 이 Functor의 핵심은 타입 안정성을 유지하면서 값을 변형시킬 수 있다는 점입니다. 

<br />

<strong>그럼 왜 Functor를 사용하나?</strong>

영상에서는 Functor를 이용할 때의 장점을 예시로 들어서 말해주고 있는데요.

- 첫 번째, 값이 없는 케이스 → null 이 생각나시죠?

- 두 번째, 값이 미래에 준비될 것으로 예상되는 케이스 → 이건 비동기?!

위의 상황과 같이 일반적으로 모델링할 수 없는 상황을 모델링할 수 있다는게 Functor의 가장 큰 사용 이유라고 할 수 있겠습니다.


<br />

java에서 Optional이 모나드의 예시라고 위에 잠깐 언급을 했는데요. 

첫 번째, 값이 없는 케이스에서 Functor를 잠깐 보겠습니다.

옵셔널을 보면 코드가 이렇습니다.

```java
class FOptional<T> implements Functor<T, FOptional<?>> {

    private final T valueOrNull; // T 값을 하나 갖고

    private FOptional(T valueOrNull) {
        this.valueOrNull = valueOrNull;
    }

    public <R> FOptional<R> map(Function<T, R> f) {
        if (valueOrNull == null) // 값이 비어있으면 empty() 호출, f 함수 호출 안함
            return empty();
        else
            return of(f.apply(valueOrNull));
    }

    public static <T> FOptional<T> of(T a) {
        return new FOptional<T>(a);
    }

    public static <T> FOptional<T> empty() {
        return new FOptional<T>(null); // 값이 비어있는 경우 null을 값으로 가진 Functor를 반환
    }
}
```
<br />

위의 옵셔널을 보면 Functor는 값을 가질수도 갖고 있지 않을 수도 있습니다. 슈뢰딩거의 고양이가 생각나죠?

타입안정성은 유지한 채로 null을 처리할 수 있게 된 것이죠!

옵셔널을 사용하는 쪽에서는 null check가 불필요합니다. 

null인 경우에는 그냥 로직이 실행되지 않게 되는 거죠.

<br />


```java
FOptional<String> optionStr = FOptional(null);
FOptional<Integer> optionInt = optionStr.map(Integer::parseInt);

-----------------------------------------------------------------

FOptional<String> optionStr = FOptional("1");
FOptional<Integer> optionInt = optionStr.map(Integer::parseInt);
```

<br />

이걸 두고 타입안정성을 유지하면서 null을 인코딩할 수 있다고 표현합니다.

이미 써본 분들이 대부분이겠지만 Functor의 개념을 알고 생각하니 색다르지 않습니까?!!


<br />


그렇다면 값이 미래에 준비되는 케이스는 어떨까요?

Promise Functor를 살펴봅시다.

Promise<T>는 값이 어떻게 준비되든 그 속사정을 모르더라도 Functor로 아래와 같이 사용할 수 있습니다.

```java
Promise<Customer> customer = // ...
Promise<byte[]> bytes = customer
        .map(Customer::getAddress)
        .map(Address::street)
        .map((String s) -> s.substring(0, 3))
        .map(String::toLowerCase)
        .map(String::getBytes);
```

다들 익숙하시죠?! 

<br />

Promise<Customer>는 아직 값을 갖고 있지 않고 미래에 준비될 것을 약속하기만 합니다. 
하지만 아직 값이 없음에도 여기에 map을 적용할 수 있다는 것이죠. Promise에 값이 준비될 때까지 기다리지 않고 대신 다른 타입의 새로운 Promise를 반환합니다. 
나중에 원래의 Promise의 값이 준비되면 새 Promise는 map()의 인자로 전달된 함수를 적용하고 그 결과를 전달합니다. 
이렇게 비동기가 처리되는 것이죠.(non-blocking)

또 비동기 연산들의 합성 또한 가능해지고요!

<br />

### 그럼 모나드는...?

그럼 이제 모나드가 뭐냐?

모나드는 <strong>Functor에 flatMap()을 추가한</strong> 것입니다. 

글을 한 번에 쓰고 싶었는데 아직 뇌가 좀 쉬고 싶은 관계로 이 글은 내일 더 찌도록 하겠습니다. 

그럼 20000!