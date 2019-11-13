# JarClassLoader

* https://docs.oracle.com/javase/tutorial/deployment/jar/jarclassloader.html
* https://docs.oracle.com/javase/tutorial/deployment/jar/examples/JarClassLoader.java


## 코틀린 코드

```kotlin
import java.lang.reflect.Modifier
import java.net.JarURLConnection
import java.net.URL
import java.net.URLClassLoader
import java.util.jar.Attributes


class JarClassLoader(private val url: URL): URLClassLoader(arrayOf(url)) {

    fun getMainClassName(): String? {
        val u = URL("jar", "", "$url!/")
        val jarURLConnection = u.openConnection() as? JarURLConnection
        return jarURLConnection?.mainAttributes?.getValue(Attributes.Name.MAIN_CLASS)
    }

    fun invokeClass(name: String, args: Array<String>) {
        val method = loadClass(name).getMethod("main", args.javaClass)
        method.isAccessible = true
        val modifiers = method.modifiers

        if (method.returnType !== Void.TYPE || !Modifier.isStatic(modifiers) || !Modifier.isPublic(modifiers)) {
            throw NoSuchMethodException("main")
        }
        try {
            method.invoke(null, arrayOf<Any>(args))
        } catch (e: IllegalAccessException) {
            // This should not happen, as we have disabled access checks
        }
    }
}
```

* URLClassLoader 
    * url 을 이용하여 클래스와 리소스를 로딩하는 클래스
* getMainClassName method
    * jar 파일 중 어떤 클래스가 애플리케이션의 entry point 가 될 지 결정
    * jar 파일의 manifest 에서 Main-Class 헤더에 명시된 내용
* JAR URL format
    * jar 파일을 가리키는 url 형태 - `jar:http://www.example.com/jarfile.jar!/`
    * jar 파일 내의 컨텐츠를 가리키는 형태 - `jar:http://www.example.com/jarfile.jar!/mypackage/myclass.class`
* JarURLConnection
    * 애플리케이션과 jar 파일 사이의 communication link
    * `URLConnection` 을 `JarURLConnection` 으로 캐스팅하면 jar 를 다루는 기능들을 사용할 수 있다.
* invokeClass method
    * 애플리케이션 내에서 main class 가 invoke 될 수 있도록 함
    * entry point 가 될 클래스의 이름과, 해당 클래스의 main 메소드에 전달될 argument 의 배열을 파라미터로 받음