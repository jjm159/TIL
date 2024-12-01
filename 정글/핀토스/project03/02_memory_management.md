# Memory Management
- 목적
    - 가상 페이지와 물리 프레임을 효츌적을 관리하는 것
- supplemental page table 구현
    - 어떤 메모리 영역이 사용중인지, 어떤 목적으로, 누구에게 등등의 정보
- page는 가상 페이지, frame은 물리 페이지

## Page Structure and Operations
#### page 구조체
- 가상 메모리 페이지 정보
- virtual address
    - 유저 공간의 가상 메모리 주소
- frame
- operations
- union
    - 서로 배타적이기 때문에 가능한 자료형
    - a uninit_page, anon_page, file_page, or page_cache 타입 끼리 배타적

#### Page Operations
- 페이지 유형에 따라 작업 방식이 달라짐
- `함수 포인터`를 사용하여 클래스 `상속` 개념을 구현
    - struct pag_operations

## Implement Supplemental Page Table
- 매핑 정보 외에 추가 메타 데이터를 저장하여 페이지 폴트 및 자원 관리 지원
- 가상 주소와 관련된 정보 저장

#### Implement supplemental page table management functions in vm/vm.c.
- `supplemental_page_table_init`
    - 보조 페이지 테이블 초기화
    - 프로세스 시작 및 포크 시 호출
- `spt_find_page`
    - 가상주소(va)에 해당하는 struct page를 보조 페이지 테이블에서 검색
    - 실패 시 NULL 반환
- `spt_insert_page`
    - 보조 페이지 테이블에 새로운 struct page 삽입
    - 동일한 가상 주소가 이미 존재하지 않는지 확인

## Frame Management
- 물리 메모리 관리
- 물리 페이지를 나타내는 구조체 struct frame 구현
```c
/* The representation of "frame" */
struct frame {
  void *kva; // 커널 가상 메모리 주소
  struct page *page;
};
```

#### Implement vm_get_frame, vm_claim_page and vm_do_claim_page in vm/vm.c.
- `vm_get_frame`
    - 사용자 풀에서 새 물리 페이지를 할당(palloc_get_page 사용)
    - 성공 시 프레임 구조체 생성 및 초기화 후 반환
    - 실패 시 `PANIC` 처리
- `vm_do_claim_page`
    - 페이지에 대해 물리 프레임 할당
    - MMU를 설정하고 페이지 테이블에 가상 주소와 물리 주소 매핑 추가
- `vm_claim_page`
    - 가상 주소(va)에 대한 페이지 클레임
    - vm_do_claim_page 호출로 작업 수행

