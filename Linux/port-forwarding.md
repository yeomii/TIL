# 포트 포워딩 커맨드 정리

* 80 포트로 들어오는 트래픽을 5000 포트로 포워딩
```
iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 5000
iptables -t nat -L
service iptables save
service iptables restart
```

## 레퍼런스
* https://blog.outsider.ne.kr/580
