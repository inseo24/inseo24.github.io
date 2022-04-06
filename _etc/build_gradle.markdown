---
layout: post
title: build.gradle 보기
permalink: /etc/build/gradle
position: etc
nav_order: 6
parent: etc
---


# build.gradle 쓰기

<br/>

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

<br/>

## 기본 용어

<br/>

- classpath : 클래스나, jar 파일이 존재하는 위치
- compileOnly : 컴파일 시점에 필요한 라이브러리. 컴파일 이후 실제 실행할 때는 없어도 됨
- runtimeOnly : 컴파일 시점에는 필요없지만 실제 실행 시점에 필요한 라이브러리
    
    runtimeOnly 라이브러리는 호환 가능한 라이브러리가 있으면 컴파일 이후엔 다른 호환 가능한 라이브러리로 교체하는 것이 가능하다.(컴파일하지 않고 라이브러리만 교체)
    
    예를 들어, h2 라이브러리는 컴파일 시점에는 전혀 필요하지 않고, 실행 시점에만 필요하다. 빌드된 최종 결과 라이브러리에 포함된다.
    
<br/>

## build script에 외부 의존성 작성

<br/>

[https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:build_script_external_dependencies](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:build_script_external_dependencies)

script classpath를 직접 조작하기보다는 플러그인을 적용하는 걸 추천한다고 한다.(그레이들 홈페이지에서는)

<br/>

## build.gradle 보기

<br/>

**아래는 groovy로 작성됨**(build.gradle.kt 아님!)([출처: 설로인 샌드박스](https://github.com/sirloin-dev/meatplatform-sandbox/blob/main/server/build.gradle))

<br/>

```groovy
/*
 * sirloin-sandbox-server
 * Distributed under CC BY-NC-SA
 */
buildscript {
    ext {
        // lang
        version_target_jvm = 17
        version_kotlin = "1.6.10"
        version_javax_validation = "2.0.1.Final"
        version_javax_inject = "1"
        version_jsr305 = "3.0.2"

        // Spring + infrastructure
        version_servlet = "2.2.16.Final"
        version_springBoot = "2.6.3"
        version_springBootTest = "2.6.3"
        version_springDataJdbc = "2.3.2"
        version_hikariCP = "5.0.1"

        // MySQL
        version_mysql = "8.0.28"

        // Library
        version_jackson_kotlin = "2.13.1"
        version_sirloin_jvmlib = "1.0.1"

        // Testing & Code quality
        version_junit5 = "5.8.2"
        version_detekt = "1.19.0"
        version_jacoco = "0.8.7"
        version_hamcrest = "2.2"
        version_mockito = "4.3.1"
        version_mockitoKotlin = "4.0.0"
        version_restDocs = "2.0.6.RELEASE"
        version_javaFaker = "1.0.2"
        version_h2database = "2.1.210"
        version_asciidoctor = "3.3.2"
    }

    repositories {
        mavenCentral()
        maven { url "https://plugins.gradle.org/m2/" } // 프록시로 추가하는 거 같은데 왜인진 모르겠음 (...)
        maven { url "https://repo.spring.io/plugins-release" }
    }

    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$version_kotlin"

        // Spring
        classpath "org.springframework.boot:spring-boot-gradle-plugin:$version_springBoot"
        // https://kotlinlang.org/docs/reference/compiler-plugins.html
        classpath "org.jetbrains.kotlin:kotlin-allopen:$version_kotlin"

        // Testing & Code quality
        classpath "io.gitlab.arturbosch.detekt:detekt-gradle-plugin:$version_detekt"
        classpath "org.asciidoctor:asciidoctor-gradle-jvm:$version_asciidoctor"
    }
}

group "com.sirloin.sandbox.server"

allprojects {
    apply plugin: "idea"
    apply from: "$rootDir/gradle/scripts/java.gradle"
    apply from: "$rootDir/gradle/scripts/kotlin.gradle"
    apply from: "$rootDir/gradle/scripts/detekt.gradle"
    apply from: "$rootDir/gradle/scripts/testing.gradle"
    apply from: "$rootDir/gradle/scripts/jacoco.gradle"

    repositories {
        mavenCentral()
        maven { url "https://oss.sonatype.org/content/repositories/snapshots/" } // 넥서스 소나타입? ...이것도 프록시인듯
        maven { url "https://jitpack.io" } // github에서 가져오려면 필요한 설정
    }

    dependencies {
        // Sir.LOIN jvm libs
        implementation "com.github.sirloin-dev.sirloin-jvmlib:sirloin-jvmlib-annotation:$version_sirloin_jvmlib"
        implementation "com.github.sirloin-dev.sirloin-jvmlib:sirloin-jvmlib-crypto:$version_sirloin_jvmlib"
        implementation "com.github.sirloin-dev.sirloin-jvmlib:sirloin-jvmlib-net:$version_sirloin_jvmlib"
        implementation "com.github.sirloin-dev.sirloin-jvmlib:sirloin-jvmlib-text:$version_sirloin_jvmlib"
        implementation "com.github.sirloin-dev.sirloin-jvmlib:sirloin-jvmlib-time:$version_sirloin_jvmlib"
        implementation "com.github.sirloin-dev.sirloin-jvmlib:sirloin-jvmlib-util:$version_sirloin_jvmlib"

        testImplementation "com.github.sirloin-dev.sirloin-jvmlib:sirloin-jvmlib-test:$version_sirloin_jvmlib"
    }
}
```

<br/>

### buildScript

<br/>

`buildScript` 블록은 나머지 빌드 스크립트에서 사용할 수 있는 플러그인, task 클래스, 기타 클래스를 결정한다. 제 3의 플러그인, task 클래스 또는 기타 클래스(build script 포함!)를 추가로 사용하려면 buildScript 블록에서 해당 종속성을 지정해야 한다.

<br/>

### ext

<br/>

ext : `project.ext` 의 줄임으로 프로젝트를 위한 **ext**ra properties 을 정의 한다고 생각하면 된다. 여기에 작성한 내용에 접근할 때 점을 찍고 접근할 필요 없이 바로 version_xxx 를 쓰면 된다.(뭐 예를 들어, println(project.version_mysql) 이렇게 접근할 것 없이 바로 version_mysql 로 쓰면 된다!)

<br/>

### buildScript vs allprojects

<br/>

buildScript는 gradle 자체를 위한 것으로 gradle이 빌드를 수행할 방법에 대해 변경할 수 있다.

반면 allprojects는 gradle에서 빌드 중인 모듈을 위한 것이다. 

각 모듈에 대한 dependency가 다르게 되면 각 모듈 내에 build.gradle 파일이 각각 존재하기 때문에 보통은 일반적으로 allprojects에 dependency는 비어 있는 경우도 많다.

위의 파일에서는 모든 모듈에서 동일하게 사용할 dependency만 공유하기 위해서 작성되어 있다. 

<br/>

### Gradle에서 Proxy 설정

<br/>

왜 gradle에서 proxy를 설정할까?

프록시가 뭐냐 → 클라이언트가 다른 네트워크 서비스에 간접적으로 접속할 수 있게 도와주는 거다.

그럼 왜 프록시를 쓰겠냐 → 직접 접속이 안되니까 쓰겠지!

보통 회사에서 보안 정책 때문에 외부 package repository 서버 접속이 안되는 경우가 있어서 gradle에서도 proxy 설정을 한다.

[Gradle에서 Proxy를 설정하는 방법](https://docs.gradle.org/current/userguide/build_environment.html#sec:accessing_the_web_via_a_proxy) 을 보면 **gradle.properties** 파일에 저장해서 쓰는 것 같다.

<br/>


1. project의 root 디렉토리 아래에 저장하거나
2. 사용자 홈 디렉토리 아래의 .gradle 폴더 아래에 저장하거나
3. 자바 실행 시 실행 옵션으로 proxy 환경을 셋팅하거나.

이렇게 쓰는 거 같은데 설로인 프로젝트에는 1번 방식으로 되어 있다. 이건 취향 차이인 건가 싶다.

<br/>

### apply

<br/>

![image](https://user-images.githubusercontent.com/84627144/160972227-4ca20270-ff91-4ce5-a6b1-fa47a2e2a132.png)

일단 난 저기 `apply plugin: “idea”` 부터 처음 봤기 때문에 이것부터 검색해봤다.

그냥 별 거 없고 IDEA 플러그인을 사용하기 위한 설정이었다.

IDEA 플러그인은 IntelliJ IDEA에서 사용하는 파일을 생성하여 IDEA(파일 - 프로젝트 열기)에서 프로젝트를 열 수 있다. 외부 종속성(관련 소스 및 javadoc 파일 포함)과 프로젝트 종속성이 모두 고려된다.

<br/>

[apply from:](https://docs.gradle.org/current/userguide/plugins.html#sec:script_plugins)

→ 로컬 파일 시스템이나 원격 위치의 스크립트로부터 플러그인을 자동으로 적용시킨다.

파일시스템 위치는 프로젝트 디렉토리에 위치한 걸 말하고, 원격 위치는 HTTP URL을 말한다. 다양한 script 플러그인은 주어진 타겟에 적용된다. (여기서는 allprojects에 적용됨)

<br/>

[apply plugin:](https://docs.gradle.org/current/userguide/plugins.html#sec:binary_plugins)

plugin id를 쓰는 binary 플러그인에 사용된다. (예를 들어, ‘java’는 Java 플러그인에 관한 것)

<br/>

## 근데 넥서스가 뭐냐

<br/>

당연히 내가 개발환경부터 하나하나 셋팅하고 한 게 아니라 있는 코드에 api 추가하는 식으로 해왔기 때문에 build.gradle 제대로 안봤다 ^ㅁ^ 

Nexus Repository Manager 라는게 있구나.

위에서 프록시를 설정하는 것과 같은 맥락인데 사내망에서 프로젝트 진행 시 외부 레포지터리 접속이 어렵기 때문에 필요한 라이브러리를 다운 받을 수 있게 하는 프록시 역할을 한다.

넥서스에 대한 기본적인 내용이 잘 담긴 [블로그 글](https://nextshds.tistory.com/63)이 있다. 

나는 다뤄보지 못해서 잘 모르겠지만 큰 회사라면 다 쓰겠구나 싶다. 

여튼 저 넥서스 메이븐 레포지터리에 추가해서 쓰는 건가 보다. [(추가 관련 블로그 글)](https://bcho.tistory.com/790)

<br/>

## 다음에는

<br/>

저 gradle 디렉토리 안에 있는 packaging.gradle을 이해해야 api-main 안에 build.gradle 도 볼 수 있고 그 다음 Appilcation.kt에서 어떻게 굴러가는지 알 수 있을 거 같다.

진짜 groovy 도 알아야 하냐? 진짜 도라방스
