---
layout: post
title: 그런 REST API로 괜찮은가
permalink: /etc/rest
position: etc
parent: etc
nav_order: 7
---


# 그런 REST API로 괜찮은가

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


어제밤 이펙티브 코틀린 스터디 준비하다가 같이 스터디 하셨던 분이 디엠으로 추천해주신 영상이다.

영상 제목이 유명해서 뭔가 알고는 있었지만?, 직접 보진 않았어서 이번에는 꼭 끝까지 보고 정리했으면 좋겠다 싶었다.

그래서 영상 보고 쉬엄쉬엄 정리를 해봤다. 


### **1. REST**

### **REST 란?**

- **RE**presentational
- **S**tate
- **T**ransfer
- REST is a way of providing interoperability between computer systems on the Internet
- 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식

### 발전 과정(WEB, HTTP, REST)

### **2) WEB (1991)**

Q. 어떻게 인터넷에서 정보를 공유할 것인가?

A. 표현 형식 - HTML, 식별자 - URI, 전송 방법 - HTTP

### 3**) HTTP / 1.0 (1994 - 1996)**

기존의 웹을 망가트리지 않고 향상 시킬 수 있는 방법?

Q. “How do i improve HTTP without breaking the Web?”

A. HTTP Object Model

### 4**) REST (1998)**

4년 후 Roy T.Fielding은 REST를 Microsoft Research에서 발표

*"Representational State Transfer : An Architectural Style for Distributed Hypermedia Interaction"*

### 5**) REST (2000)**

Roy T.Fielding이 REST를 박사 논문으로 발표

*"Architectural Styles and the Design of Network-based Software Architectures"*

### **2. API**

### **REST API의 등장**

- 1998년 Microsoft XML-RPC, SOAP 등장
- 2000년 Salesforce에서 API 공개했지만 단순한 API임에도 굉장히 복잡했음
- 2004년 flickr에서 API를 SOAP의 형태와 논문의 이름을 가져온 REST라는 이름의 API를 공개했다.
- 그 결과, REST는 SOAP에 비해 굉장히 단순하고 규칙이 적으며 쉽다는 특징으로 급부상하게 된다.
- 2006년 AWS가 자사 API의 사용량이 85%가 REST임을 공개
- 2010년 Salesforce.com에서도 REST API를 추가함을 공개

### **2) Roy T.Fielding의 반박**

2008년 CMS 표준인 CMIS를 제작했다. 이 CMIS에는 REST 바인딩을 지원한다고 했지만, Roy T.Fielding은 CMIS에 REST가 없다고 반박함.

2016년 Microsoft에서 제작한 REST API 가이드라인도 Roy T.Fielding은 REST가 아닌 **HTTP API라고** 이야기 함.

**Microsoft의 REST API**

- URI는 https://{serviceRoot}/{collection}/{id} 형식이어야 한다.
- GET, PUT, DELETE, POST, HEAD, PATCH, OPTIONS를 지원해야한다.
- API 버저닝은 Major.minor로하고 URI에 버전 정보를 포함시킨다 등

이에 대해 Roy T.Fielding은 다음과 같은 말을 했다.

- REST API는 반드시 **hypertext-driven이여야한다.**
- REST API를 위한 **최고의 버저닝 전략은 버저닝을 안하는 것이다.**

### **3. REST API**

### **REST를 따른다는 것은 ?**

그렇다면 여기서 "도대체 무엇이 문제인가?" 에 대한 의문이 들 수 있다.

REST API를 풀어 쓴다면 다음과 같다.

- REST 아키텍쳐 스타일을 따르는 API
- REST는 분산 하이퍼미디어 시스템(ex. 웹)을 위한 아키텍쳐 스타일
- **아키텍쳐 스타일이란 제약 조건의 집합**
- **즉, 제약 조건들을 모두 지켜야 REST를 따른다고 할 수 있다.**

**REST 아키텍처를 구성하는 스타일은 다음과 같다.**

대체로 오늘날 사용되는 API는 대부분의 위의 조건을 만족시킨다. 
**하지만 이 중 Uniform interface를 만족시키지 않은 경우가 많다.**

### **Uniform Interface의 제약 조건**

- Identification of resources
- Manipulation of resources through representations
- **Self-descriptive messages**
- **H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate (HATEOAS)

오늘날 'Identification of resources'와 'Manipulation of resources through representations'의 조건은 대체로 잘 만족한다. 

하지만 나머지 2개의 조건은 잘 지켜지지 않음(Self-descriptive messages, HATEOAS)

---

### Self-descriptive messages 의 조건

**만족하지 않는 경우의 예**

- *목적지가 생략되어 있어 위반*

```java
GET / HTTP /1.1
```

- *클라이언트가 해석하지 못하기 때문 위반*

```java
HTTP/1.1 200 OK
[ {"op": "remove", "path": "a/b/c/" }
```

**만족하는 경우의 예**

- *목적지를 정의했기에 만족*

```java
GET / HTTP /1.1
Host: www.test.com
```

- *Content-Type의 application/json-patch+json 명세를 통해 Body 내용을 해석할 수 있어 만족*

```java
HTTP/1.1 200 OK
Content-Type: application/json-patch+json
[ {"op": "remove", "path": "a/b/c/" }
```

---

### HATEOAS의 조건

- *<a> 태그를 통해 HyperLink를 사용한 상태 전이가 가능하기 때문에 HTML은 이 조건을 만족한다.*

```java
HTTP/1.1 200 OK
Content-Type: text/html

<html>
<head></head>
<body><a href="/test">test</a></body>
<html>
```

- *JSON은 HTTP의 Link Header를 통해 만족시킬 수 있다.*

```java
HTTP/1.1 200 OK
Content-Type: application/json
Link: </article/1>; rel="previous",
      </article/3>; rel="next";

{
      "title": "The second article",
      "contents": "blah blah.."
}
```

---

- **왜 Uniform Interface를 만족해야할까?**
    
    → **독립적 진화**
    
    - 서버와 클라이언트가 각각 독립적으로 진화한다.
    - 서버의 기능이 변경되어도 클라이언트를 업데이트할 필요가 없어야 한다.(서로 영향을 받지 않게끔)
    - REST를 만들게 된 계기 "How do I improve HTTP **without breaking the Web"**

### **REST를 잘 적용한 사례 (Web)**

- 웹 페이지를 변경했다고 웹 브라우저를 업데이트할 필요가 없다.
- 웹 브라우저를 업데이트했다고 웹 페이지를 변경할 필요가 없다.
- HTTP 명세가 변경되어도 웹은 잘 동작한다.
- HTML 명세가 변경되어도 웹은 잘 동작한다.

REST가 웹의 독립적 진화에 도움을 준 것들은 다음과 같다.

- HTTP에 지속적으로 영향을 줌
- Host 헤더 추가
- 길이 제한을 다루는 방법이 명시 (414 URI Too Long 등)
- URI에서 리소스의 정의가 추상적으로 변경됨 ("식별하고자 하는 무언가")
- 기타 HTTP와 URI에 많은 영향을 줌
- HTTP / 1.1 명세 최신판에서 REST에 대한 언급이 들어감

### **왜 REST가 잘 안되는가**

일반 웹 페이지 vs HTTP API

[Web vs HTTP API](https://www.notion.so/1054e2e52f3348b888f6d9762b43af1c)

[HTML vs JSON](https://www.notion.so/a32dc38204a446a9abae368bfc64eced)

즉, 문법 해석은 가능하지만, **의미를 해석하려면 별도의 문서가 필요하다. (**API 명세서 필요)

REST API를 적용하는 방법에는 두가지가 존재한다.

---

### Media Type 방식

```java
GET /todos HTTP/1.1
Host: test.com

HTTP/1.1 200 OK
Content-Type: application/vnd.todos+json

[
   {"id": 1, "title": "회사 가기"},
   {"id": 2, "title": "집에 가기"},
]
```

- Media Type 정의
- Media Type 문서 작성 ('id'가 뭔지, 'title'이 뭔지 의미 정의)
- IANA(Internet Assigned Numbers Authority)에 Media Type을 등록 
(단, 만든 문서를 Media Type의 명세로 등록)

번거롭지만 이 메시지를 보는 사람은 명세를 확인할 수 있으므로 메시지의 의미를 온전히 해석할 수 있음 (Media Type 등록은 선택이긴 함)

---

### Profile 방식

```java
GET /todos HTTP/1.1
Host: test.com

HTTP/1.1 200 OK
Content-Type: application/json
Link: <https://test.com/docs/todos>; rel="profile"

[
   {"id": 1, "title": "회사 가기"},
   {"id": 2, "title": "집에 가기"},
]
```

- "id"가 뭐고 "title"이 뭔지 의미를 정의한 명세를 작성
- Link 헤더에 profile relation으로 해당 **명세를 링크**

이 메시지를 보는 사람은 명세를 찾아갈 수 있으므로 이 문서의 의미를 온전히 해석할 수 있다. 

단, 클라이언트가 Link 헤더(RFC 5988)와 profile(RFC 6906)을 이해해야하며 Content negotiation을 할 수 없다는 단점을 가지고 있다.

---

### HATEOAS 방식

```java
GET /todos HTTP/1.1
Host: test.com

HTTP/1.1 200 OK
Content-Type: application/json
Link: <https://test.com/docs/todos>; rel="profile"

[
   {"link": "https://test.com/todos/1", "title": "회사 가기"},
   {"link": "https://test.com/todos/2", "title": "회사 가기"},
]
```

```java
GET /todos HTTP/1.1
Host: test.com

HTTP/1.1 200 OK
Content-Type: application/json
Link: <https://test.com/docs/todos>; rel="profile"

[
   "links": {
      "todo": "https://test.com/todos/{id}"
   },
   "data": [
      {"id": 1, "title": "회사 가기"},
      {"id": 2, "title": "회사 가기"}
   ]
]
```

```java
GET /todos HTTP/1.1
Content-Type: application/json

{
   "title": "점심 약속"
}

HTTP/1.1 204 No Content
Location: /todos/1
Link: </todos/>; rel="collection"
```

**데이터에 다양한 방법으로 하이퍼링크를 표현한다. (**단, 링크를 표현하는 방법을 직접 정의해야한다.)

### **JSON으로 하이퍼 링크를 표현하는 방법을 정의한 명세들을 활용**

- JSON API
- HAL
- UBER
- Siren
- Collection+json

### **정리**

- 오늘날 대부분의 API는 REST를 따르지 않고 있다. (특히 Self-descriptive, HATEOAS)
- REST API는 모든 제약조건을 지켜야한다.


### 느낀점

- 생각보다 REST에 대해 제대로 모르고 있었다는 생각이 들었다.
- 전에 모던 딥다이브 책 스터디할 때 스터디장이 REST가 현실에 지켜지지 않는 경우가 많다면서 엄청 꼼꼼히 정리해줬었는데, 그 때 대충 읽고 넘겼던 걸 이번에 다시 보게 된 거 같다.
- 기존 시스템을 건들지 않으면서 발전시키는 게 얼마나 어려운지 좀 느꼈던 거 같다.