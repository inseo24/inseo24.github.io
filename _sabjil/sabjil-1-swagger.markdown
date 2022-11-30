---
layout: post
title: Swagger UI Redirection 문제 (feat. Load Balancer)
permalink: /sabjil/swagger
position: sabjil
parent: sabjil
nav_order: 1
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
4. Swagger-UI는 지정한 path로 검색 시 내부적으로 만든 url로 redirect를 시킴
   1. 이 url은 `REDIRECT_URL_PREFIX + uiRootPath + swaggerUiPath` 구조(`SwaggerUiHome` 코드에서 확인 가능)
   2. 여기서 맨 앞 REDIRECT_URL_PREFIX 는 `"redirect:"`를 의미
   3. `uiRootPath`가 servlet.context-path에서 설정하는 root path에 해당한다. 
   4. `swaggerUiPath`는 `springdoc.swagger-ui.path`로 설정한 값에 해당한다.(yaml 설정)
   5. 문제는 저 리다이렉션을 할 때, `www.xxx.com/service-name/`에서 `/service-name/` 이 날아가 버린다.
   6. 이거 때문에 Istio에서 해당 주소가 어디 서비스인지 찾지 못해 404 에러가 뜬다.


---

<br/>

### 해결 방안


- 첫 번째로 가장 간단한 방법은 server.servlet.context-path를 설정해 주는 것임
   1. application.yml에 server.servlet.context-path = service-name 을 설정해주면,
   2. Swagger UI redirection 할 때, base-path에 포함되어 문제 없이 작동함
      
<br/>

- 1번처럼 하면 Istio 설정에 변경할 게 많아지니 다른 방식을 쓰자고 함
   1. 기본적으로 나오는 petshop config를 안나오게 설정하고, api-docs path를 넣어줌
   2. 이 때, service-name도 같이 써준다.
   3. 이럼 이제 `https://xxxx/service-name/swagger-ui/index.html` 로 접속하면 된다.

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

- 기어코 swagger ui path를 지정해주고 싶다면, 지정한 후 redirect 를 시켜주면 된다. 
   1. 나는 결국 index.html로 가는데 굳이? 라는 생각도 있어서 따로 리다이렉트를 하진 않았음 
   
     ```kotlin
    @Bean
    fun routerFunction(): RouterFunction<ServerResponse> =
        route(GET("/swagger-ui.path에 설정한 url")) {
            ServerResponse.temporaryRedirect(URI.create("/service-name/webjars/swagger-ui/index.html")).build()
        }
     ```


<br/> 
 
**참고**

[https://github.com/springdoc/springdoc-openapi/issues/742](https://github.com/springdoc/springdoc-openapi/issues/742)


---
