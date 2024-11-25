# maven central에 라이브러리 업로드하기

- Gradle을 사용해서 Maven Central에 라이브러리를 업로드하려면 몇 가지 설정이 필요
- Gradle에서는 Maven Central 배포를 위해 `maven-publish 플러그인`과 `Signing 플러그인`을 주로 사용

## 목차
1. Sonatype 계정 등록 및 Group ID 확인
    - OSSRH JIRA에서 Group ID 등록
2. Gradle 설정
    - maven-publish와 signing 플러그인 설정
    - pom 정보를 정확히 입력
3. GPG 서명 및 Gradle Properties
    - GPG 키를 생성하고, Gradle에서 서명 설정
4. 배포 작업
    - ./gradlew publish로 Sonatype Staging Repository에 업로드
    - Sonatype Nexus에서 검토 후 Maven Central에 릴리스

## 1. Maven Central 업로드를 위한 사전 준비

#### 1.1 소유 도메인 준비
- Maven Central 배포를 위해 고유한 Group ID가 필요
- 소유 도메인의 역순으로 설정
    - 예: com.example
    - 도메인이 없는 경우 
        - GitHub를 사용해 io.github.username 형식의 Group ID를 사용 가능

#### 1.2 Sonatype 계정 생성
- Maven Central 배포
    - `Sonatype OSSRH(Open Source Software Repository Hosting)`를 통해!
    1.	Sonatype 계정 생성 및 로그인.
    2.	프로젝트 등록 요청:
        - OSSRH JIRA에서 새로운 이슈를 생성.
        - 주요 정보:
        - Group ID (프로젝트 고유 식별자).
        - 프로젝트 설명.
        - 도메인 소유 증명(GitHub URL 또는 도메인 기반 이메일).

## 2. 프로젝트 준비

#### 2.1 build.gradle 설정
- Gradle을 사용하는 프로젝트의 build.gradle 또는 build.gradle.kts에 Maven Central 배포를 위한 설정을 추가
- 플러그인 추가
    ```grooby
    plugins {
        id("maven-publish")
        id("signing")
    }
    ```

#### 2.2 Maven Central 배포 설정
- publishing 블록 설정
```groovy
publishing {
    publications {
        create<MavenPublication>("mavenJava") {
            groupId = "com.example" // Group ID
            artifactId = "your-library" // Artifact ID
            version = "1.0.0" // 버전

            from(components["java"]) // Java 컴포넌트

            pom {
                name.set("Your Library")
                description.set("A library for XYZ")
                url.set("https://github.com/username/your-library")

                licenses {
                    license {
                        name.set("The Apache License, Version 2.0")
                        url.set("http://www.apache.org/licenses/LICENSE-2.0.txt")
                    }
                }

                developers {
                    developer {
                        id.set("your-id")
                        name.set("Your Name")
                        email.set("your-email@example.com")
                    }
                }

                scm {
                    connection.set("scm:git:git://github.com/username/your-library.git")
                    developerConnection.set("scm:git:ssh://git@github.com:username/your-library.git")
                    url.set("https://github.com/username/your-library")
                }
            }
        }
    }

    repositories {
        maven {
            name = "OSSRH"
            url = uri("https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/")
            credentials {
                username = project.findProperty("ossrhUsername") as String? ?: ""
                password = project.findProperty("ossrhPassword") as String? ?: ""
            }
        }
    }
}
```

- GPG 서명 추가
    - `signing` 플러그인을 사용하여 아티팩트에 GPG 서명을 추가
    ```
    signing {
        useGpgCmd() // GPG 명령어를 사용
        sign(publishing.publications["mavenJava"])
    }
    ```

- GPG 서명을 위해 공개 키와 개인 키를 사전에 생성해야 함

#### 2.3 Gradle Properties 설정
- `~/.gradle/gradle.properties` 파일에 `Sonatype 인증 정보`와 `GPG 서명 키`를 추가
```
ossrhUsername=your-sonatype-username
ossrhPassword=your-sonatype-password
signing.keyId=YOUR_KEY_ID
signing.password=YOUR_GPG_PASSWORD
signing.secretKeyRingFile=/path/to/your/secring.gpg
```
- `ossrhUsername`과 `ossrhPassword`
    - Sonatype 계정 정보
- `signing.keyId`
    - GPG 키 ID
- `signing.secretKeyRingFile`
    - GPG 비밀 키 파일 경로

#### 2.4 라이브러리 배포 빌드

1. 배포 아티팩트 확인
    ```
    ./gradlew publishToMavenLocal
    ```
	- 로컬 Maven 저장소에 배포하여 아티팩트 구조를 확인

2. Maven Central에 업로드
    ```
    ./gradlew publish
    ```
	- publish 작업은 아티팩트를 Sonatype Staging Repository에 업로드

## 3. Maven Central 배포 과정

#### 3.1 Sonatype Staging Repository 확인
1. 업로드가 완료되면 Sonatype Nexus Repository Manager에 로그인
2. Staging Repository에서 업로드된 라이브러리를 확인

#### 3.2 Staging Repository 배포
1. Staging Repository에서 모든 내용을 확인한 후 Close 버튼 클릭
2. Release 버튼을 클릭하여 Maven Central에 배포

#### 3.3 배포 확인
- 배포 후 약 10~30분 내에 Maven Central에서 라이브러리를 확인 가능
	- Maven Central
        - [https://search.maven.org/](https://search.maven.org/)

---

## 추가
- [git actions ci workflow로 자바 라이브러리 배포](https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-java-packages-with-gradle)