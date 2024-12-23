# 학습 내용
- 자료구조와 알고리즘
    - 비선형 자료구조
    - BFS, DFS
    - 최단 경로
    - 최소 신장 트리
    - 위상 정렬

# 퀴즈
## 캐시 메모리를 사용하면 컴퓨터 시스템의 성능이 왜 향상되는지 지역성(locality) 개념을 포함하여 설명
프로그램이 메모리에 접근할 때, 특정 시간 또는 특정 메모리 공간을 집중적으로 사용하는 경향이 있다. 이를 각각 시간적 지역성, 공간적 지역성이라고 한다. 
- 시간적 지역성
    - 한 번 접근된 데이터는 가까운 미래에 다시 접근될 가능성이 높다는 원리
    - 루프 내에서 반복적으로 사용되는 변수는 시간적 지역성의 좋은 예
- 공간적 지역성
    - 메모리의 주소에 접근한 후, 그 주변 주소에 있는 데이터에 접근될 가능성이 높다는 원리 
    - 배열이나 연속적인 메모리 블록에 접근할 때 종종 발생
캐시 메모리는 이러한 지역성의 원리를 활용하여 자주 사용되거나 연속적으로 사용될 가능성이 높은 데이터를 미리 캐시에 저장한다. 이로 인해 cpu는 필요한 데이터를 캐시에서 빠르게 찾을 수 있다. 

## BFS와 DFS 코드 구현 

```python
def dfs(graph, start_node):
    visit = []
    stack = []
    stack.push(start_node)
    while stack:
        node = stack.pop()
        if node not in visit:
            visit.append(node)
            stack.push(graph[node])
    return visit
```

```python
def bfs(graph, start_node):
    visit = []
    queue = []
    queue.append(start_node)
    while queue:
        node = queue.pop()
        if node not in visit:
            visit.append(node)
            queue.extend(graph[node])
    return visit
```

## 프로세스와 쓰레드의 차이

### 정의 
- 프로세스
    - 독립적으로 실행되는 프로그램의 인스턴스. 자체적인 주소 공간, 메모리, 데이터 스택 및 다른 시스템 자원을 갖는다.
- 쓰레드
    - 프로세스 내부의 실행 흐름 단위. 프로세스의 자원과 주소 공간을 공유하면서 실행된다.

### 차이 
- 자원 공유
    - 프로세스
        - 각 프로세스는 독립적인 메모리 공간과 시스템 자원을 가짐. 프로세스간 공유는 IPC(Inter-Process Communication) 메너키즘을 통해 이뤄짐
    - 쓰레드
        - 같은 프로세스 내 쓰레드들은 코드, 데이터 및 시스템 자원을 공유함
