# Gradle

- [실리콘밸리 엔지니어와 함께하는 Gradle](https://www.inflearn.com/course/%EC%8B%A4%EB%A6%AC%EC%BD%98%EB%B0%B8%EB%A6%AC-%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4-%ED%95%A8%EA%BB%98%ED%95%98%EB%8A%94-gradle/dashboard)
- [Why Gradle?](https://docs.gradle.org/current/userguide/userguide.html#why_gradle)

## 1. Intro

### 그레이들?
- 빌드 자동화 도구
- 빌드, 테스트, 배포와 같은 작업들을 미리 정의해 둔 설정 파일을 통해 자동으로 실행해주는 도구
- 코드 컴파일, 바이너리 패키징, 테스트 실행 등의 작업을 실행해줌

### 선언형 빌드 스크립트 (Declarative Build Scripts)
- 빌드 과정에서 수행할 작업을 절차적으로 지정하는 대신,
- 빌드 결과가 어떤 모습이어야 하는지에 초점을 맞추어 정의하는 스크립트 방식
    - how 보다는 what에 집중
- 스크립트 설정 파일에는 빌드 프로세스의 각 단계, 빌드 과정에서 수행할 작업(Task)을 정의
- 의존성이나 task를 쉽게 정의할 수 있음!
- 그루비나 코틀린과 같은 DSL을 사용해서 스크립트 구성

### DSL (Domain-Specific Language)
- 도메인 특화 언어
- 모든 범위의 일을 수행하는 것이 아니라, 특정 문제나 작업에 최적화된 언어
- Gradle 설정 파일(groovy, kotlin을 사용하는)은 `빌드 및 배포 관리`라는 도메인에 특화된 DSL
- 또 다른 DSL로는 다음과 같은 것들이 있음 
    - SQL: 데이터베이스 질의과 관리에 특화된 언어
    - HTML: 웹 페이지의 구조와 콘텐츠를 정의하는 데 특화된 마크업 언어
    - CSS: HTML 문서의 스타일링에 특화된 언어
- groovy는 `build.gradle`, kotlin은 `build.gradle.kts`

### 의존성 관리 Dependency Management
- maven central이나 Jcenter 같은 레포지토리를 통해 외부 의존성들을 명시하고, 버전을 관리할 수 있음
- 자동으로 의존성을 해결해주고, 로컬에 캐싱해준다. -> 성능 향상, 안정성 향상

### Incremental Builds
- 지난 빌드와 다른 부분만 빌드
- checksums이나 timestamps를 확인
- 빌드 속도 향상!

### 멀티 프로젝트 빌드
- 여러 개의 서브 프로젝트 또는 모듈들을 관리할 수 있음
- 각 서브 프로젝트는 각각의 의존성이나 task들을 가질 수 있음

### 커스텀과 확장성
- 풍부한 API와 플러그인 생태계 가짐
- tasks를 커스텀하고 플러그인을 개발할 수 있음
- java 컴파일이나 test running, JAF 패키징 같은 일반적인 task를 위한 플러그인들 존재
- 이러한 유연성으로 웹이나 안드로이드 앱이나 다양한 프로젝트 타입들을 지원함

### Build Scans
- 빌드에 대한 인사이트 제공
- 퍼포먼스나 의존성, task 실행 시간등의 정보 제공

### CI/CD 연동
- 젠킨스와 같은 툴에 gradle을 쉽게 설치하고 사용 가능

### 사용하기
- gradle은 jvm 환경에서 돌아가기 때문에 JDK 필요
- brew install gradle로 쉽게 설치 가능

---

---


## 
