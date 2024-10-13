# 학습 내용
- C언어
- 자료구조 C언어 구현
- B-Tree

# 퀴즈 

## 피보나치 수열(1,1,2,3,5,...)의 n번째 값을 리턴하는 함수를 dynamic programming의 상향식, 하향식 접근법으로 작성

```python
def fib_bottom_up(n):
    # n이 1일 경우 바로 값을 반환
    if n <= 1:
        return n

    # 초기 두 피보나치 수
    fib_0 = 0
    fib_1 = 1

    # 피보나치 수를 계산
    for i in range(2, n + 1):
        # 다음 피보나치 수는 이전 두 수의 합
        next_fib = fib_0 + fib_1
        # 이동
        fib_0 = fib_1
        fib_1 = next_fib
    
    return fib_1
```

```python
def fib_top_down(n, memo=None):
    # 메모이제이션을 위한 리스트 초기화
    if memo is None:
        memo = [-1] * (n + 1)
    
    # 기저 조건
    if n <= 1:
        return n
    
    # 이미 계산된 값이면 반환
    if memo[n] != -1:
        return memo[n]
    
    # 재귀적으로 피보나치 수 계산
    memo[n] = fib_top_down(n-1, memo) + fib_top_down(n-2, memo)

    return memo[n]
```