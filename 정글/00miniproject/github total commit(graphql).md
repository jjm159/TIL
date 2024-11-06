- rest api로는 private한 commit 수까지 가져오기 어려움
- grphql api를 사용하면 한번의 요청으로 쉽게 가져올 수 있음
- 시간상 이유로 graphql 라이브러리를 사용하지 않고 그래프 ql 쿼리를 문자열로 바디에 넣어 http 요청으로 해결

```python 

 def getTotalCommitCountToday(self, loginId, accessToken):

        def koreaNowDatetime():
            KST = timezone(timedelta(hours=9))
            return datetime.now(KST)

        def koreaNextDatetime(now):
            tomorrow = now + timedelta(days=1)
            return tomorrow

        def dateTimeToKSTString(date):

            # 문자열로 변환 (ISO 8601 형식)
            formatted_time = date.strftime('%Y-%m-%dT%H:%M:%S%z')

            # '+0900'을 '+09:00' 형식으로 변경
            formatted_time = formatted_time[:-2] + ':' + formatted_time[-2:]

            return formatted_time

        fromDatetime = koreaNowDatetime()
        toDatetime = koreaNextDatetime(fromDatetime)

        return self.getTotalCommitCount(
            loginId=loginId, 
            accessToken=accessToken, 
            fromDatetime=dateTimeToKSTString(fromDatetime), 
            toDatetime=dateTimeToKSTString(toDatetime)
        )

    # from, to를 date 타입으로 받아서, 아래 형태로 바꿔서 query 날리기
    def getTotalCommitCount(self, loginId, accessToken, fromDatetime, toDatetime):
        query = self.getTotalCommitCountQuery(loginId, fromDatetime=fromDatetime, toDatetime=toDatetime)
        headers = self.getTotalCommitCountHeader(accessToken)
        data = json.dumps({"query": query})
        
        response = requests.post(self.graphqlUrl, headers=headers, data=data)

        print(response.status_code)

        # 결과 출력
        if response.status_code == 200:
            result = response.json()  # 응답을 JSON으로 파싱
            try:
                total_commit_contributions = result['data']['user']['contributionsCollection']['totalCommitContributions']
            except KeyError:
                print("The expected key was not found in the response.")
        else:
            print(response.text)

        return total_commit_contributions
    
    def getTotalCommitCountQuery(self, loginId, fromDatetime, toDatetime):
        return f"""
        query {{
            user(login: "{loginId}") {{
                contributionsCollection(from: "{fromDatetime}", to: "{toDatetime}") {{
                    totalCommitContributions
                }}
            }}
        }}
        """

    def getTotalCommitCountHeader(self, accessToken):
        return {
            "Authorization": f"Bearer {accessToken}",
            "Content-Type": "application/json"
        }

```