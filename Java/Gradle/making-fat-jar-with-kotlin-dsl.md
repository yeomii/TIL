# Gradle Kotlin DSL 로 fat jar 파일 만들기
* gradle 5 버전 기준
* fat jar 는 의존성을 가진 모든 라이브러리를 포함한 jar 파일을 말한다.
* 아래 예제에는 META-INF/*.RSA 등의 특정 파일을 제외하는 기능도 포함되어 있다.

## 추가 플러그인 없이 fat jar 를 만드는 방법

@ gradle.build.kts
```kotlin
import org.gradle.jvm.tasks.Jar

val fatJar = task("fatJar", type = Jar::class) {
    baseName = "${project.name}-fat"

    from(configurations.runtimeClasspath.get().map({ if (it.isDirectory) it else zipTree(it) })) {
        exclude("META-INF/*.RSA", "META-INF/*.SF", "META-INF/*.DSA")
    }
    with(tasks.jar.get() as CopySpec)
}

tasks {
    "build" {
        dependsOn(fatJar)
    }
}
```

## shadow 플러그인 사용해서 shadow jar 만들기

* 위 방법으로 fat jar 를 만들 때 생기는 문제는, akka 라이브러리에 의존성이 있는 경우 reference.conf 파일이 제대로 통합되지 않는다는 것이다. [참고](https://doc.akka.io/docs/akka/snapshot/additional/packaging.html#gradle-the-jar-task-from-the-java-plugin)
* shadow 플러그인을 써서 문제를 해결할 수 있다.

@ gradle.build.kts
```kotlin
import com.github.jengelman.gradle.plugins.shadow.tasks.ShadowJar
import com.github.jengelman.gradle.plugins.shadow.transformers.AppendingTransformer

plugins {
    kotlin("jvm") // version "1.3.11"
    java
    id("com.github.johnrengelman.shadow") version "5.1.0"
}

...

tasks {
    "shadowJar"(ShadowJar::class) {
        exclude("META-INF/*.RSA", "META-INF/*.SF", "META-INF/*.DSA")
        transform(AppendingTransformer::class.java) {
            resource = "reference.conf"
        }
    }
}

tasks {
    build {
        dependsOn(shadowJar)
    }
}
```