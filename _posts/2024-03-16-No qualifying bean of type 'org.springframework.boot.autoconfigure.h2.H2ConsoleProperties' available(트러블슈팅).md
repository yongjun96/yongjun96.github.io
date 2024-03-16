---
categories: [개발, exception]
tags: [exception, 트러블슈팅, SpringTroubleShooting, security]		# TAG는 반드시 소문자로 이루어져야함!
---

## 해당 문구의 에러
<br>
No qualifying bean of type 'org.springframework.boot.autoconfigure.h2.H2ConsoleProperties' available

<br><br>

![트러블슈팅 에러](../assets/img/postimg/2024-03-16/트러블슈팅.png)

<br><br>

controller 테스트 과정에서 발생했다.

<br><br>

securityConfig 안에

<br><br>

![시큐리티 컨피그의 커스텀 메소드](../assets/img/postimg/2024-03-16/webSecurityCustomizer.png)

<br><br>

<span style="color:#FA5858;">.requestMatchers(toH2Console()</span> --> 해당 부분을 지워 주면 해결된다.
