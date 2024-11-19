# Build Lifecycle
-[Docs: Build Lifecycle](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:build_phases)

---

## 개요
- build를 작성하면서 task를 정의하고, task 간 의존성을 정의
- gradle은 tasks들이 이 의존성에 따라 실행되도록 순서를 보장해줌
- 작성한 build script와 플러그인들은 이러한 task dependency graph 를 구성해줌.
- build, assemble, createDocs 같은 tasks를 포함하는 build script에서, build -> assemble -> createDocs 와 같은 순서로 실행되도록 할 수 있음

## Task Graphs
- 어떤 task를 실행하기 전에 gradle은 task graph를 만든다.
- 빌드 중인 모든 프로젝트에서 tasks는 유향 비순환 그래프(DAG, Directed Acycle Graph)를 생성함
- plugins과 build scripts는 모두 task graph에 영향을 준다. 
    - [task dependency mechanism](https://docs.gradle.org/current/userguide/tutorial_using_tasks.html#sec:task_dependencies)
    - [annoted inputs/outputs](https://docs.gradle.org/current/userguide/incremental_build.html#sec:task_inputs_outputs)

## Build Phases
- gradle의 build는 3개의 단계가 있다.
- 이 3개의 단계를 순서대로 실행한다.

### 1. Initialization
- `settings.gradle(.kts)` 파일 찾기
- `Settings` 인스턴스 생성
- settings file을 확인해서 어떤 프로젝트가 빌드에 사용되는지, 포함되어야 하는지 결정
- 모든 프로젝트에 대해 `Project` 인스턴스 생성

### 2. Configuration
- 앞선 phase에서 결정된 빌드에 참여하는 모든 프로젝트의 build scripts(`build.gradle(.kts)`)를 확인해서
- task graph 생성

### 3. Execution
- 선택된 tasks를 스케줄링하고 실행
- tasks 간의 의존성으로 실행 순서 결정
- parallel하게 tasks들 실행

---

-[Docs: Build Lifecycle](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:build_phases)
