---
layout: post
title: AWS SES 테스트 시 Email 미전송 문제 / Sender 이름 한글 깨짐 문제
permalink: /sabjil/ses-test
position: sabjil
parent: sabjil
nav_order: 2
---

# 삽질 일기 2편 : AWS SES 테스트 시 Email 미전송 문제 / Sender 이름 한글 깨짐 문제

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

1. 실제 코드상으로는 잘 동작하는데 테스트 코드에서만 제대로 동작하지 않아서 왜 그럴까 싶었다.
2. 다른 건 한글이 잘 나오는데 Sender 이름만 한글이 깨진다.


---

<br/>

### 해결 방안 

1. 코드가 동작하기 전에 테스트 코드가 종료 되어 보내지지 않은 것이었다. latch를 써서 보내는 걸 기다려주자.

```kotlin
    
    @Test
    fun sendEmailTest() {
        val request = SendEmailRequest().apply {
            withDestination(Destination().withToAddresses(TO))
            withMessage(Message().apply {
                withBody(Body().apply {
                    withHtml(Content().withCharset("UTF-8").withData(HTMLBODY))
                        .withText(Content().withCharset("UTF-8").withData(TEXTBODY))
                })
                withSubject(Content().withCharset("UTF-8").withData(SUBJECT))
            })
            withSource(FROM)
        }

        val threadCount = 1
        val latch = CountDownLatch(threadCount)


        awsSimpleEmailServiceAsync.sendEmailAsync(request, object : AsyncHandler<SendEmailRequest, SendEmailResult> {
            override fun onSuccess(request: SendEmailRequest?, result: SendEmailResult?) {
                println("\n on Success ")
                val response: String = result!!.messageId
                println(response)
                latch.countDown()
            }

            override fun onError(e: Exception) {
                println("\n on Error")
                println(e.message)
                latch.countDown()
            }
        })
        latch.await()
    }
  
```



<br/>



2. 알고보니 얘만 MIME 인코딩이 필요하다고 한다.

  - import com.amazonaws.AmazonWebServiceRequest 에 주석으로 달려 있다.
  
```
The sender name (also known as the friendly name) may contain non-ASCII characters. 
These characters must be encoded using MIME encoded-word syntax, as described in RFC 2047 . MIME encoded-word syntax uses. 
the following form: =?charset?encoding?encoded-text?= . 
destination – The destination for this email, composed of To:, CC:, and BCC: fields. message – The message to be sent.
```

  -  `javax.mail:mail:1.4.1` 의존성에 있는 MimeUtilty를 활용했다.
```kotlin
private fun encodeNameWithAddress(fromName: String, from: String): String {
        return MimeUtility.encodeText(fromName, "utf-8", "Q") + "<" + from + ">"
    }
```


<br/>

^^ 끝
