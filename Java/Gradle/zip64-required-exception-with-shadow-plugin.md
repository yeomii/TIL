# gradle shadow 플러그인을 사용하면서 Zip64RequiredException 이 발생하는 경우

* gradle shadow 플러그인으로 fat jar 를 만들 때, 파일 갯수가 65535 개를 넘어가거나 4GB 를 넘어갈 경우, `Zip64RequiredException` 이 발생한다.
* zip64 옵션을 활성화 해주면 해결된다.
    * https://docs.gradle.org/5.1/dsl/org.gradle.api.tasks.bundling.Zip.html#org.gradle.api.tasks.bundling.Zip:zip64

```gradle.kts
tasks {
    "shadowJar"(ShadowJar::class) {
        ...
        isZip64 = true
        ...
    }
}
```