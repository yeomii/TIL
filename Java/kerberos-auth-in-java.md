# Java code 로 kerberos 인증하기 (kinit)

## kotlin code

```kotlin
fun main() {
    val krb5Conf = this.javaClass.classLoader.getResource("krb5.conf").file
    val jaasConf = this.javaClass.classLoader.getResource("jaas.conf").file
    System.setProperty("java.security.krb5.conf", krb5Conf)
    System.setProperty("java.security.auth.login.config", jaasConf)
    Config.refresh()

    val kp = KerberosPrincipal("${principal}@${REALM}")
    val subject = Subject(false, setOf(kp), setOf<Object>(), setOf<Object>())
    val loginContext = LoginContext("Krb5Login", subject, SimpleCallbackHandler(user, password))
    loginContext.login()
    Subject.doAs(loginContext.subject, PrivilegedExceptionAction {
        // do something
    })
}

class SimpleCallbackHandler(private val name: String, private val password: String): CallbackHandler {
    val log = LoggerFactory.getLogger(this.javaClass)

    override fun handle(callbacks: Array<out Callback>?) {
        callbacks?.forEach {callback ->
            when(callback) {
                is NameCallback -> {
                    log.info(callback.prompt)
                    callback.name = name
                }
                is PasswordCallback -> {
                    log.info(callback.prompt)
                    callback.password = password.toCharArray()
                }
                else -> throw UnsupportedCallbackException(callback, "The submitted Callback is unsupported")
            }
        }
    }
}
```

## config files

* src/main/resources/jaas.conf
```
Krb5Login {
   com.sun.security.auth.module.Krb5LoginModule required debug=true;
};
```

* src/main/resources/krb5.conf
```
[libdefaults]
default_realm = REALM_A
...

[realms]
REALM_A = {
  kdc = ...
  admin_server = ...
  admin_server = ...
}
REALM_B = {
...
}
```

## references
* JAAS (Java Authentication and Authorization Service)
    * https://docs.oracle.com/javase/8/docs/technotes/guides/security/jaas/JAASRefGuide.html
    * https://docs.oracle.com/javase/8/docs/technotes/guides/security/jgss/tutorials/AcnOnly.html
* Krb5LoginModule
    * https://docs.oracle.com/javase/8/docs/jre/api/security/jaas/spec/com/sun/security/auth/module/Krb5LoginModule.html
* CDH Hive jdbc driver guide
    * https://docs.cloudera.com/documentation/other/connectors/hive-jdbc/2-6-5/Cloudera-JDBC-Driver-for-Apache-Hive-Install-Guide.pdf