# 다리놓기

- [boj 1010](https://www.acmicpc.net/problem/1010)

## 조합 풀이
- 조합으로 풀면 쉽게 풀린다.
- 겹칠 수 없다는 조건에 따라, n개의 다리를 m개의 다리에 모두 연결했을 때 남은 m-n개의 다리의 조합 수를 계산하면 답

## DP 풀이
- 점화식을 이용해서 풀 수도 있다.
- 점화식
    - dp[m][n] = dp[m-1][n-1] + dp[m-1][n]
- 해석
    - dp[m-1][n-1]
        - n번째 다리가 m번째 다리를 선택한 경우
    - dp[m-1][n]
        - n번째 다리가 m번째 다리를 선택하지 않은 경우
        - 그래서 m-1의 다리 중 선택했을 것이라는 경우
    