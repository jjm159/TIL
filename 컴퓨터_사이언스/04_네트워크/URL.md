# URL

[HTTP 완벽 가이드](https://book.naver.com/bookdb/book_detail.nhn?bid=8509980)

- URL은 인터넷의 리소스(텍스트, 이미지, 동영상 등 웹 상에서 사용되고 식별할 수 있는 모든 자원)를 가리키는 표준이름이다. 
- URL이 있기 전에는 네트워크 상에 산재해 있는 데이터에 접근하기 위해 애플리케이션마다 다르게 설정된 수많은 분류 방식으로 각 데이터에 접근해야 했다.

- URL은 사실 더 일적인 개념인 URI(Uniform Resource Identifier)의 부분 집합이다. URI에 URL과 URN이 포함된다. 
- URN은 URL의 한계를 극복하기 위해 만들어진 개념이다. 
- URL은 리소스의 이름만을 의미하지 않는다. '위치'를 포함하고 있다. 
- 그래서 리소스의 위치가 변경되면 원래 사용하던 URL로는 해당 리소스에 접근이 불가해진다. 
- URL은 URI라는 '식별자'의 의미만 가지고 있는 것이 아닌 것이다. 
- 그래서 어느 곳에 있든지 그 리소스의 이름을 고유하게 가지게 하여 위치가 변경되어도 접근할 수 있게 하기 위해 탄생한 것이 URN이다. 
- 하지만 아직 표준으로 채택되지 않아 사용되지는 않는다. 

- 한계는 있지만, 그래도 잘 사용하고 있는 URL은 다음과 같은 형식을 가지고 있다.

```
<스킴>://<사용자이름>:<비밀번호>@<호스트>:<포트>/<경로>?<질의>#<프래그먼트>
```

- `스킴` : 리소스를 가져오기 위해 어떤 프로토콜을 사용하여 서버에 접근하는지 가리킨다.
- `사용자이름` : 몇몇 스킴은 리소스에 접근하기 위해 사용자 이름이 필요하다.
- `비밀번호` : 사용자의 비밀번호를 가리킨다. 사용자이름에 콜론(:)으로 이어서 기술한다.
- `호스트` : 리소스를 호스팅하는 서버의 호스트 명이나 IP주소.
- `포트` : 리소스를 호스팅하는 서버가 열어놓은 포트번호. 대부분의 스킴들은 기본 포트 번호를 가지고 있다. HTTP의 기본 포트번호는 80이다.
- `경로` : 이전 컴포넌트와 빗금(/)으로 구분되어 있으며, 서버 내 리소스가 서버 어디에 있는지
- `파라미터` : 위 사진에는 없지만, 경로 다음에 나올 수 있다. 쿼리스트링과 같이 키-밸류 형태를 가지고 있다. RestfulAPI에서의 지침으로는, 파라미터는 경로에 추가하여 리소스를 식별하기 위해 사용하고, 쿼리는 저 리소스를 분류하거나 필터링하기 위해 사용한다고 한다. 리소스를 식별하기 위해 경로에 추가적으로 제공되는 정보라고 보면 될 것 같다.
- `쿼리스트링` : 웹 애플리케이션에 파라미터를 전달하기 위해 사용된다. 이 파라미터는 애플리케이션이 제공하는 리소스의 범위를 좁히기 위해 사용된다. **필터링과 분류**!
    
    > Best practice for RESTful API design is that path params are used to identify a specific resource or resources, while query parameters are used to sort/filter those resources.
    > 
- `프레그먼트` : 리소스의 조각이나 일부분을 가리키는 이름이다. URL이 특정 객체를 가리킬 경우에 프래그먼트 필드는 서버에 전달되지 않는다. 이 필드는 클라이언트에서만 사용한다.

### parameter와 query의 차이

[When do I use path params vs. query params in a RESTful API?](https://stackoverflow.com/questions/30967822/when-do-i-use-path-params-vs-query-params-in-a-restful-api)

### 정리 너무 잘해주셨다!😃

[[HTTP] URI, URL 구조](https://victorydntmd.tistory.com/287)

---
### 기타 참고
[RFC 1738 - Uniform Resource Locators (URL)](https://tools.ietf.org/html/rfc1738)

[URI vs URL vs URN :: 마이구미](https://mygumi.tistory.com/139)

[](https://tools.ietf.org/html/rfc3986#page-50)

[해시뱅(#!)에 대해서... :: Outsider's Dev Story](https://blog.outsider.ne.kr/698)