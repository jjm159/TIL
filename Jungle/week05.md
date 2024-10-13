# 학습 내용
- 이진 트리, AVL, 레드 블랙 트리

# 퀴즈

## 55와 22가 출력되도록

### 소스
```c
#include <stdio.h>
int main()
{
    int numArr[5] = { 11, 22, 33, 44, 55 };
    int *numPtrA;
    void *ptr;

    numPtrA = &numArr[2];
    ptr = numArr;

    printf("%d\n", /** 변수는numPtrA만을 사용하세요. **/);
    printf("%d\n", /** 변수는 ptr만을 사용하세요. **/);

    return 0;
}
```

### 답
```
*(numPtrA + 2)
*((int *)ptr + 1) or *(++(int *)ptr)
```

## 실행 결과
```c
#include <stdio.h>
int main() {
    char* str[2];
    str[0] = "hello!";
    str[1] = "jungler";
    printf("1. %s\n", str[0] + 1);
    printf("2. %s\n", (str + 1)[0] + 2);
    return 0;
}
```

### 답
1. ello!
2. ngler

## ??? 부분 값
```c
#include <stdio.h>
void main() {
    char arr_char[5];
    int arr_int[5];
    int64_t arr_int64[5];
    printf("char base : %p, %p\n", arr_char, &arr_char[2]);
    printf("int base : %p, %p\n", arr_int, &arr_int[0]);
    printf("int64 :%p, %p\n", arr_int64, &arr_int64[3]);
}
```

### 출력:
char base : 0x8004000fd3, ???
int base : 0x8004000fbc, ???
Int64 : 0x8004000f90, ???

### 답
0x8004000fd5
0x8004000fbc
0x8004000fa8


## 추가 문제 
- AVL, RB Tree, BTS에 데이터 넣은 후 모습, 뺐을 때 모습
- n개의 원소가 삽입되어 있는 이진 탐색 트리, AVL 트리, 레드 블랙 트리에 새로운 원소 삽입 시 최악의 시간 복잡도
    - 이진 탐색 트리 : O(n)
    - AVL 트리, 레드 블랙 트리 : O(log n)