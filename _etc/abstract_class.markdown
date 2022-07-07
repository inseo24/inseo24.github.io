---
layout: post
title: 추상 클래스는 언제 사용할까(abstract class)
permalink: /etc/abstract/class
position: etc
parent: etc
nav_order: 8
---


<br/>

# 추상 클래스(abstract class)

출처 : [https://khalilstemmler.com/blogs/typescript/abstract-class/](https://khalilstemmler.com/blogs/typescript/abstract-class/)

이 페이지에서 추상화가 무엇이고, 추상 클래스는 무엇이고, 실제 프로젝트에서 사용할만한 예시를 살펴보자.

## 주의

이 글은 출처에 있는 글을 발번역한 글입니다. 참고해주세요.

<br/>


### 추상화는 무엇일까요?

일반적으로 말하자면, 추상화는 복잡도를 줄이는 기술입니다. 더 나은 아키텍처를 위해 문제를 해결하는 데 도움을 주기도 합니다. 추상화는 로우 레벨의 세부 사항을 미리 구현할 필요성을 없애고 하이 레벨에 집중할 수 있게 도와줍니다.(나중에 세부사항을 신경쓸 수 있게끔)

또한 추상화를 통해 선언(우리가 원하는 것을 요구하는 데)에 더 많은 시간을 쓸 수 있고, 더 필수적인 사항에 시간을 할애할 수 있습니다.

<br/>

### Concretion & Abstractions

우리는 객체지향 프로그래밍에서 구체화된 클래스와 추상화된 클래스를 다룹니다.

![https://khalilstemmler.com/img/blog/typescript/abstract-class/abstraction-vs-concretion.svg](https://khalilstemmler.com/img/blog/typescript/abstract-class/abstraction-vs-concretion.svg)

구체화된 클래스는 실제적이고 런타임에 사용하기 위해 new 키워드로 객체를 생성할 수 있습니다. 

우리가 마주치는 대부분의 클래스가 Concrete class에 해당합니다.

```tsx
//Concrete class
class User {
  private name: string;

  public getName (): string {
    return this.name;
  }

  constructor (name: string) {
    this.name = name;
  }
}

const user = new User('Khalil'); // Creating an instance 
                                 // of a concrete class`
```

반면 추상화는 구체화가 가져야 할 프로퍼티나 메서드를 정하는 blueprints나 contracts에 해당합니다. 우리는 추상화를 이용해 객체나 클래스의 구조를 계약화할 수 있습니다.(미리 계약서를 쓰는 것처럼 정한다는 느낌인듯)

```tsx
// Interface (abstraction)
interface Box {
  length: number;
  width: number;
}

const boxOne: Box = { length: 1, width: 2 }; // ✅ Valid! Has all props
const boxTwo: Box = { length: 1 };           // ❌ Not valid, missing prop`

// Interface (abstraction)
interface Box {
  length: number;
  width: number;
}

// Concrete class implementing Box abstraction
class MobileBox implements Box { // ✅ Valid! Implements all necessary props
  public length: number;  
  public width: number;

  constructor (length: number, width: number) {
    this.length = length;
    this.width = width;
  }
}

let boxThree = new MobileBox(1, 2);`
```

<br/>

### 객체 지향 언어에서 어떻게 추상화를 만들까?

전의 예시를 보면, 인터페이스 키워드를 사용해서 추상화를 만들었다. 보통 객체 지향에서 추상화를 얘기할 때, 2가지 도구를 사용한다. 

1. 인터페이스(interface 키워드 사용)
2. 추상 클래스(abstract 키워드 사용)

추상화 도구로써의 type system: 타입스크립트 같은 type system과 함께 언어를 사용해서 타입을 이용해 추상화를 구현할 수 있다.

<br/>

### 추상화는 왜 필요할까?

추상하과 중요한 이유는 여러 이유가 있겠지만 핵심은 하이 레벨과 로우 레벨을 분리해 플러그인 아키텍터를 만들 수 있다는 점이다. 

전통적인 프로그래밍에서 우리는 동적 바인딩을 안전하게 구현할 수 없었다. - 추상화를 참조하고 dependency들이 알아서 코드와 함께 돌아갈 것이라 기대하기 어려웠다.

즉, 확장이 어렵다는 점이다. 새로운 타입이 추가된다면 어떻게 해야할까?

시간이 지날수록 추상화 없이 새로운 코드를 추가하는 것은 복잡성을 계속 증가시킬 것이다. 프로그램이 서브루틴의 서브루틴을 타며 거대하게 증가해 나가는 걸 볼 수 있을 거다.

![https://khalilstemmler.com/img/blog/typescript/abstract-class/program-flow.png](https://khalilstemmler.com/img/blog/typescript/abstract-class/program-flow.png)

이 dependency 관계를 뒤집을 수 있는 방법이 있다면 어떨까? 

먼저 제공해야 하는 모든 것을 포함한 계약을 정의하고, 후에 개발자에게 구현하도록 맡기면 어떨까?

구현(결정) 대신 추상화에 대해 프로그래밍할 수 있다면 어떨까?

port, adapters가 바로 이걸 이해할 수 있는 가장 간단한 방법일 거다. 추상화를 유효한 adapter를 정의하는 port라고 생각해보자. 이렇게 디자인하면, 정의한 후 향후 개발자에게 계약을 기반으로 호환되는 adpater을 구현할 필요가 없어진다.

![https://khalilstemmler.com/img/blog/typescript/abstract-class/ports-and-adapters.svg](https://khalilstemmler.com/img/blog/typescript/abstract-class/ports-and-adapters.svg)

<br/>

### 추상화는 아래 개념과 관계되어 있다.

- **[Liskov Substitution Principle](https://khalilstemmler.com/articles/solid-principles/solid-typescript/)**  - 유효한 하위 타입을 정의하기 때문에, 각 구현은 계약을 구현하는 한 작동하고 교환할 수 있어야 합니다.
- **[Dependency Inversion](https://khalilstemmler.com/articles/tutorials/dependency-injection-inversion-explained/) -** 구체화에 직접 의존하지 않고 구체화가 의존하는 추상화에 의존한다. 이걸로 핵심 코드 단위를 테스트할 수 있게 된다.
- **[Inversion of Control](https://khalilstemmler.com/articles/tutorials/dependency-injection-inversion-explained/)** - 우리는 hook point에서 클라이언트 개발자가 동작을 정의할 수 있는 기능을 제공할 수 있다.

<br/>

## 타입스크립트에서의 추상클래스

TypeScript에서 추상 클래스는 보통 관련 하위 클래스 안에서 공유할 몇 가지 행동을 둔다. 추상 클래스를 인스턴스화할 수 없다는 점을 아는 것은 매우 중요하다. 공유되는 행동에 접근하는 유일한 방법은 추상 클래스를 하위 클래스로 확장하는 것이다.

예시를 위해 디지털 책을 판매한다고 가정해보자. 몇몇은 공통 로직이고 몇몇은 특정한 부분의 로직이다. 우선, 공통 로직을 위해 책 추상화를 만들어보자.

추상 클래스를 선언하려면 abstract 키워드를 사용한다.

```tsx
abstract class Book { 
  // .. 
}
```

<br/>

### common properties를 정의하기

책 추상화 안에서, 우리는 책에 대한 계약을 정의할 수 있다. 모든 책의 하위 클래스에는 저자, 제목 같은 속성이 필요하다고 가정해보자. 이걸 인스턴스의 변수로 정의하고 추상 클래스의 생성자를 이용해 입력으로 받아들이면 되겠쥬?

바로 이렇게요!

```tsx
abstract class Book { 
  private author: string;  private title: string;

    constructor (author: string, title: string) {
    this.author = author;
    this.title = title;
  }
}
```

<br/>

### 공통 로직을 정의하기

그런 다음 일반 메서드를 써서 책 추상 클래스 안에 공통 로직을 넣어보자.

```tsx
abstract class Book { 
  private author: string;
  private title: string;

  constructor (author: string, title: string) {
    this.author = author;
    this.title = title;
  }

  // Common methods
  public getBookTitle (): string {    
		return this.title;  
	}  
	
	public getBookAuthor (): string {    
		return this.title;  
	}
}
```

기억하세요! 추상 클래스는 인터페이스와 비교할 수 있는 추상화다. 추상 클래스는 직접 인스턴스화할 수 없다. 예시로 쓰긴 했으나 아래와 같은 오류가 발생할 거다.

```tsx
let book = new Book (          // ❌ error TS2511: Cannot create an 
  'Robert Greene',             // instance of an abstract class. 
  'The Laws of Human Nature'
);
```

그러므로 우리는 Book의 구체적인 클래스 중 하나로 PDF 클래스를 만든다. 

추상 클래스인 Book을 새로운 하위 클래스를 이용해 확장/상속한다.

```tsx
class PDF extends Book { // Extends the abstraction
  
  private belongsToEmail: string;
  
  constructor (author: string, title: string, belongsToEmail: string) {
    super(author, title); // Must call super on subclass
    
    this.belongsToEmail = belongsToEmail;
  }
}
```

PDF는 구체적인 클래스이므로 인스턴스화할 수 있다. PDF 객체에는 Book 추상화의 모든 속성과 메서드가 있어 마치 원래 PDF에 선언된 것처럼 메소드를 호출할 수 있다. 

```tsx
let book: PDF = new PDF(
	'Robert Greene', 
	'The Laws of Human Nature', 
	'khalil@khalilstemmler.com'
);

book.getBookTitle();  // "The Laws of Human Nature"
book.getBookAuthor(); // "Robert Greene"
```

<br/>

### 필수 메서드 정의하기

추상 클래스는 한 가지 중요한 특징을 갖는다 : 추상 메서드의 개념이다.

추상 메서드는 구현하는 하위 클래스에서 정의해야 하는 메서드다.

아래와 같이 정의한다 :

```tsx
abstract class Book { 
  private author: string;
  private title: string;

  constructor (author: string, title: string) {
    this.author = author;
    this.title = title;
  }

  abstract getBookType (): string; // No implementation}
```

추상적인 메서드에 대한 구현이 없다는 것을 발견 했나요? 이는 서브 클래스에서 추상 메서드를 구현하기 때문이다. 

PDF와 추가 EPUB 서브 클래스로 예시를 보면, 두 개 모두에 필요한 추상 메서드를 추가한다.

```tsx
...

class PDF extends Book {
  ...
  getBookType (): string { // Must implement this    return 'PDF';  }}

class EPUB extends Book {
  
  constructor (author: string, title: string) {
    super(author, title);
  }
  
  getBookType (): string { // Must implement this    return 'EPUB';  }}
```

필요한 추상 메서드를 구현하지 않을 경우 클래스는 구체화되지 않는다. 즉, 코드를 컴파일하거나 인스턴스를 만드려고 하면 오류가 발생한다. 

이렇게 쓰는 이유는 뭘까? 이걸 이해하기 위한 가장 좋은 방법은 실제 시나리오를 고려하는 것이다. 

바로 아래부터 이 이유에 대해서 알아보자.

<br/>

## 사용 케이스(추상 클래스를 사용하는 경우)

이제 추상 클래스가 어떻게 작동하는지 알았으니, 실제 사용 케이스를 알아보자.

추상 클래스를 사용하는 두 가지 주요한 경우가 있다.

1. 공통된 행동을 공유할 때
2. 템플릿 메소드 패턴 (프레임워크 hook method)

<br/>

### 공통 행동 공유(기본 HTTP class 예제)

가장 많이 알려진 게 여러 백엔드 API 엔드포인트에 의존하는 프론트엔드 애플리케이션 내에서 HTTP 데이터 가져오는 로직을 수행하는 것이다.

아래에서 데이터를 가져와야 한다고 가정해 보자.

- **`Users` API  : http://example.com/users**
- **`Reviews` API :  example.com/reviews**

공통 로직은 뭘까?

- Axios 같은 HTTP 라이브러리를 셋팅
- 일반적인 refetching 로직을 설정하기 위한 인터셉터 설정 (예: 액세스 토큰이 만료될 때)
- HTTP 메소드 수행

추상 클래스를 사용해서, BaseAPI를 이미 제공된 일부 구현으로 추상화를 정의할 수 있다.

```tsx
export abstract class BaseAPI {
  protected baseUrl: string;
  private axiosInstance: AxiosInstance | any = null;
   
  constructor (baseUrl: string) {
    this.baseUrl = baseUrl
    this.axiosInstance = axios.create({})
    this.enableInterceptors();
  }

  private enableInterceptors () {
    // Here's where you can define common refetching logic
  }

  protected get (url: string, params?: any, headers?: any): Promise<any> {
    return this.getAxiosInstance({
      method: 'GET',
      url: `${this.baseUrl}${url}`,
      params: params ? params : null,
      headers: headers ? headers : null
    })
  }

  protected post (url: string, params?: any, headers?: any): Promise<any> {
    return this.getAxiosInstance({
      method: 'POST',
      url: `${this.baseUrl}${url}`,
      params: params ? params : null,
      headers: headers ? headers : null
    })
  }
}
```

여기서, BaseAPI 추상화 내에서 낮은 수준의 행동을 배치했다. 이제 우리는 하위 클래스에서 하이 레벨의 행동을 정의할 수 있다.

```tsx
export class UsersAPI extends BaseAPI {
  constructor () {
    super('http://example.com/users');
  }
 
  // High-level functionality
  async getAllUsers (): Promise<User[]> {    
    let response = await this.get('/');    
    return response.data.users as User[];  
  }...
}
```

이런 패턴은 널리 쓰인다. Apollo의 REST DataSource API도 동일하게 추상 클래스 접근 방식을 기반으로 한다.

<br/>

### 템플릿 메소드 디자인 패턴 (front-end 라이브러리, 프레임워크 예시)

템플릿 메소드 패턴은 추상 클래스에서는 알고리즘의 뼈대를 정의하고 일부 단계를 하위 클래스에 맡기고 그에 따라 작동하는 행동 디자인 패턴이다.

![https://khalilstemmler.com/img/blog/typescript/abstract-class/template_method_design_pattern.svg](https://khalilstemmler.com/img/blog/typescript/abstract-class/template_method_design_pattern.svg)

커스터마이징이 가능하기 때문에 템플릿 메소드 패턴은 앵귤러, 뷰, 리액트 같은 라이브러리, 프레임워크에서 널리 사용된다. 예를 들기 위해, 리액트를 생각해보자.

클래스 기반 컴포넌트를 사용해 리액트 컴포넌트를 만들 때, 반드시 positively 구현해야 할 것은 무엇일까?

바로 렌더링 방법일 것이다. 

따라서 컴포넌트에 대한 추상 클래스의 단순한 버전은 아래처럼 보일 수 있다.

```tsx
abstract class Component {
  private props: any;
  private state: any;

  abstract render (): void; // Mandatory
}
```

위의 추상 클래스를 사용하려면 추상화를 확장하고 render 메소드를 구현하해야 한다.

```tsx
class Box extends Component {
  render () {
    // Must implement this method
  }
}
```

렌더링은 중요하기 때문에 구현해야 한다 브라우저에 표시될 HTML, CSS는 개발자가 결정한다. 라이브러리는 결정할 수 없기 때문에 개발자가 제공해야 한다.

추상 클래스가 이런 문제 해결에 적합한 도구다. 렌더링 단계는 많은 단계 중 하나의 단계이고 그 외 뒤에서 실행되는 알고리즘이 있기 때문이다.

여기서부터는 mounting, updating, unmounting이 나오는데 class component 때 보던 개념이라 지금도 공부하나? 싶어서 일단 생략했다. 그래도 프론트 개발자면 알긴 해야 하는데 나는 예전에 한 번 본 적은 있어서 일단 대충 읽고 넘어가는 걸로.

<br/>

## 요약

- 객체 지향 프로그램은 구체화와 추상화를 포함한다. 추상 클래스는 객체 지향 프로그래밍에서 추상화를 구현할 수 있는 두 가지 방법 중 하나이다.
- 추상 클래스를 사용하여 공유하는 행동을 정의하거나 템플릿 메소드 패턴을 구현하자

<br/>

## 자주 질문 받는 사항

### 추상 클래스의 생성자는 뭘 위해 있나?

추상 클래스의 생성자는 로우 레벨의 추상 클래스 메서드가 사용할 수 있도록 추상 클래스 내에서 속성을 설정하거나 다른 설정 로직을 실행하는 데 사용된다. 서브클래스(파생된 클래스)에서 super를 호출해서 생성자를 호출한다.

### 추상 메서드는 뭘 하나요?

추상 메서드는 클라이언트가 추상 클래스로 작동하기 위해 필요한 특정 기능을 구현하도록 강제하기 위해 존재한다. 정확한 세부 사항은 하위 클래스로 추상화 한다.

### 인터페이스와 추상 메서드의 차이는 뭔가요?

둘 다 객체지향 프로그래밍의 추상화 예시다. 

인터페이스는 추상화의 속성과 메서드만 정의할 수 있으나, 

추상 클래스는 몇 가지 일반 행동 외에도 속성과 메서드를 정의할 수 있다.

implements 키워드를 쓰는 건 인터페이스고, extends 키워드를 쓰는 건 추상 클래스이기도 하다.

추상화가 필요할 때, 기본적으로는 인터페이스를 쓰지만 일반적인 동작을 정의하거나 템플릿 메서드 패턴을 사용해 알고리즘을 일반화해야 할 때는 추상 클래스를 쓰세요.
