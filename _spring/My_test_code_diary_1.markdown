---
layout: post
title: Test Code 작성기 1
permalink: /spring/test_code/1
position: spring
parent: Spring
nav_order: 1
---

# Test Code 작성기 1 - 어노테이션, Controller, Repository

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

막 입사한 12월 피나는 코드리뷰를 받고 한 번도 써본 적도 없는 테스트 코드를 작성할 걸 과제로 받았다!

테스트 코드를 다들 짠다더라~ 하는 얘기만 들었지 직접 해본 적이 없었기 때문에 어떻게 공부를 해야할 지 고민을 많이 했던 것 같다.

프로그래밍 공부는 역시 인도 영어로 시작하는 거 아니겠어? 하고 유데미에서 아래 강의를 구매해 들었다.

<br />

<img width="977" alt="image" src="https://user-images.githubusercontent.com/84627144/176341651-8a1b01bf-a112-472b-b2a4-7475a37f30c1.png">

<br />

이 강의를 선택한 이유는 다른 건 아니고 유튜브 떠돌다가 발견했는데 유데미 링크까지 친절하게 있길래 괜찮다 싶어서 들었다. 이번 테스트 작성기 1에선 여기 강의에서 들은 내용을 오랜만에 정리해볼까 한다. 

<br />

### 어노테이션

- `@Test` - 테스트 메서드를 나타냄, 어떤 속성도 선언 x
- Lifecycle 어노테이션
    - `@BeforeAll`  `@AfterAll` → 1번만
    - `@BeforeEach` `@AfterEach` → 1 or more
- `@Disabled` - 테스트 하지 않는 것 표시
- `@DisplayName` - 테스트 표현
- `@RepeatedTest` - 특정 테스트 반복
- `@ParameterizedTest` - 여러 다른 매개변수를 대입해가면서 반복 실행
    - `@CsvSource(value = ....)`
- `@Nested` - 테스트 클래스 안에 내부 클래스를 정의
- Assertion - 수행 결과 판별
    - static
- AssertAll - 끝까지 실행
- assertThrows(expectedType, executable) - 예외 발생 확인 테스트
- assertTimeout(duration, executable) -  특정 시간 안에 끝나는지
- Assumption - 전제문이 true면 실행, false면 종료
    - assumeTrue - false 일 때, 테스트 전체가 실행되지 않음
    - assumingThat
    

<br />

### Controller

```kotlin
@WebMvcTest(BoardController::class)
internal class BoardControllerTest {
```

<br />

처음 테스트 코드를 봤을 때, internal 키워드를 붙여야 하는지 아닌지 고민을 했던 거 같다. 

이게 인텔리제이 기본으로 generate test로 생성하면 internal이 붙은 클래스가 나와서 붙이긴 했는데 솔직히 멀티 모듈 프로젝트 아니면 상관없지 않나 싶긴 하다.internal이 자바에는 없어서 public으로 바뀌니까. 하지만 어차피 기본으로 넣어주는 거 냅두도록 하자.

(혹시 멀티 모듈일 경우 다른 모듈에서 호출이 안될 수 있다는 것만 알아두면 되겠지?)

<br />

1. `@WebMvcTest`
    1. controller 테스트를 위해 보통 사용한다.
    2. 웹상에서 요청과 응답에 대해 테스트할 수 있다.
    3. 시큐리티, 필터까지 자동으로 테스트하며, 수동으로 추가/삭제가 가능하다.
    4. `@SpringBootTest` 어노테이션보다 가볍게 테스트 할 수 있다.
    5. `@Controller` `@ControllerAdivce` `@JsonComponent` `@Converter` `@GenericConverter` `@Filter` `@HandleIntercepter` 등의 내용만 스캔하도록 제한한다.
    
2. `@MockMvc`
    1. 웹 애플리케이션을 서버에 배포하지 않고도 스프링 MVC 동작을 재현할 수 있는 클래스
    
3. LocalDateTime 은 추가 라이브러리가 필요함 → 직렬화 이슈
    
    ```kotlin
    아래 라이브러리 디펜던시에 추가
    // https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-jsr310
    implementation("com.fasterxml.jackson.datatype:jackson-datatype-jsr310:2.12.3")
    ```
    
    ```kotlin
    val startDate = LocalDateTime.of(2022, 1, 7, 11, 19)
    val endDate = LocalDateTime.of(2022, 1, 8, 19, 19)
    ```
    
    ```kotlin
    content = jacksonObjectMapper().registerModule(JavaTimeModule()).writeValueAsString(request)
    ```
    
4. 예시 코드
    
    ```kotlin
    @WebMvcTest(BoardController::class)
    internal class BoardControllerTest {
    
        @Autowired
        lateinit var mockMvc: MockMvc
    
        @MockBean
        lateinit var boardService: BoardService
    
        @Test
        @DisplayName("게시글을 생성한다")
        fun create_board() {
            val request = CreateBoardRequest("name", "title")
    
            mockMvc.post("/board") {
                contentType = MediaType.APPLICATION_JSON
                content = jacksonObjectMapper().writeValueAsString(request)
            }.andExpect {
                status { isOk() }
            }
        }
    }
    ```
    
<br />

### Repository

```kotlin
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
```

<br />

1. `@DataJpaTest`
    1. JPA 관련 테스트 설정만 로드한다.
    2. DataSource의 설정이 정상적인지, JPA를 사용해 데이터를 제대로 생성, 수정, 삭제하는지 등의 테스트가 가능하다.
    3. 가장 좋은 점은 내장형 데이터베이스를 사용해 실제 데이터베이스를 사용하지 않고 테스트 데이터베이스로 테스트할 수 있다.
    4. 기본적으로 @Entity 어노테이션이 적용된 클래스를 스캔해 스프링 데이터 JPA 저장소를 구성함. 만약 최적화된 별도의 데이터 소스를 사용해 테스트하고 싶다면 기본 설정된 데이터소스를 사용하지 않도록 아래와 같이 설정해도 된다. 
        
        ⇒ `@AutoConfigureTestDatabase(replace = AutoConfigureTest.Replace.NONE)`
        
    5. `@AutoConfigureTestDatabase` 의 기본 설정값인 Replace.Any를 사용하면 기본적으로 내장된 임베디드 데이터베이스를 사용한다. 위의 코드에서 Replace.NONE으로 설정하면 `@ActiveProfiles` 에 설정한 프로파일 환경값에 따라 데이터 소스가 적용된다. yml 파일에서 프로퍼티 설정을 spring.test.database.replace : NONE 으로 변경하면 됨
    6. `@DataJpaTest` 는 기본적으로 `@Transactional` 을 포함하고 있다. 테스트가 완료되면 자동으로 롤백하기 때문에 직접 선언적 트랜잭션 어노테이션을 달아줄 필요가 없다. 
    
2. 예시 코드
    
    ```kotlin
    @DataJpaTest
    @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
    internal class BoardRepositoryTest {
    
        @Autowired
        lateinit var boardRepository: BoardRepository
    
        @Test
        @DisplayName("Board Entity를 저장한다.")
        fun save() {
            val boardJpaEntity = BoardJpaEntity("name", "title")
            val savedEntity = boardRepository.save(boardJpaEntity)
    
            assertThat(savedEntity.name).isEqualTo("name")
            assertThat(savedEntity.title).isEqualTo("title")
        }
    }
    ```
    
<br />

### 궁금했던 거

1. mock() ← 이거 대신 `@Mock` 을 쓰는 이유 : 안 그럼 계속 반복 호출해줘야 하니까.
    
    
2. mocked된 오브젝트에 의존성을 주입하고 싶을 때는, `@InjectMocks` 로 의존성 주입이 필요한 대상에 붙이고, 필요한 의존성들은 `@Mock` 을 이용해 주입한다.
    
    
3. 왜 Mock 객체를 사용할까?
    
    테스트가 다른 부분에 의존할 수록 테스트 상황 때문에 다른 걸 모두 초기화해야 할 상황에 직면할 수도 있다.
    
    고로 이런 의존 관계를 없애기 위해 mock을 사용
    
4. 뭘 테스트 하지?
    
    경험이 있다면 망가지기 쉬운 것들만 골라서 테스트하면 될 테지만, 난 그런 경험이 없으니... 
    
    - **Right** → 결과가 옳을까?
    - **Boundary** → 모든 경계 조건이 correct할까? 의도한대로 동작했는가?
        
        CORRECT
        
        Conformance, Ordering, Range, Reference, Existence, Cardinality, Time
        
        엉터리 입력 값을 넣어보기, 혹은 null, “”, 0, 중복값, 순서 바꿔서 넣어보거나 ...
        
        ex) age → 10000 ?
        
    - **Inverse**
        
        역관계가 확인 가능한가?
        
    - **Cross-check**
        
        다른 수단을 사용해 결과를 교차 확인할 수 있나?
        
    - **Error**
        
        error condition을 강제로 만들어낼 수 있나? 
        
        ex) 메모리 고갈, 디스크 공간 고갈, 시스템 부하 ...
        
    - **Performance**
        
        performance 특성이 한도 내에 있는지?
        
5. 테스트 하는 이유
    - 회귀 방지 ↔ 빠른 피드백
    - 안정감
    - 다른 개발자들에게 코드 흐름을 보이기 쉬움
    - **설계를 테스트 할 것**
    
6. 테스트 팁
    - private 메서드는 테스트 하지마라
    - 도메인 지식을 테스트로 유출하지마라
        - 제품 코드와의 결합도가 높아짐 → 최종 결과값은(isEqualTo 같은 건) 값이나 값 객체(enum class 등)를 사용할 것.
    - 테스트를 위한 코드를 제품에 추가 하지 마라

<br />

### test 하기 힘든 것

Non-testable → 의도한 전략대로 반환

- random, shuffles, LocalDateTime.now()
- 외부 세계 - HTTP, 외부 저장소

<br />

### Stub

테스트하는 대상이 의존하는 요소의 동작을 시뮬레이션한다.

실제 코드와 준비되지 않은 코드를 미리 정해진 답변으로 가장한다.

<br />

### Why we use @Mock

- 이미 검증된 라이브러리 클래스를 호출하는 것은 문제가 되지 않음
- 검증 안 된 다른 팀원이 구현한 클래스를 호출하는 것은 문제가 된다!
- 예를 들어, mapper 객체는 service를 테스트 할 때 `@Autowired` 되야 한다. → mapper 를 남이 구현한 상태에서 아직 완성되지 않은 경우 어떻게 테스트할까?
    - 내가 구현한 service 로직 테스트 할 때 mapperfmf 최소한의 기능만 하는 mock 객체로 만들자



<br>

일단 여기까지 쓰고 다음 번에 가장 고민이 많았던 Service 로직 테스트 코드 도전기에 써봐야겠다. 

쓸 거 진짜 많네.