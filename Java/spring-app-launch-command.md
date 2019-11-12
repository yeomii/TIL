# 리눅스에서 간단히 spring boot jar 를 띄우는 커맨드

```bash
nohup java -jar -Dspring.profiles.active=prod  -Duser.timezone=Asia/Seoul {jar} >> log-name 2>&1 &
```