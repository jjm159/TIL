## 목적
- 사용자의 private repository의 commit count를 가져오기 위해서 깃허브 OAuth2.0 Access token이 필요 

## 사전 작업 
- developer setting에서 OAuth2.0 app 추가해서 client_key와 id를 가져와야 함 

## 프로세스

### 1. client_id로 깃허브 로그인 페이지 가져오기 
- 주의: scope에 user를 추가해줘야 total 커밋 개수를 가져올 수 있음
```
  var clientId = ''
  var clientKey = ''
  var scope = 'repo%20user'
  const githubLoginUrl = `https://github.com/login/oauth/authorize?client_id=${clientId}&scope=${scope}

  function buttonTap() {
      window.location.assign(githubLoginUrl)
  }
```

- 해당 로직으로 리다이렉트된 깃허브 로그인 페이지에서 로그인을 완료하면, url에 param으로 code 값 전달
- 이 값을 서버로 전달 

### 2. access_token 요청하기
- https://github.com/login/oauth/access_token 에 요청
```
{
    "client_id": "",
    "client_secret": "",
    "code": "위에서전달받은값"
}
```
- 응답으로 access token 받고 끝

## 참고
- [gitHub OAuth 로그인 구현](https://supersfel.tistory.com/entry/gitHub-OAuth-로그인-구현)
- [(1) oAuth with Github & Python](https://dev.to/schbenedikt/python-oauth-with-github-1bgb)
- [개체 - GitHub Docs](https://docs.github.com/ko/graphql/reference/objects)