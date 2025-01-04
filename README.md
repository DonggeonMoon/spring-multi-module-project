# 스프링 멀티 모듈 프로젝트

## 사용 버전

```text
Gradle: 8.10(>7.0)
Spring Boot: 3.4.1
Spring Dependency Management: 1.1.7
```

## 멀티 모듈 구조

```text
spring-multi-module-project (root project)
├── module1 (subproject)
└── module2 (subproject)
```

## settings.gradle

루트 프로젝트의 [settings.gradle](settings.gradle)에 서브프로젝트(모듈)를 추가해줘야 한다.

```groovy
rootProject.name = 'spring-multi-module-project'
// 서브프로젝트 추가
include 'module1'
include 'module2'
```

## build.gradle

### 루트 프로젝트

루트 프로젝트의 bulid.gradle([build.gradle](build.gradle))에서는 서브프로젝트에서 공통적으로 사용할 플러그인, 자바 설정, 리포지터리, 의존성을 설정할 수 있다.

```groovy
plugins {
    // 추가할 플러그인
}

// 버전 값을 한번에 관리하고 가독성을 높이기 위해 ext 블록 사용
ext {
    javaVersion = 17
}

// 전역 변수
group = 'com.dgmoonlabs'
version = '1.0-SNAPSHOT'

// 서브프로젝트에서 사용할 플러그인, 자바 설정, 리포지터리, 의존성
// 여기에 추가할 경우 루트 프로젝트보다 하위에 위치한 프로젝트들은 모두 아래 설정을 공유함
subprojects {
    apply plugin: 'java'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'

    java {
        toolchain {
            languageVersion = JavaLanguageVersion.of(17)
        }
    }

    repositories {
        mavenCentral()
    }

    dependencies {
        implementation 'org.springframework.boot:spring-boot-starter'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
        testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
    }

    tasks.named('test') {
        useJUnitPlatform()
    }

}

// 그외 태스크들
tasks {
    //...
}


```

### 서브프로젝트

개별 서브프로젝트([module1 build.gradle](module1/build.gradle), [module2 build.gradle](module2/build.gradle)에서만 사용할 플러그인, 자바 설정,
리포지터리, 의존성을 설정할 수 있다

```groovy
dependencies {
    //서브프로젝트를 위한 의존성 추가
}
```


# 부록
## Gradle settings.gradle 사용 가이드

> 참고: 이 문서에서는 안정적으로 사용 가능한 프로퍼티와 블록만 다루며, 실험 단계에 있는 기능(`@Incubating` 표기)은 제외함

#### 프로퍼티 목록

- settings: Gradle 설정 스크립트 객체 참조
- layout: 프로젝트의 디렉토리 및 파일 레이아웃 관리
- buildscript: 빌드 스크립트에서 사용할 클래스패스 의존성 및 리포지토리 정의
- settingsDir: settings.gradle 파일이 위치한 디렉토리 참조
- rootDir: 루트 프로젝트 디렉토리 참조
- rootProject: 루트 프로젝트 객체 참조
- startParameter: Gradle 실행 시 전달된 시작 매개변수 참조
- providers: 속성 값이나 환경 변수 등을 동적으로 제공하는 API 참조
- gradle: 현재 실행 중인 Gradle 인스턴스에 대한 정보 참조
- buildCache: 빌드 캐시 설정 관리
- pluginManagement: 플러그인 리포지토리 및 플러그인 버전 관리
- sourceControl: 외부 소스 제어 시스템(예: Git) 통합 관리

#### 메서드

```groovy
include '모듈명' // 멀티모듈 프로젝트에서 하위 모듈을 포함 (계층형 디렉토리 구조에서 사용)
includeFlat '모듈명' // 멀티모듈 프로젝트에서 평면 디렉토리 구조의 하위 모듈을 포함
project '모듈명' // 특정 하위 모듈의 프로젝트 객체를 참조
findProject '모듈명' // 지정된 하위 모듈을 찾아 프로젝트 객체를 반환 (없으면 null 반환)
enableFeaturePreview '그레이들 실험적 기능명' // Gradle 실험적 기능을 활성화
```

### 블록(예시 포함)

```groovy
pluginManagement { // 플러그인 리포지토리 및 버전 관리 블록
    repositories { // 사용할 플러그인을 다운로드할 리포지토리를 지정
        gradlePluginPortal() // Gradle 공식 플러그인 포털
        mavenCentral() // Maven 중앙 리포지토리
    }
    plugins { // 사용할 플러그인의 ID와 버전을 지정
        id 'org.springframework.boot' version '3.0.0' // Spring Boot 플러그인 (버전 3.0.0)
        id 'com.example.plugin' version '1.2.0' // 사용자 정의 플러그인 (버전 1.2.0)
    }
}

includeBuild('../external-library') { // 독립적으로 관리되는 외부 Gradle 빌드를 통합
    dependencySubstitution { // 기존 의존성을 로컬 프로젝트로 대체
        substitute(module("com.example:external-lib")) // 모듈 의존성을 찾음
                .using(project(":external-library")) // 해당 모듈을 로컬 프로젝트로 대체
    }
}

buildCache { // 빌드 캐시 설정 블록
    local {
        enabled = true
        directory = file("${rootDir}/.build-cache") // 로컬 캐시 디렉토리 설정
    }
    remote(HttpBuildCache) {
        url = 'https://cache.example.com' // 원격 캐시 URL
        credentials {
            username = 'user' // 인증 정보 설정
            password = 'password'
        }
        push = true // 빌드 결과를 원격 캐시에 저장
    }
}

sourceControl { // 외부 소스 제어 시스템 설정 블록
    gitRepository("https://example.com/repo.git") { // Git 저장소 설정
        producesModule("com.example:module") // 저장소에서 제공할 모듈 지정
        branch = 'develop' // 사용할 브랜치 설정
    }
}

sourceControl { // 다중 Git 저장소를 통합하는 블록
    gitRepository("https://example.com/repo1.git") { // 첫 번째 저장소 설정
        producesModule("com.example:module1") // 저장소에서 제공할 첫 번째 모듈 지정
    }
    gitRepository("https://example.com/repo2.git") { // 두 번째 저장소 설정
        producesModule("com.example:module2") // 저장소에서 제공할 두 번째 모듈 지정
    }
}

sourceControl { // 특정 커밋을 기준으로 외부 소스를 가져오는 블록
    gitRepository("https://example.com/repo.git") { // Git 저장소 설정
        producesModule("com.example:module") // 저장소에서 제공할 모듈 지정
        commit = 'abc123def456' // 고정된 커밋 ID를 기준으로 소스 가져오기
    }
}
```