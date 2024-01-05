## 삽질의 배경

OAuth 로그인을 성공한 후에 `response.sendRedirect()`를 호출해서, 응답값에 리다이렉트할 링크를 지정해 주었다. <br>
그리고 이제 쿠키에 JWT토큰을 넣으려고 했는데, 계속 쿠키에 들어가질 않았다. <br>
SameSite도 알아보고, 별에 별걸 다 알아봤는데도 정확한 원인 파악이 되지 않았다. <br>
온갖 디버깅 이후 알아낸 사실은 알고보니 addHeader 메서드 자체가 안먹히는 것이었다.

### 왜 `response.addHeader()` 메서드가 동작하지 않았을까?

그 원인은 `response.sendRedirect()`메서드에 있었다.

![image](https://github.com/gomudayya/TIL/assets/129571789/0f5f4165-cb54-44fc-9d1b-4fc5a445ca6a)

`HttpServletResponse`의 구현체들은 `sendRedirect()`메서드를 호출하면 위와 같이 `setAppCommited(true)` 메서드를 호출한다. <br>
`setAppCommitted(true)` 메서드는 Response객체의 isCommitted boolean 상태 변수를 true값으로 바꾼다.
이렇게 함으로써 해당 Response가 커밋된 상태라고 표시를 해주는 것이다.

해당내용은 주석에도 적혀있는데

> Sends a redirect response to the client using the specified redirect location URL with the status code SC_FOUND 302 (Found),
> clears the response buffer and commits the response.

``sendRedirect()``를 호출하면 response 버퍼를 비우고 response를 커밋한다고 나와있다. <br>
이렇게 commited상태가 되면 응답객체를 아무리 변경하려고 해도 변경이 되지 않는다

![image](https://github.com/gomudayya/TIL/assets/129571789/725d5916-8885-4454-96ee-5e669fcb6435)

위의 코드를 보면 response를 조작하는 모든 메서드들이 response의 상태를 변경하기 전에 `isCommited()`를 호출해 커밋된상태인지 아닌지 확인한다. <br>
그리고 커밋된 상태이면 그냥 return을 시킨다.

### 그래서 Committed 된다는게 무슨 의미인데?

DB의 커밋도 아니고, Git의 커밋도 아니고.. Response에서


