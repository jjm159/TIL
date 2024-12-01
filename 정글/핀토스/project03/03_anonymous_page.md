# Anonymous Page
- 어노니모스 페이지가 뭔데?
    - 파일이나 디바이스와 연결되지 않은 메모리 페이지
    - 스택이나 힙과 같이 특정 파일 소스가 없는 메모리 영역에서 사용
- 구조체 `anon_page`
    - vm/anon.h에 정의
    - Anonymous Page의 상태를 저장하기 위해 필요한 멤버들을 추가해야 해.
- 지연로딩
    - 페이지는 미리 할당
    - 실제 메모리 로드(프레임 매핑)는 페이지가 필요한 순간(page fault)

## Page Initialization with Lazy Loading
- Lazy Loading
    - 메모리 로딩을 필요할 때까지 지연하는 설계
    - 페이지는 할당되지만, 실제 물리 프레임이나 contents는 아직 로드 안됨
    - 콘텐츠는 page fault가 발생하면 로드
- 페이지 초기화 flow
    - `vm_alloc_page_with_initializer`
        - kernel에 new page request 요청이 오면 실행
        - new page를 하나 초기화
            - page 구조체 할당
            - page type에 따른 initializer 할당
        - 그리고 다시 user program으로 return
    - user program이 실행되면
        - 페이지 폴트가 실행됨
            - 가지고 있다고 생각하는 (가상 메모리 주소) 페이지에 접근을 시도
            - 하지만 해당 페이지에는 아무 contnets가 없음
    - fault handling procedure
        - `uninit_initilaize` 실행
        - `anon_initializer` 또는 `file_backed_initializer` 실행
- page fault 처리
    - 지연 로딩이 필요한 경우, 이전에 설정된 초기화자가 호출되어 페이지를 메모리에 로드
- 페이지 life cycle
    - 초기화 -> (page_fault -> lazy_load -> swap_in -> swap_out -> ...) -> destroy
    - 사이클 전환시, page의 type에 따라  the required procedure가 다를 수 있음
    - 이 프로젝트에서는 타입에 맞는 transition processes를 구현하는 게 목적

## Lazy Loading for Executable
- 프로세스가 실행될 때, 지금 필요한 부분만 메인 메모리에 올린다.
- 바이너리 image를 한번에 메모리에 올리는 즉시 로딩에 비해 오버헤드를 줄인다.
- `VM_UNINIT` 타입
    - 지연 로딩을 위해 도입된 페이지 타입
    - 모든 페이지는 초기에 VM_UNINIT 상태로 생성
    - `struct uninit_page` 구조체!
        - `include/vm/uninit.h`
    - 함수
        - `uninit_new`
        - `uninit_initialize`
        - `uninit_destroy`
- 페이지 폴트
    - 아직 로드되지 않은 페이지 접근 시 발생
    - 핸들러 호출
        - page_fault(userprog/exception.c)
            - 페이지 폴트 발생 시 호출 (인터럽트 핸들러)
        - vm_try_handle_fault (vm/vm.c)
            - `valid page fault`
                - 잘못된 주소에 접근하는 진짜 페이지 폴트
            - `invalid page fault` (`bogus`)
                - 아직 로드를 하지 않은 안 진자 페이지 폴트
                - 이 경우에는 contents를 load하고 user program에게 제어 넘김
    - 유효하지 않은 페이지 폴트 3가지 (bogus)
        - `lazy-loaded page`
            - `vm_alloc_page_with_initializer` 여기서 할당했던 페이지 초기화 함수를 호출
                - anon_initializer
                - file_backed_initializer
            - `lazy_load_segment`(userprog/process.c) 를 구현해야 함!
        - `swaped-out page`
        - `write-protected page`

#### Implement `vm_alloc_page_with_initializer()`
- vm_alloc_page_with_initializer 구현
    ```c
    bool vm_alloc_page_with_initializer(enum vm_type type, void *va, 
            bool writable, vm_initializer *init, void *aux);
    ```
    - vm_type에 따라 초기화 함수 할당
    - swap_in 핸들러
        - 페이지 타입에 따라 페이지 자동으로 초기화
        - init, aux를 사용
    - 페이지 구조체 생성
        - 보조 페이지 테이블에 새로 만든 페이지를 insert
- page fault handler
    - call chain에 따라 진행
    - 최종적으로 swap_in을 호출하여 uninit_initialize 호출
    - uninit_initialize를 수정해야 할 수 있음
- uninit_initialize
    ```c
    static bool uninit_initialize (struct page *page, void *kva);
    ```
    - 첫 번째 페이지 폴트에서 페이지 초기화
    - 
- `vm_anon_init`과 `anon_initializer` 수정 (vm/anon.c)
- `void vm_anon_init (void)`;
    - Anonymous 페이지 서브 시스템을 초기화 
    - 익명 페이지와 관련된 설정 진행
- `bool anon_initializer (struct page *page,enum vm_type type, void *kva);`
    - page->operations에 대해 anonymous 페이지의 핸들러 먼저 설정
    - anon_page에 필요한 정보 업데이트
    - VM_ANON 페이지 초기화에 사용

#### Implement `load_segment and lazy_load_segment` in userprog/process.c
- 실행 파일에서 세그먼트를 로드하는 작업
- 모든 페이지는 지연 로드(lazy loading) 방식으로 처리
- 커널이 페이지 폴트를 가로챌 때만 메모리에 로드
- load_segment 수정
    ```c
    static bool load_segment(struct file *file, off_t ofs, uint8_t *upage,
            uint32_t read_bytes, uint32_t zero_bytes, bool writable);
    ```
    - 현재 동작
        - 루프를 통해 읽어야 할 파일의 바이트 수(read_bytes)와 0으로 채워야 할 바이트 수(zero_bytes)를 계산
        - `vm_alloc_page_with_initializer`를 호출하여 `처리 대기 상태(pending)`의 페이지 객체를 생성
    - 수정 사항
        - 페이지를 초기화하기 위한 aux 값을 설정
        - aux는 바이너리 로드에 필요한 정보를 포함하는 구조체로 설정
        - `vm_alloc_page_with_initializer` 호출 시 `lazy_load_segment`를 초기화 함수로 전달
- lazy_load_segment
    ```c
    static bool lazy_load_segment(struct page *page, void *aux);
    ```
    - 동작
        - lazy_load_segment는 vm_alloc_page_with_initializer 호출에서 네 번째 인자로 제공
        - 페이지 폴트가 발생할 때 커널은 이 함수(초기화 함수)를 호출하여 세그먼트를 로드
    - 입력
        - page 구조체: 해당 페이지에 대한 정보를 포함
        - aux: load_segment에서 설정한 추가적인 데이터
    - 작업
        - aux를 사용해 파일에서 읽어올 데이터를 찾아야 함
        - 파일에서 세그먼트를 읽어 페이지에 로드
        - 지연 로드(lazy loading) 방식으로만 데이터를 메모리에 로드

#### You should adjust the `setup_stack` in userprog/process.c to fit stack allocation into the new memory management system
- 스택 페이지 초기화
    - setup_stack 수정
    - 첫 번째 스택 페이지는 지연 로드 없이 즉시 할당 및 초기화
    - 명령어 인자를 사용해 초기화
- 스택 페이지 식별
    - 스택 페이지를 식별하기 위해 vm_type의 보조 마커(예: VM_MARKER_0) 사용 가능
- 페이지 폴트 처리 수정
    - vm_try_handle_fault 수정
    - 보조 페이지 테이블에서 페이지를 검색하기 위해 spt_find_page 호출
- 결과
    - 모든 요구 사항 구현 후, 프로젝트 2의 모든 테스트(fork 제외)를 통과해야 함

## Supplemental Page Table - Revisit

#### Implement `supplemental_page_table_copy` and `supplemental_page_table_kill` in `vm/vm.c.`
- `supplemental_page_table_copy`
    ```c
    bool supplemental_page_table_copy (struct supplemental_page_table *dst,
        struct supplemental_page_table *src);
    ```
    - 부모 프로세스의 보조 페이지 테이블을 자식 프로세스로 복사
    - 모든 페이지를 초기화 상태로 할당하고 즉시 클레임(claim)
- `supplemental_page_table_kill`
    ```c
    void supplemental_page_table_kill (struct supplemental_page_table *spt);
    ```
    - 프로세스 종료 시, 보조 페이지 테이블의 모든 자원 해제
    - 각 페이지 엔트리에 대해 destroy(page) 호출
    - 실제 페이지 테이블과 물리 메모리 정리는 호출자에 의해 처리됨

## Page Cleanup
#### Implement `uninit_destroy` in `vm/uninit.c` and `anon_destroy` in `vm/anon.c`
- 이 함수들은 초기화되지 않은 페이지(uninitialized page)를 삭제하는 작업을 처리
- 초기화되지 않은 페이지는 다른 페이지 객체로 변환되더라도, 프로세스 종료 시 여전히 남아 있을 수 있음
- `uninit_destroy`
    ```c
    static void uninit_destroy (struct page *page);
    ```
    - page 구조체에서 사용 중인 자원을 해제
	- 페이지의 VM 타입을 확인하고 그에 맞게 처리
    - 현재는 익명 페이지(anonymous page)만 처리
    - 나중에 이 함수에서 파일 기반 페이지(file-backed page)를 정리하도록 확장해야 함
- `anon_destroy`
    ```c
    static void anon_destroy (struct page *page);
    ```
    - 익명 페이지에서 사용 중인 자원을 해제
    - page 구조체 자체를 명시적으로 해제할 필요 X
    - 이 작업은 호출자가 처리
---
