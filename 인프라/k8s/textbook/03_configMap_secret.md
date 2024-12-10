# ConfigMap, Secret
- 컨테이너를 사용하여 애플리케이션을 실행할 때 장점
    - 다양한 환경 차이를 원천적으로 없앨 수 있다!
    - 모든 환경에서 완전히 동일한 바이너리 사용 - 테스트, 운영, ..
- 하지만, 환경 간 차이가 아애 없을 수 없다! 
    - 환경별 설정값을 `주입`
- ConfigMap, Secret
    - 컨테이너에 설정값을 주입하는 데 쓰는 리소스
    - 포맷 제한 없이 데이터 보유 가능
    - 파드 정의에서 다른 리소스와 독립적인 장소에 보관된 컨피그맵과 비밀값의 데이터를 가져올 수 있음
- 쿠버의 설정 관리는 엄청 유연함!!! 거의 모든 요구 사항 만족 가능!

## 1. 쿠버네티스에서 애플리케이션에 설정이 전달되는 과정
- 개요
    - 명령형, 선언형 둘 다 가능
    - 컨피그맵과 비밀값은 어떤 기능을 하진 않는다.
    - 적은 양의 데이터를 저장하는 것이 목적이다.
    - 파드로 전달되어 컨테이너 환경의 일부가 되고, 컨테이너는 이 리소스에 저장되어 있는 데이터를 읽는다.
- **`환경변수`**
    - evn가 추가된 pod 정의 매니페스트
    ```yaml
    spec:
      containers: # 컨테이너를 설정해줄 때 env를 지정해 줄 수 있다! 
        - name: jm
            image: jm/jm-server
            env:
            - name: JM_ENV # 환경 변수 이름 정의
              value: "25" # 환경 변수 값 정의
    ```
    - 환경 변수는 파드 생명 주기 내내 변하지 않는다. 수정 불가능
    - 설정 값을 바꾸거나 패치를 적용하는 경우에도 배치가 필요
    - 쿠버는 이렇게 배치가 잦기 때문에 애플리케이션도 이를 고려해야 한다.
- **`ConfigMap`**
    - 파드에서 읽어 들이는 데이터를 저장하는 리소스
    - 한 개 이상의 키-값 쌍, 텍스트, 바이너리까지 다양하게 저장 가능
    - 환경 변수, 설정 파일(JSON, XML, YAML 등)을 파드에 전달 가능
    - 바이너리 파일 형태로된 라이선스 키도 전달 가능
    - 파드에 여러 개의 컨피그맵 전달 가능, 반대로 여러 파드에 컨피그맵 전달 가능
    - 정의(pod의 container 정의 중 일부분)
    ```yaml
    env:
      # 환경 변수를 바로 지정
    - name: JM_ENV
      value: 25
      # 환경 변수를 configMap에서 가져옴
    - name: JM_ENV_ENV 
      valueFrom:
        configMapKeyRef: # 컨피그맵에서 읽어 들이라!
          name: jm-config # 컨피그맵 리소스의 이름
          key: j.name # 컨피그맵에서 읽어 들일 항목의 이름
    ```
    - **컨피그맵을 참조한 파드는 해당 컨피그맵이 있어야 클러스터에 배치할 수 있다.**
        - jm-config라는 configMap 리소스가 있어야 이 Pod를 클러스터에 배치할 수 있다.
    - 컨피그맵 만들기
        - `리터럴 방식(--from-literal)`
            ```bash
            $ kubectl create configmap jm-config --from-literal=jm.name='hulla'
            ```
        - 환경 설정 리터럴 값을 jm-config에 저장해 놓고, 이걸 yaml에서 configMapKeyRef로 받아와서 JM_ENV_ENV 라는 환경변수에 할당함

## 2. 컨피그맵에 저장한 설정 파일 사용하기
- 환경 파일을 통해 configMap 생성하기
    - `환경 파일`
        ```jm.env
        JM_ENV=jm
        ```
    - `환경 파일 방식(--from-env-file)`
        ```bash
        - kubectl create configMap jm-env --from-env-file=jm/jm.env
        ```
    - 환경 파일에 있는 key-value 텍스트를 가져와서 jm-env라는 configMap 리소스를 만듬
    - yaml에서 configMapRef에서 파일로 모든 env가 등록됨
    - `읽어들이는 파드 정의`
        ```yaml
        env:
          - name: JM_ENV
            value: "JM_1"
          - name: JM_NAME
            valueFrom:
              configMapKeyRef: # key로 값을 가져와서 JM_NAME에 할당
                name: jm-config-literal # 이 configMap에 key-value로 값이 있어야 함 - 이건 환경변수는 아님
                key: jm.name # jm-config-literal이라는 configMap에 name이라는 key로 저장된 value를 가져와서 JM_NAME에 할당
        envFrom:
          - configMapRef: # configMap 리소스에 key-value 모두 가져옴
              name: jm-config-env-file # 이 configMap에 key-value로 환경변수가 있어야 함
        ```
- `우선 순위`
    - 환경 변수의 이름이 중복되는 경우
        - env에서 정의된 값이 envFrom보다 우선!!!
    - JSON파일
        - 환경변수는 모든 JSON 설정 파일보다 우선!!!
- __설정의 출처별로 다른 우선순위 적용 전략__
    - 1. 기본 설정은 컨테이너 이미지에 포함
    - 2. 각 환경의 실제 설정값은 configMap에 담겨 컨테이너 파일 시스템에 전달
        - 앱에서 설정 파일을 찾도록 지정된 경로에 파일 형태로 주입
        - 컨테이너 이미지에 담긴 파일을 덮어쓰는 형태임
    - 3. 변경 필요한 설정값은 파드 정의에서 환경 변수 형태로 적용
- __파일의 내용을 항목으로 사용해서 `ConfigMap` 생성__
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: jm-web-config-dev
    data:
        config.json: |-
            {
                "ConfigController": {
                    "Enabled": true
                }
            }
    ```
    - jm-web-config-dev 컨피그맵 리소스에는 `config.json`이라는 파일이 생성되어 저장
    - 이 컨피그맵을 사용하는 곳에서는 볼륨을 마운트하고 해당 경로에 이 파일을 지정해주어야 함
    - 어떤 텍스트 파일도 YAML 포맷에 삽입 가능

- 이렇게 설정한 ConfigMap을 적용하려면 성행되어야 하는 두 가지
    - 1. 애플리케이션에서 ConfigMap에서 주입한 데이터를 알아서 설정값에 반영해야 함 -> 이건 애플리케이션 로직!!
    - 2. 파드에서 컨피그맵을 참조하여, 컨테이너 파일 시스템의 지정된 위치에 데이터를 들여 와야 함!!!

## 3. 컨피그맵에 답긴 설정값 데이터 주입하기
- 파일로 설정값 주입
- 컨테이너 파일 시스템은 `컨테이너 이미지`와 `그 외 출처에서 온 파일`로 구성되는 가상 구조
- 이 컨테이너 파일 시스템에 컨피그맵도 추가 가능
- `컨피그맵`은 `디렉터리`, `각 항목`은 `파일 형태`로 컨테이너 파일 시스템에 추가
- `볼륨`
    - 컨피그맵에 저장된 데이터를 파드로 전달
- `볼륨 마운트`
    - 컨피그맵을 읽어 들인 볼륨을 파드 컨테이너의 특정 경로에 위치시킴
- __컨피그맵을 `볼륨 마운트` 형태로__
    ```yaml
    spec:
    container:
        - name: web
          image: jm/todo-list
          volumeMounts:
            - name: config # 이 이름이 volume과 같아야 함
              mountPath: "/app/config" # volume이 올라갈 컨테이너 내의 경로
              readOnly: true
    volumes:
        - name: config # 이 이름을 volume mount에서 적어줘야 함
          configMap:
            name: jm-web-config-dev # 위에서 정의한 ConfigMap 리소스의 이름
    ```
- 컨피그맵이 디렉터리로 취급!!! 각 항목이 파일로 취급!!!!
    - jm-web-config-dev라는 컨피그맵이 디렉터리로 취급되고, 이 안에 정의된 항목인 config.json이 파일이 됨!!!
- __모든 설정을 하나의 컨피그맵으로 관리__
    - configMap 수정
    ```yaml
    data:
        config.json: |-
            {
                "ConfigController": {
                    "Enabled": true
                }
            }
        logging.json: |-
            {
                "Logging": {
                    "LogLevel": {
                        "TodoList.Pages": "Debug"
                    }
                }
            }
    ```
    - 이렇게 하면 두 개의 파일이 volume에 읽어지고, volume mount에서 정의한 컨테이너 경로에 생성된다.
- __파드가 동작 중인 상황에서 컨피그맵을 업데이트__
    - 애플리케이션 나름!!!!
        - 시작 시 설정 읽어오는 앱이면 pod 다시 시작해야 설정 반영
        - 지속적으로 설정을 확인하는 앱이면 다시 시작 안해도 된다.
- __주의!!!__ : 이미 컨테이너 이미지에 있는 경로인 경우
    - __덮어써진다!!!__
    - 볼륨 마운트에서 정의한 경로가 컨테이너에 이미 존재하지는 않는지 주의 필요
- __`단일 항목만` 전달__
    ```yaml
    spec:
    container:
        - name: web
          image: jm/todo-list
          volumeMounts:
            - name: config 
              mountPath: "/app/config"
              readOnly: true
    volumes:
        - name: config
          configMap:
            name: jm-web-config-dev
            items:
              - key: config.json # configMap의 항목 중에 하나 지정
              - path: config.json # 파일로 전달하도록 지정
    ```
- __컨피그맵은 텍스트 파일을 잘 추상화한 객체__
    - 일 뿐.. 보안적 수단이 없다.

## 4. 비밀값 이용하여 민감한 정보가 담긴 설정값 다루기
- __`Secret`__
    - 클러스터 내부에 별도로 관리
    - 노출이 최소화
    - 해당 값을 사용하는 노드에만 전달됨
    - 디스크에 저장되지 않고 메모리에만 
    - 모두 암호화 되어 전달
    - 권한이 있다면 평문을 볼 수 있지만, Base64로 인코딩 되어 원본을 바로 볼 수는 없음
- Secret 값을 주입 받는 앱 정의
    ```yaml
    spec:
      contianers:
        - name: hello
          image: jm/hello
          env:
            - name: JM_SECRET # Secret 항목이 담길 환경변수
              valueFrom:
                secretKeyRef:
                  name: hello-secret-literal # Secret 리소스 이름
                  key: secret # Secret 항목 이름
    ```
- __환경 변수 주의__
    - 환경 변수는 컨테이너 내의 모든 프로세스에서 접근 가능
    - 오류 발생 시 환경 변수가 로그로 남을 수 있음
    - 대안은 `파일 형태`로 전달하는 것
- Secret 정의
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: todo-db-secret
    type: Opaque
    stringData:
      POSTGRES_PASSWORD: "hello-1234"
    ```
    - 이 Secret 리소스를 사용하는 법
        - 1. 환경 변수로 직접 전달
        - 2. 파일 형태로 전달
            - 파일로 전달하고, 설정 파일의 경로를 환경변수에 지정
- __파일 형태로 전달__
    ```yaml
    template:
      metadata:
        labels:
          app: todo-db
      spec:
        containers:
          - name: db
            image: postgres:11.6-alpine
            env:
            - name: POSTGRES_PASSWORD_FILE
              value: /secrets/postgres_password
            volumeMounts:
              - name: secret
                mountPath: "/secrets"
        volumes:
          - name: secret
            secret:
              secretName: todo-db-secret-test
              defaultMode: 0400
              items:
              - key: POSTGRES_PASSWORD
                path: postgres_password
    ```
    - todo-db-secret-test이라는 Secret에서 POSTGRES_PASSWORD 라는 항목만 사용, postgres_password라는 파일로 지정
    - 컨테이너에는 secret이라는 볼륨을 찾아서 /secrets 경로에 볼륨을 배치
    - /secrets/postgres_password 가 생성될 것!!
    - 환경 변수 POSTGRES_PASSWORD_FILE에 이 경로를 넣어서, 애플리케이션에서 이 경로로 접근 가능하도록 함
    - 0400 권한을 주었기 때문에, 컨테이너 사용자만 읽을 수 있음

- __정리__
    - 컨테이너 속 환경 변수나 설정 파일의 형태로, 컨피그맵이나 비밀값 리소스의 데이터를 주입하면 된다.

## 5. 쿠버네티스의 애플리케이션 설정 관리
- 목표!
    - 애플리케이션 외부 환경에서 설정값을 주입 받는 것
    - 나름 우선순위를 부여해서 파일과 환경 변수의 형태로 주입
- 설계 단계에서 고려해야할 두가지
    - 1. 애플리케이션의 중단 없이 설정 변경에 대응이 필요한가?
        - 파드 교체와 중단 없이 설정 변경
            - 파드 교체가 반드시 필요한 환경 변수는 활용 불가능
            - 봄륨 마운트를 이용하여 설정 파일을 수정하는 방법을 사용해야 함
            - 볼륨을 수정하는 것이 아니라 기존 컨피그맵이나 비밀값을 업데이트해야 함. 볼륨을 수정하면 파드를 교체해야 하기 때문
        - 파드 교체 허용
            - 설정 객체에 버전 명명법 도입
            - 새로운 설정 객체를 배치하고, 애플리케이션의 정의를 수정하여 새로운 설정 객체를 가리키게 한다.
            - 파드 교체 없는 업데이트는 불가능하지만 설정값 변경 이력이 남아 설정 롤백이 가능해진다.
    - 2. 민감 정보를 어떻게 관리할 것인가?
        - 전담팀이 있는 경우, 버전 관리 정책이 적합
        - 형상 관리 도구에 저장된 YAML 템플릿 파일로 컨피그맵과 비밀값 정의가 생성되는 완전 자동화된 배치
        - YAML 템플릿 파일에는 민감 정보가 채워질 빈칸을 두고, 배치 절차 중에 채우는 방식
- 중요한 건,
    - 애플리케이션 __`플랫폼을 통해`__ 설정값이 주입되어야 한다는 점
    - 그래야 __환경에 무관하게 동일한 이미지를 사용__ 할 수 있다!