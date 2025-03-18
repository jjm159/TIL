# 8주차

## 1. 응용 프로그램을 구현할 때 multiprocess 와 multithread 중 하나를 선택하는 기준은 어떤 것이 있는지 몇 가지 제시하세요. (1점)
#### 선택 기준
- ● 안정성 vs. 자원 사용: 
    - 시스템의 안정성이 매우 중요한 경우, 멀티프로세스가 선호됩니다. 
    - 리소스가 제한적인 환경에서는 멀티스레드가 더 효율적일 수 있습니다.
- ● 구현의 복잡성: 
    - 스레드는 공유 메모리로 인해 동기화 문제가 복잡해질 수 있으므로,
    - 개발자의 동시성 제어에 대한 이해도가 중요합니다.
- ● 응답 시간: 
    - 멀티스레드는 컨텍스트 스위칭이 빠르기 때문에, 
    - 더 빠른 응답 시간을 요구하는 경우 유리할 수 있습니다.
- ● 플랫폼 및 언어 지원: 
    - 사용 중인 프로그래밍 언어나 플랫폼이 멀티스레드 또는 
    - 멀티프로세스 중 어느 쪽을 더 잘 지원하는지도 중요한 요소가 될 수 있습니다.

## 2. 데드락을 해결하기 위한 전략을 두 가지 이상 설명하십시오. (1점)
#### 데드락 해결 전략
- 1. 데드락 예방(Deadlock Prevention):
    - 이 접근법은 데드락 발생을 원천적으로 차단합니다. 
    - 데드락이 발생하는 네 가지 필수 조건(상호 배제, 점유와 대기, 비선점, 순환 대기) 중 
    - 적어도 하나를 제거함으로써 데드락을 방지합니다. 
    - 예를 들어, 비선점 조건을 제거하면 어떤 리소스도 필요할 때 
    - 다른 프로세스에 의해 선점될 수 있으며, 이는 데드락을 방지할 수 있습니다.
- 2. 데드락 회피(Deadlock Avoidance):
    - 데드락 회피는 시스템이 데드락 상태로 진입하는 것을 회피하는 전략입니다. 
    - 이를 위해 시스템은 리소스 할당 결정 시 데드락의 가능성을 고려합니다. 
    - 가장 유명한 예는 뱅커스 알고리즘(Banker's Algorithm)으로,
    - 이는 프로세스에 리소스를 할당하기 전에 안전 상태를 유지할 수 있는지 확인합니다. 
    - 만약 할당으로 인해 데드락이 발생할 위험이 있다면, 리소스는 할당되지 않습니다.
- 3. 데드락 탐지 및 회복(Deadlock Detection and Recovery):
    - 이 전략은 시스템이 데드락을 탐지하고, 이를 해결하기 위한 조치를 취하는 것을 포함합니다. 
    - 데드락 탐지는 주기적으로 리소스 할당 그래프를 검사하여 
    - 순환 대기 조건을 찾는 것으로 이루어질 수 있습니다. 
    - 데드락이 탐지되면, 시스템은 프로세스를 중지하거나 리소스 할당을 롤백하여 데드락을 해결합니다.
- 4. 자원의 상호 배제 제거(Ignoring Mutual Exclusion):
    - 일부 경우에, 리소스의 상호 배제 조건을 제거할 수 있습니다. 
    - 예를 들어, 리소스가 복사가 가능하거나 공유가 가능한 경우, 
    - 여러 프로세스가 동시에 해당 리소스를 사용할 수 있습니다. 
    - 이러한 방식으로 상호 배제 조건을 제거함으로써 데드락 발생 가능성을 줄일 수 있습니다.

## 3. Semaphore와 Mutex의 특징과 주요 차이점은 무엇인가요? (1점)
- Semaphore: 
    - Semaphore는 공유 자원에 대한 접근을 제한하는 데 사용되며, 
    - 이는 특정 숫자로 초기화됩니다. 
    - 이 숫자는 동시에 해당 자원에 접근할 수 있는 스레드의 최대 수를 나타냅니다. 
    - Semaphore는 스레드가 자원을 사용할 때마다 감소하고, 자원을 해제할 때마다 증가합니다.
- Mutex (Mutual Exclusion): 
    - Mutex는 공유 자원에 대한 접근을 단일 스레드에게만 허용합니다.
    - 이는 주로 데이터의 무결성을 보호하기 위해 사용되며, 한 번에 하나의 스레드만이 공유 자원에 접근할 수 있도록 합니다. 
    - Mutex는 소유권 개념을 가지고 있어, 잠금을 건 스레드만이 잠금을 해제할 수 있습니다.
- 이 두 가지 메커니즘은 모두 동시성을 관리하고 데이터 무결성을 보장하는 데 필수적이지만, 
- 사용되는 상황과 목적에 따라 선택됩니다. 
    - Mutex는 보다 엄격한 제어가 필요할 때, 
    - Semaphore는 여러 자원에 대한 동시 접근을 허용할 때, 
    - 특히 Counting Semaphore는 자원의 수량이 제한되어 있을 때 유용합니다.

## 4. 다음 ANSI C 프로그램에서 출력되는 내용은 무엇인가요? (1점)
```c
#include <stdio.h>
int f(int x, int *py, int **ppz)
{
    int y, z;
    **ppz += 1;
    z = **ppz;
    *py += 2;
    y = *py;
    x += 3;
    return x + y + z;
}
int main()
{
    int c, *b, **a;
    c = 4;
    b = &c;
    a = &b;
    printf("%d\n", f(c, b, a));
    return 0;
}
```
- 19


## 5. 다음 C 코드에서 발생하는 메모리 누수(memory leak)를 찾고, 해결 방안을 제시하세요. (1점)
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int data;
    char *description;
} item;

item* create_item(int data, const char *desc) {

    item *new_item = (item *)malloc(sizeof(item));

    if (new_item == NULL) {
        return NULL;
    }
    
    new_item->data = data;
    new_item->description = (char *)malloc(strlen(desc) + 1);
    strcpy(new_item->description, desc);
    
    return new_item;
}

int main() {
    item *myItem = create_item(5, "Test Item");

    printf("Item: %d, Description: %s\n", myItem->data, myItem->description);
    
    // 다른 작업 수행

    free(myItem); // 메모리 해제
    return 0;
}
```
- 이 코드에서는 create_item 함수에서 item 구조체와 description 문자열에 대한 메모리를 할당합니다. 
- 그러나 main 함수에서는 오직 item 구조체에 대해서만 메모리를 해제하고 있고,
- description 필드에 대한 메모리 해제가 누락되어 있습니다. 
- 올바른 메모리 해제를 위해 main 함수에서 free(myItem->description);을 추가해야 합니다.

