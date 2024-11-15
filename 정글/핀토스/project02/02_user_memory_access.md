# User Memory Access

- [깃북](https://casys-kaist.github.io/pintos-kaist/project2/user_memory.html)

## What?
- 유저가 invalid pointer로 접근시 유저 프로세스를 종료시켜야 한다.

## How?
- 앞서 project2의 Introduction에 이와 관련되어 힌트가 있었다.
- 올바르게 동작하게 만드는 두 가지 방법 (둘 중 하나로 구현하면 된다.)
    - 포인터 유효성 체크 
        - thread/mmu.c, include/threads/vaddr.h 함수 참조
    - KERN_BASE 아래를 가리키는지 확인
        - userprog/exception.c의 page_fault() 수정

- vaddr.h
    - 여기에 is_user_vaddr 라는 함수가 있다. 유저의 가상 메모리 공간에 접근하는 건지 확인하는 함수다.
- mmu.c
    - 그리고 이걸 추가로 사용한다.
    - 가상 메모리 주소에 제대로 매핑된 주소인지 확인한다.
    ```c
    /* Looks up the physical address that corresponds to user virtual
    * address UADDR in pml4.  Returns the kernel virtual address
    * corresponding to that physical address, or a null pointer if
    * UADDR is unmapped. */
    void *
    pml4_get_page (uint64_t *pml4, const void *uaddr) {
        ASSERT (is_user_vaddr (uaddr));

        uint64_t *pte = pml4e_walk (pml4, (uint64_t) uaddr, 0);

        if (pte && (*pte & PTE_P))
            return ptov (PTE_ADDR (*pte)) + pg_ofs (uaddr);
        return NULL;
    }
    ```
### Result
```c

static bool is_valid_address(void *addr); // 선언

// project 2
// 유저 프로그램의 주소 접근이 유효한지 확인
// 시스템 콜 호출시 사용
static bool 
is_valid_address(void *addr)
{
    if (addr == NULL)
        exit(-1);
    if (!is_user_vaddr(addr))
        exit(-1);
    if (pml4_get_page(thread_current()->pml4, addr) == NULL)
        exit(-1);
}
```