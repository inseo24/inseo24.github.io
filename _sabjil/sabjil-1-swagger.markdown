---
layout: post
title: 1 : Swagger UI 와 Load Balancer 
permalink: /sabjil/swagger
position: sabjil
---


# 삽질 일기 1편 : 로드 밸런싱 + Swagger UI Redirection 문제 해결


<br/>

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

### 문제

1. Swagger UI에 path로 값을 지정했고 Local 환경에서 잘 동작함 
2. 배포 후 404 Not Found 가 뜸
3. 환경 : 
   1. Istio 를 이용해 로드 밸런싱을 하고 있었고, 
   2. `www.xxx.com/service-name/` 로 서비스를 구분해 로드 밸런싱을 해줌
4. Swagger-UI는 지정한 path로 검색 시 base-path로 redirect를 시킴
   1. 이 때 `/service-name/` 이 날아가버림 
   2. Istio에서 해당 주소가 어디 서비스인지 찾지 못해 404 에러가 뜸
---

<br/>

### 해결 방안


1. 첫 번째로 가장 간단한 방법은 server.servlet.context-path를 설정해 주는 것임
   1. application.yml에 server.servlet.context-path = service-name 을 설정해주면,
   2. Swagger UI redirection 할 때, base-path에 포함되어 문제 없이 작동함

      
<br/>

2. 1번처럼 하면 Istio 설정에 변경할 게 많아지니 다른 방식을 쓰자고 함
   1. 기본적으로 나오는 petshop config를 안나오게 설정하고, api-docs path를 넣어줌
   2. 이 때, service-name도 같이 써준다.

```yaml
springdoc:
  api-docs:
    path: /swagger/api-docs

swagger-ui:
  config-url: /service-name/api-docs/swagger-config
  url: /service-name/swagger/api-docs
  disable-swagger-default-url: true
```

<br/> 

3. 기어코 swagger ui path를 지정해주고 싶다면, 지정한 후 redirect 를 시켜주면 된다. 
   1. 근데 결국 index.html로 가는데 굳이? 라는 생각도 있어서 따로 리다이렉트를 하진 않았음


<br/> 
 
**참고**

[https://github.com/springdoc/springdoc-openapi/issues/742](https://github.com/springdoc/springdoc-openapi/issues/742)


---
