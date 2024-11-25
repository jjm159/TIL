# 프록시 과제

## 개요
- 웹 브라우저에서 웹사이트를 접근할 때 중간에 프록시 서버를 거칠 수 있다.
- 이번 과제는 이 프록시 서버를 구축하는 거고, 중간에 http 요청을 바꿔서 목적지 서버에 전달하고 그 응답을 받아 클라이언트에게 전달한다.
- 더 나아가서 클라이언트의 요청을 여러 스레드를 생성하여 비동기 처리하고 동시에 처리할 수 있도록 하고,
- 마지막에는 서버의 응답을 메모리에 캐시 처리 해서 같은 요청에 대해 빠르게 응답하도록 처리한다.
- 이번 과제를 수행하면서 자연스럽게 브라우저의 http 요청과, 서버의 http 응답을 중간에서 분석했고, http에 더 잘 이해하게 되었다.

---

## Part I: Implementing a sequential web proxy

- proxy 서버는 이전 tiny 서버와 다르게 main 함수 하나만 덩그러니 놓여있다.
- 하지만, 이전에 tiny 서버와 echo 서버를 통해 사용했던 함수들을 잘 조합하면 쉽게 구현할 수 있다. 

### 기본 구조
- 기본 구조는 tiny 서버와 거의 똑같다. 
- 소켓을 열고, 요청이 오면 client의 fd를 받아서 네트워크 IO 버퍼에 있는 값들을 읽어온다.
- 다른 점은 http 요청을 파싱해서 문제에서 요구하는 대로 start line과, header에 있는 값들을 바꿔서 서버에 보내야 한다.
- 서버의 응답은, 아직은 바로 클라이언트에 보내줘도 된다. 캐싱을 아직 처리하기 전이니까.

### 요구 사항
- 클라이언트에서 보낸 요청을 그대로 서버에 전달하는 게 아니라, 문제에서 요구하는대로 수정해서 보내야 한다. 
- start line 
    - https://와 host, port까지 포함된 uri에서 path만 전송
        ```
        GET /hub/index.html HTTP/1.0
        ```
        - 이런식으로 
    - http 버전을 1.0으로 
        - 1.1도 1.0으로 바꿔서 전송해야 함
- 헤더 
    - host 받드시 포함. 클라가 보낸게 있으면 클라가 보낸 걸로 전송
    - User-Agent 포함
    - Connection: close
    - Proxy-Connection: close

### 과정
- 목표
    - 클라이언트의 요청 읽기
    - request라는 string 포인터 변수에, 문제의 조건에 맞게 string 할당
    - 서버에 전송
    - 받은 데이터를 그대로 클라이언트에 전송
- 그대로 보내면 안되기 때문에 line by line으로 읽어들이면서 문제의 조건을 채워주어야 한다. 
- 처음 줄은 start line이기 때문에 쉽게 sscanf로 method, path, version을 읽어왔다.
- 이를 위해 Rio_readlineb를 사용해서, 줄 단위로 buf에 읽어들였고, 이 string에 어떤 값이 포함되어 있는지 확인해서 문제의 조건에 맞게 header 값을 변경해 주었고, 필요한 헤더는 추가로 넣어주었다.
- 다른건 어렵지 않았는데, uri에서 host와 port 그리고 path를 parsing 하는 부분이 c언어로는 쉽지 않았다.
- strncmp, strchr, strncpy 등의 함수들을 조합해서 다행히 어찌어찌 구현을 완료 했다.
- 이렇게 추출한 hostname과 portnumber로 서버에 연결하고, 앞서 완성된 request 문자 데이터를 서버에 전송한다.
- 받은 데이터는 그대로 클라이언트에 전송하면 끝난다.

### 구현 몇 가지
- line by line으로 request에 string을 할당할 때 sprintf를 사용하는데, 반환값을 사용해서 다음에 쓸 곳을 계속해서 업데이트 해야 한다.
    ```c
    requestIdx += sprintf(request + requestIdx, "%s %s %s\n", method, path, "HTTP/1.0");
    ```
- url 파싱할 때, strncmp로 필요 없는 부분을 점프 
    - http 같은 프로토콜은 pass
    ```c
    const char *url_ptr = uri;

    if (strncmp(uri, "https://", 8) == 0)
    {
        url_ptr += 8; // "https://" 뒤로 포인터 이동
    }
    else if (strncmp(uri, "http://", 7) == 0)
    {
        url_ptr += 7; // "http://" 뒤로 포인터 이동
    }
    ```
    - 이후, 필요한 부분의 pointer를 찾아서 추출
    ```c
    const char *slash_pos = strchr(url_ptr, '/');

    if (slash_pos)
    {
        strcpy(path, slash_pos); 
    }
    else
    {
        strcpy(path, "/"); // 기본 경로는 "/"
    }
    ```
- 서버의 응답 읽어오기
    - 클라이언트의 request 처럼 line by line으로 읽을 필요가 없다. 그대로 전송하면 된다.
    ```c
    char buf[MAXLINE];
    size_t count = 0;
    while ((count = Rio_readlineb(&rio, buf, MAXLINE)) != 0)
    {
        Rio_writen(clientfd, buf, count);
    }
    ```

---

## Part II: Dealing with multiple concurrent requests

- csapp 책에 나와 있는 thread 기본 코드를 그대로 사용하면 쉽게 적용할 수 있다. 보너스 점수 같은 느낌으로 빠르게 완료하였다.
- doit이라는 함수로 클라이언트의 요청을 처리하는 작업들을 모아두었기 때문에 빠르게 적용할 수 있었다.

```c 
int main {

    while (1) {
        // ..

        Pthread_create(&tid, NULL, doit, &connfd);

        // ..
    }
}

void doit(void *vargp)
{

  Pthread_detach(pthread_self());

  int clientfd = *(int *)vargp;

  // ... 같음

}

```

- 주의 할 점은, close 부분이다.
- thread 마다 소켓이 연결되어 있기 때문에 close 처리를 스레드 핸들러가 해주어야 한다.

---

## Part III: Caching web objects

- 아주 어렵게 생각할 필요가 없다. 기본적으로 네트워크 통신보다, 메모리에서 불러오는게 아주 많은 속도 차이가 난다.
- key의 hash 처리를 통해 O(1)으로 캐시 데이터에 접근하고, 이런 처리는 굳이 안해도, 지금은 괜찮다.
- 연결 리스트를 사용해서 구현했다. 
- key는 uri를 사용했고, response를 그대로 value에 할당하였다.

### 구현

- cache 구조체
    ```c
    typedef struct cache_item_t {
    int size;
    char *key; // uri
    void *value;
    struct cache_item_t* next;
    } cache_item_t;

    typedef struct {
    int total_size;
    cache_item_t * head;
    cache_item_t * tail;
    } cache_t;
    ```

- create_cache
    ```c
    cache_t* create_cache()
    {
        cache_t * new_cache = malloc(sizeof(cache_t));
        new_cache->total_size = 0;
        new_cache->head = NULL;
        new_cache->tail = NULL;
        return new_cache;
    }
    ```

- getFromCahce
    ```c
    cache_item_t *getFromCahce(cache_t *cache, char * uri)
    {
    printf("%s\n", uri);

    cache_item_t * current = cache->head;

    while (current != NULL)
    {
        char *key = current->key;

        if (!strcmp(key, uri)) {
        return current;
        }
        current = current->next;
    }
    return NULL;
    }
    ```

- updateToCache
    ```c
    int updateToCache(cache_t *cache, char *uri, char *response, int size)
    {
        if (size > MAX_OBJECT_SIZE) {
            return;
        }

        char* new_uri = malloc(strlen(uri)+1);
        strcpy(new_uri, uri);

        char* new_response = malloc(size);
        memcpy(new_response, response, size);

        while (is_over_max_size(cache, size))
        {
            cache_item_t *current = cache->head;
            cache->head = current->next;
            cache->total_size -= current->size;
            Free(current->key);
            Free(current->value);
            Free(current);
        }

        cache_item_t * new_item = (cache_item_t *) malloc(sizeof(cache_item_t));
        new_item->key = new_uri;
        new_item->value = new_response;
        new_item->size = size;
        new_item->next = NULL;
        
        if (cache->head == NULL) {
            cache->head = new_item;    
        } else {
            new_item->next = cache->head;
            cache->head = new_item;
        }

        cache->total_size += size;
    }
    ```


### 주의
- 힙 메모리를 할당 받아서 저장하기 때문에 메모리 해제 처리에 유의해야 한다. 
- 가장 좋은 방법은, 메모리를 할당한 곳에서 해제하는 규칙에 따르는 것이다.
- uri나 response를 doit에서 malloc했다면, 해제도 반드시 doit에서 하게 만드는 게 좋다.
- 캐시를 처리하는 함수에서 이 메모리를 해제해버리면 생각해야 하는 경우의 수가 많아진다.
    - 캐시 처리 함수에서 메모리를 해제한 경우 -> doit에서는 하지 않아야 한다.
    - 캐시 처리 함수에서 메모리 해제를 하지 않은 경우 -> doit에서 해야 한다.
- 그래서 캐시 데이터로 적재할 때에는 새로운 메모리를 할당받아서 데이터를 복사해서 저장하면 문제는 단순해진다.

---

## 트러블 슈팅

### 테스트 실수
- 웹 -> 프록시 접근 -> tiny 라고 생각했지만, 
- 프록시 입장에서 웹이 보낸 host로 요청을 보내면 당연히 프록시로 보내게 됨
- 웹에서 tiny로 보내도, proxy를 거치도록 브라우저 설정이 필요
- 이게 귀찮으면, curl로도 이러한 테스트를 실행할 수 있음

```shell
$ curl -v --proxy localhost:6000 localhost:8080/tiny.c
```
