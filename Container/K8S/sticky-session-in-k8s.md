# 쿠버네티스에서 stick session 설정 사용하기

## 문제 상황
* k8s 에 grafana 이미지를 올려서 서비스 하고 있는데, css 파일이 로드되지 않는 문제가 수시로 발생

## 문제 원인
* ingress 를 사용하다보니 웹페이지 리소스를 여러개의 파드에서 읽어온다.
* grafana 에서 사용하는 css 파일에는 이미지마다 서로 다른 해시값이 붙어있기 때문에 A 파드에서 html 을 읽어온 후 app.A_HASH.css 파일을 B 파드에서 읽어오려고 하면 해당 파일은 없고 app.B_HASH.css 파일만 존재하기 떄문에 404 응답만을 받는다.

## 문제 해결 방법
* grafana 에서는 HA 를 위해 여러개의 서버를 쓰고, LB 를 쓸 경우 사용자 세션을 어떻게 관리해야 할 지 고려해야 한다고 언급하고 있다. ([참고](https://grafana.com/docs/tutorials/ha_setup/#user-sessions))
* k8s 에서는 ingress 가 nginx 를 이용해 로드밸런서 역할을 하므로 nginx 에서 sticky session 을 사용할 수 있도록 설정을 해주면 된다.

## k8s 에서 nginx sticky session 설정하기
* 간단히 아래와 같이 ingress.yaml 파일ㅇ annotation 으로 쿠키를 이용한 sticky session 설정을 해주면 된다.

* https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/ingress.yaml
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-test
  annotations:
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
    nginx.ingress.kubernetes.io/session-cookie-expires: "172800"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "172800"

spec:
  rules:
  - host: stickyingress.example.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /
```

## 레퍼런스
* [Medium : Sticky Sessions in Kubernetes](https://medium.com/@zhimin.wen/sticky-sessions-in-kubernetes-56eb0e8f257d)
* [k8s ingress-nginx sticky session docs](https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/)
* https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/ingress.yaml