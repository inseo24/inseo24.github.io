---
layout: page
title: BigDecimal 클래스를 공부해보자!(precision, scale ...)
permalink: /java/bigDecimal
position: java
---

# BigDecimal의 precision, scale 이 뭘까
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

- 출처

[bigdecimal-precision-and-scale](https://stackoverflow.com/questions/35435691/bigdecimal-precision-and-scale)

<br/>
<br/>

### 시작
시작은 약 2달 전, 처음으로 BigDecimal을 업무상 써야할 시기가 왔다. 쫌쫌 어찌저찌 쓰고 있는데 갑자기 오셔서 BigDecimal을 아냐는 사수님의 질문에 검색은 해봤는데 아직 잘 이해하지 못한 것 같다고 대답했었다. 그 때 막 설명을 해주셨었는데 기억이 정말 1도 안나는 것! ㅎㅎ
2달 전에 어드민에서 사용할 화폐 관리 서비스 개발하면서 쓰긴 썼지만, 오늘 다시 보니 한 번 정리해 봐야 겠다! 하고 노션에 정리했다.

<br/>

### Precision, Scale

- Precision : 전체 자릿수 개수

- Scale: 소수점 오른쪽부터 자릿수 개수

BigDecimal은 유효한 숫자의 역할을 하는 unscaledValue와 32-bit의 scale로 이뤄져 있다. 실제 이 값들이 나타내는 BigDecimal은 unscaledValue * (10^(-scale)) 이다. 즉, scale이 0 이상이면 scale은 소수점 오른쪽 자리의 숫자 개수가 되고, 음수이면 unscaledValue에 10이 (-scale)번 곱해지게 된다.

<br />

BigDecimal 클래스엔 <b>5개의 필드</b>가 있다.

1. intVal - unscaledValue
2. scale
3. precision - 유효한 숫자를 나타내고 처음 계산되기 전까진 0이다. 0이 아닌 계산된 값이라면 precision의 값은 그만큼의 정확성을 보장한다.
4. stringCache - Canonical string representation을 저장하기 위함. 즉, 우리가 일반적으로 숫자를 쓸 때 그 표현문에 클래스 내부에서 저장하고 있는 것으로 추측할 수 있다.
5. intCompact - unscaledValue를 나타냄, 여긴 Long 타입이고 위의 intVal은 BigInteger 타입이다. 표현하고자 하는 숫자의 유효 숫자 개수가 적어 Long으로 나타낼 수 있으면 intCompact에 저장된 값을 사용하고 그렇지 않으면 intVal에 저장후 사용한다.

<br/>

### example

- 123.456 precision: 6, scale: 3
- -93.123 precision: 5, scale: 3
- 0.0 precision: 1, scale: 1
- 10 precision: 2, scale: 0

negative scale : (given number) * 10 ^(-scale value)

- given number = 1234.56
- scale = -5
    - (1234.56) * 10 ^(-(-5))
    - (1234.56) * 10 ^ (+5)
    - 123456000
    
<br/>

### 테스트 해보자!

스택오버플로우 검색하고 위의 예시를 쭉 봤는데 한 번 테스트를 해볼까?! 하고 마음이 들었다.

BigDecimal을 한 번 인텔리제이에서 찍고 디버거를 돌려서 필드 값을 확인해보자!

위에 나온 예시들과 잘 맞는 걸 확인 할 수 있다.

<br />
<img width="310" alt="image" src="https://user-images.githubusercontent.com/84627144/160070759-09d6072c-fc85-4d87-a257-b2eb1a35aad1.png">
<br />

위에 숫자들은 Long에 담기는 크기가 intVal이 모두 null 상태이고 intCompact에 담겨있다.

BigDecimal(BigInteger()) 로 숫자를 담으면 이 값이 intVal에 담기는 것을 확인할 수 있다.

그런데 intCompact에 담긴 저 마이너스 값은 뭘까 → 당연히 Long으로 표현할 수 없어서 그런 거라고 생각이 들긴 하는데, 음 이건 아마도 unscaledValue 값으로 바뀐거라 생각하면 되겠지?

<br />
<img width="444" alt="image" src="https://user-images.githubusercontent.com/84627144/160070776-3ce0643d-d58c-4985-9701-030f12a98343.png">
<br />

위의 두 예시에선 BigDecimal() 안에 String으로 입력했는데 이걸 Double형으로 넣으면 우리가 생각하는 정확한 소수 표현이 되지 않는다. 부동 소수점 방식을 배우다 보면 오차가 발생함을 알 수 있는데, 보통 그 값과 가장 근사한 값을 반환하다 보니 부정확한 값이 리턴될 수 있는 위험이 있다. (보통 나눴을 때 무한히 자릿수가 발생하는 무한 소수, 순환 소수들을 컴퓨터에서 정확히 처리하기 어려울 때 등등)

이걸 한 번 테스트해보자!

<br/>
<img width="722" alt="image" src="https://user-images.githubusercontent.com/84627144/160070799-867a3515-feaf-4160-b57d-df805691c0bc.png">
<br/>

우와 아니나 다를까 double로 표현한 0.1에서 불필요한 뒷자리수들이 붙는 걸 확인할 수 있다. 얘는 유효숫자가 많아서 값이 intVal에 담기는 것도 확인할 수 있다.

문자열로 넘긴 숫자가 우리가 생각한 그대로 표현할 수 있고 값도 intCompact 안에 잘 담겨있다.

위에서도 볼 수 있지만 String으로 BigDecimal을 생성하면 precision이 잘 계산되어 0이 아님을 확인할 수 있다. 아마도 String을 숫자로 변환할 때 하나하나 숫자를 읽어서 계산할 것이니 precision 계산이 더 정확하게 되는 듯 싶다.

<br/>
<img width="285" alt="image" src="https://user-images.githubusercontent.com/84627144/160070818-c9372aec-647f-4d5f-af65-d4516f5a57af.png">
<br/>

하다가 또 하나 위에 봤던 개념을 확인할 수 있었는데, 전체 자리수라는 precision은 0의 개수를 카운트하지 않는다는 점이다. 0.01은 1개, 0.001도 1개, 0.011은 2개임을 확인할 수 있었다.

재미가 있었다~!