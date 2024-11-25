# spring boot 라이브러리 만드는 법

## 목차

1. 새 프로젝트를 생성하거나 모듈로 추가.
2. 공통 로직 및 서비스 작성.
3. 자동 구성 클래스 추가 (필요 시).
4. JAR 파일로 패키징 및 의존성으로 추가.
5. 테스트 및 배포.

## 1. Spring Boot 라이브러리 프로젝트 설정

### 1.1. 새 프로젝트 생성
- Spring Boot 라이브러리는 일반적으로 Spring Boot Application 대신 `라이브러리 프로젝트`로 구성
- plugins에 java-library 플러그인 사용
- Gradle 설정 예시 (build.gradle)
    ```groovy
    plugins {
        id 'java-library' // 라이브러리 플러그인
        id 'org.springframework.boot' version '3.1.4' apply false // Spring Boot 플러그인
        id 'io.spring.dependency-management' version '1.1.3'
    }

    group = 'com.example'
    version = '1.0.0'
    sourceCompatibility = '17'

    repositories {
        mavenCentral()
    }

    dependencies {
        // Spring Boot와 관련된 의존성 추가
        implementation 'org.springframework.boot:spring-boot-starter'

        // 테스트 라이브러리
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }

    ```

## 2. 라이브러리 코드 작성

### 2.1. 공통 서비스 코드

- MyService라는 간단한 공통 서비스를 제공하는 라이브러리

```java
package com.example.library;

import org.springframework.stereotype.Service;

@Service
public class MyService {

    public String greet(String name) {
        return "Hello, " + name + "!";
    }
}

```

### 2.2. Spring Boot 구성

- Spring Boot에서 라이브러리를 사용할 때 자동 구성을 제공하려면 @Configuration 클래스를 추가

```java

package com.example.library.config;

import com.example.library.MyService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class LibraryAutoConfiguration {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

## 3. 라이브러리 패키징

### 3.1. JAR 파일 생성
- Gradle:
    ```shell
    $ ./gradlew build
    ```
- 생성된 custom-spring-boot-library-1.0.0.jar 파일이 build/libs 또는 target 디렉토리에 위치

## 4. 애플리케이션에서 라이브러리 사용

### 4.1. 애플리케이션 프로젝트에 라이브러리 추가
- build.gradle에 의존성을 추가
- Gradle
    ```groovy
    dependencies {
        implementation files('libs/custom-spring-boot-library-1.0.0.jar') // 로컬 파일 경로
    }
    ```

### 4.2. 애플리케이션 코드에서 서비스 사용
- 라이브러리의 MyService를 주입받아 사용
```java
package com.example.demo;

import com.example.library.MyService;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class MyRunner implements CommandLineRunner {

    private final MyService myService;

    public MyRunner(MyService myService) {
        this.myService = myService;
    }

    @Override
    public void run(String... args) {
        System.out.println(myService.greet("World"));
    }
}
```

## 5. 배포 (옵션)

### 5.1. 로컬 Maven 저장소에 배포
- 라이브러리를 로컬 Maven 저장소에 배포하여 의존성 쉽게 추가 가능
- 다음 명령어로 로컬 저장소에 라이브러리를 배포
    ```
    $ mvn install
    ```

### 5.2. 원격 Maven 저장소에 배포
- Nexus, Artifactory 또는 Maven Central에 업로드 가능
- distributionManagement 설정을 추가한 후 mvn deploy를 실행
    ```
    $ mvn deploy
    ```

## 6. 모듈화를 통해 라이브러리 관리
- Spring Boot 프로젝트가 여러 모듈로 나뉘어 있다면, 하나의 모듈을 라이브러리로 만들고 다른 모듈에서 이를 의존성으로 추가하는 방식으로 사용 가능
- Gradle 다중 모듈 설정 예:
    ```
    // settings.gradle
    include 'library-module'
    include 'application-module'
    ```
- library-module은 라이브러리로 구성되고, application-module에서 이를 의존성으로 추가
    ```
    dependencies {
        implementation project(':library-module')
    }
    ```