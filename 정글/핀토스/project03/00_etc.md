# etc

## hash
- 사용법
    - list와 유사하지만, 두 개의 함수를 init에 넣어줘야 함
- 왜?
    - hash는 key가 필요함
    - custum type(구조체, class)은 어떤 값을 hashing해서 key로 사용할 지를 개발자가 결정해 주어야 함
- pintos에서 key를 결정하는 방법
    - code
    ```c
    void
    supplemental_page_table_init (struct supplemental_page_table *spt UNUSED) {
        hash_init (&spt->spt_hash, page_hash, page_less, NULL);
    }

    unsigned
    page_hash(const struct hash_elem *p_, void *aux UNUSED)
    {
        const struct page *p = hash_entry(p_, struct page, hash_elem);
        return hash_bytes(&p->va, sizeof p->va);
    }
    ```
    - page_hash를 넘겨주는데, 이 함수가 key를 결정해줌
    - hash_bytes에 p->va를 줘서 key로 사용하도록 함
    - 이게 h->hash에 page_hash가 할당됨
    ```c
    bool
    hash_init (struct hash *h,
            hash_hash_func *hash, hash_less_func *less, void *aux) {
        h->elem_cnt = 0;
        h->bucket_cnt = 4;
        h->buckets = malloc (sizeof *h->buckets * h->bucket_cnt);
        h->hash = hash; // 여기
        h->less = less;
        h->aux = aux;

        if (h->buckets != NULL) {
            hash_clear (h, NULL);
            return true;
        } else
            return false;
    }
    ```
- 그리고 이게 실제로 어떻게 호출되는지
    ```c
    struct hash_elem *
    hash_find (struct hash *h, struct hash_elem *e) {
        return find_elem (h, find_bucket (h, e), e);
    }

    /* Returns the bucket in H that E belongs in. */
    static struct list *
    find_bucket (struct hash *h, struct hash_elem *e) {
        size_t bucket_idx = h->hash (e, h->aux) & (h->bucket_cnt - 1);
        return &h->buckets[bucket_idx];
    }
    ```
    - h->hash를 호출하면, page_hash가 호출됨
    - 지정한 key가 반환됨

## Class 상속 개념 구현
- operations
- 컴파일러 단에서 다형성을 지원하지 않지만, C언어는 무엇이든 만들 수 있기 때문에 다형성의 동작을 쌩으로 구현할 수가 있다.

## frame에서 kva를 사용하는 이유
- kernel pool과 user pool의 초기화
- 물리메모리를 os 부팅시 pool로 초기화
- pool은 커널의 주소 공간에 있음. 즉 kernel의 veirtual address로 관리됨. 
- 이게 kva의 정체
- frame에 할당되는 물리 메모리의 주소가 kva로 이름이 붙여진 이유는,
- 커널 주소 공간에서 관리되는 물리 메모리에 접근하려면, 커널의 주소 공간의 주소를 알고 있어야 하기 때문

## 스택이 쌓이는 방향
- 스택이 쌓이는 방향이 반대
- 새로운 데이터를 할당하기 전에 stack bottom을 빼서 옮겨줘야 함
```c
static bool
setup_stack (struct intr_frame *if_) {
	bool success = false;

	// 스택의 쌓이는 방향은 반대
	// 낮은 주소로 먼저 bottom을 옮겨서, bottom에 page를 할당
	// 할당하고 옮기는게 아니라 먼저 옮기고 할당해야 함!!!
	void *stack_bottom = (void *) (((uint8_t *) USER_STACK) - PGSIZE);
```

---

# 트러블 슈팅

## static
```c
static struct lock filesys_lock;
```
- syscall.h에서 static으로 선언하여 process.c에서 잘못 접근 됨
- 컴파일 오류가 나지 않고 쓰레기 값이 할당되어 무한 대기 문제 발생
