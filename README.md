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

개별 서브프로젝트([module1 build.gradle](module1/build.gradle), [module2 build.gradle](module2/build.gradle)에서만 사용할 플러그인, 자바 설정, 리포지터리, 의존성을 설정할 수 있다

```groovy
dependencies {
    //서브프로젝트를 위한 의존성 추가
}
```