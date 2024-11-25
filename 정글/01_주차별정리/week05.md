# 학습 내용
- 이진 트리, AVL, 레드 블랙 트리
- C언어 포인터, 컴파일, 디버깅, make 사용법
- 동적메모리 할당, 프로시저, 링커

---

# 학습 정리

### c 컴파일 도구에는 무엇이 있을까?

컴파일러 엔진
- gcc(GNU, 오픈소스), LLVM(LLVM 재단, 오픈소스), MSVC(마이크로소프트)
컴파일러 프런트
- 비주얼스튜디오, clang, ..

컴파일러 엔진을 사용한다는건 이런 느낌
```python
import LLVM as ll

code = input()

binary = ll.compile(code) # 기계어로 번역
```

### make 왜 써? 
make란?
- 빌드 프로세스 관리 프로그램. 스크립트에 작성된 파일들의 컴파일과 링킹 작업 수행
왜 사용?
- 수정된 파일만 재컴파일하기 때문에 빌드 시간을 크게 단축시켜준다.
- 여러 플랫폼에서 동작
- 컴파일 옵션이나 그 외 컴파일 환경에 관련된 flag들을 makefile에서 정의하고 관리

### 동적 메모리 할당
#### 정의
메모리를 실행시간에 원하는 시점에 원하는 크기만큼 할당 받는 것

#### 방법
- malloc
    - memory 할당 받는 함수
    - 초기화되지 않기 때문에 쓰레기 값이 들어 있음
    - 인자 1개, 크기
- calloc
    - n개의 메모리 할당
    - 모든 비트를 0으로 초기화 
    - 인자 2개, 크기랑 개수
- realloc
    - 이미 할당된 메모리 블록의 크기를 변경할 때 사용
    -  새로운 크기만큼 메모리 크기를 조정하며, 기존 메모리 내용을 유지
- free
    - 해당 주소를 가지는 동적 메모리 해제

#### 문제
- 동적으로 할당 가능한 메모리 영역을 관리해야 함
- 어떻게 관리해야 하는가?
- 단편화 문제를 고려해야 함
    - 단편화: 빈 공간은 있는데 들어갈 수 없는 현상

#### 해결
- 고정 크기의 블록 단위로 메모리를 할당하면 단편화 문제를 최소화 할 수 있음

```c
#include <stdio.h>

int main() {
	
	int array[2]; // 스택 (자동)
	int *array2 = malloc(sizeof(int)*2); // 힙 (동적)
	int *array3 = calloc(2, sizeof(int)); // 힙 (동적)
    int *temp = realloc(array2, sizeof(int)*10);
    if (temp == NULL) {
        free(array3);
        free(array2);
        return 1;
    }
    array2 = temp;
    
    free(array2);
    free(array3);
}

```

### 균형 이진 탐색트리
BST에서 편향 트리가 발생하여 성능을 저하시키는 문제를 해결하기 위해 높이의 균형을 맞춰주는 트리. 

종류에는 AVL, RB-Tree, B-트리 등 있음.

### 프로시저
#### 왜 함수를 사용해야하는가?
- 함수를 사용하지 않으면, 프로그램 실행중 생성되는 모든 변수가 메모리, 레지스터에 올라가야 하나?
- procedure 단위로 일을 쪼개면, 현재 작업중인 데이터만 적재해서 사용할 수 있음
    - 메모리 효율적으로 사용
#### 왜 스택 자료구조를 사용하는 걸까?
- 이전에 작업하던 일로 돌아가기 위해서!
- 일이란건 해치워야 하는 것. 남아 있으면 안됨. 쌓인 것들을 위에서 부터 차곡 차곡 해내고 다 없어지면 할 일이 다 끝난 것.
- 프로그램도 마찬가지로 프로시저를 쌓고 해치우고 하는 과정을 거치면서 작업을 수행함.
- 일을 하는 과정에 발생하는 데이터를 적재하기에 스택 자료구조가합 적합함
- 어떤 일을 하던 중, 필요한 추가 일을 하기 위해 새로운 일을 쌓고, 이 일에 필요한 데이터를 적재하고, 다시 이 일이 끝나면 이전 일로 돌아가서 일을 마져 하고,
- 이 과정이 스택의 동작 방식과 같음 
- 그래서 스택을 쓰는 것

### array allocation and access
#### 컴퓨터의 메모리 접근 방법
- 시작 주소 + 인덱스 * (사이즈)
- 컴퓨터의 메모리 접근 방법 
#### 왜 저렇게 하면 빠르게 접근이 되지? 
- 주소를 안다고 해서 그 주소로 순간이동이 되는건 아니잖아?
- 메모리는 순간이동이 된다.
    - 랜덤 엑세스 
    - 하드웨어 매직!!!!

### relocatable object files
#### "왜 재배치 가능" 이라는 표현을 사용한거지?
- 심볼(가짜주소)로 미리 빌드를 해놓고, 나중에 링킹할 때 재배치 하기 때문
- 이렇게 하면 파일이 변경되어도 해당 파일과 연관된 파일만 수정하면 됨. 
- 전체 재 배치 필요 X. 해당 파일과 연관된 파일과의 관계만 수정하면됨

#### 이렇게 하면 뭐가 좋아? 왜 이렇게 해?
- 모듈화된 개발 (큰 프로그램을 모듈화 가능)
- 주소가 바뀔 가능성이 있음(파일의 변경 등). 
- 그래서 임시로 "심볼"로 서로 참조하고, (관계만 설정)
- 링킹 과정에서 재배치하면서 실제 주소를 할당 (상대 주소 설정)

### loading executable object files
- 운영체제가 실행할 수 있는 형태의 목적파일(바이너리 파일)
- 운영체제 마다 다를 수 있음
- 해당 운영체제에서 실행 가능한 형태로 목적파일을 만들어야 하는데, 그래서 컴파일러가 운영체제에 맞게 파일을 만들어줘야 함

### exceptions
- 프로그램 실행 중에 발생하는 비정상적인 상황으로, 일반적으로 프로그램의 정상적인 흐름을 방해합니다. 
- 인터럽트, 트랩, 오류, 중단

### signals

- 프로세스에 신호를 주는 방법
- 예시
    ```c
    #include <signal.h>
    #include <stdio.h>
    #include <unistd.h>

    void handler(int signum) {
        if (signum == SIGUSR1) {
            printf("Received SIGUSR1 signal!\n");
        }
    }

    int main() {
        signal(SIGUSR1, handler);  // SIGUSR1에 대한 핸들러 설정
        while (1) {
            printf("Running...\n");
            sleep(1);
        }
        return 0;
    }
    ```

- 터미널에서 아래 명령어 실행시키면 Handler 실행됨
    ```sh
    $ kill -SIGUSR1 {프로세스ID}
    ```



---
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