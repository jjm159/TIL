# 학습 내용
- 자료구조와 알고리즘
    - 선형 자료구조
    - 재귀, 완전탐색

# 퀴즈

## 재귀함수의 장단점
- 장점
    - 코드가 직관적이어서 더 깔끔하고 이해하기 쉽다.
- 단점
    - 함수 호출시 메모리가 추가적으로 사용되어 메모리 소비가 많고, 잘못하면 스택 오버플로우 발생할 수 있다.
    - 반복문에 비해 빈번하 스택 메모리 할당/해제로 오버헤드가 발생하여 실행 속도가 느릴 수 있다.

## 정수 n이 입력으로 들어오면 1부터 n까지의 피보나치 수열을 구하는 함수 반복문, 재귀 방식 각각 구현 

```python
def fibonacci_loop(n):
    if n <= 0:
        return []
    elif n == 1:
        return [1]
    elif n == 2:
        return [1, 1]
    fib_seq = [1, 1]
    for i in range(2, n):
        fib_seq.append(fib_seq[i-1] + fib_seq[i-2])
    return fib_seq
```

```python
def fibonacci(n):
    if n <= 0:
        return []
    elif n == 1:
        return [1]
    elif n == 2:
        return [1, 1]
    else:
        sequence = fibonacci(n - 1)
        sequence.append(sequence[-1] + sequence[-2])
    return sequence
```