# Spock

* spock 은 Groovy 언어로 작성된 java, groovy 기반 애플리케이션 테스팅 프레임워크이다.
* 테스트 코드를 간결하고 예쁘게 작성할 수 있는 것이 장점이다.
* [공식 문서](http://docs.spockframework.org/)

---

## 빠르게 시작하기
* [Spock Web Console](http://meetspock.appspot.com/)

## spock 테스트 코드 작성하기
java 기반 프로젝트에서 간단하게 리스트를 테스트해보는 코드를 작성해보겠다.


@ build.gradle
```gradle
apply plugin: 'groovy'
...
dependencies {
  // mandatory dependencies for using Spock
  compile "org.codehaus.groovy:groovy-all:2.4.15"
  testCompile "org.spockframework:spock-core:1.2-groovy-2.4"
}
...
```

@ src/test/groovy/com/test/ListTest.groovy
```groovy
package com.test

import spock.lang.Specification

class ListTest extends Specification {

    def addTest() {
        // test 를 위한 setup 코드를 작성하는 block
        given:
        def list = new ArrayList()
        def elem = 1

        // stimulus, 테스트를 할 method 를 호출하는 block
        when:
        list.add(elem)

        // response, 예상되는 결과를 검증하는 block
        then:
        list.size() == 1
        list.last() == elem
    }
}
```

## Spring 프로젝트에서 사용하기
* TODO