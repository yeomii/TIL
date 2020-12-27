# Fast API 프로젝트 Heroku 로 배포하기

* Fast API - 코드 작성하기 쉽고 빠른 고성능의 웹 프레임워크 (Python 3.6+)
* Heroku - 개발자들이 클라우드 내에서 애플리케이션을 빌드, 실행, 운영할 수 있게 해주는 PaaS (Platform as a Service)

## 1. Fast API 샘플 프로젝트 준비하기
* [Fast API 웹페이지](https://fastapi.tiangolo.com/)
* [Fast API 샘플 프로젝트](https://github.com/yeomii/fast-api-heroku-example)


1. python 을 로컬에 설치하고, pip 로 `fastapi`, `uvicorn` 을 설치한다.
2. 프로젝트 루트 폴더에 app/main.py 파일을 생성한 후, Fast API 웹페이지에서 가이드하는 대로 내용을 채워넣는다.
3. `uvicorn app.main:app --reload` 커맨드로 웹서버를 실행시킨다.
4. 정상 동작하는 것을 확인했으면 `pip freeze > requirements.txt` 로 의존성을 파일에 기록한다.
5. 다음과 같은 내용의 Procfile 을 작성하고 github 에 프로젝트를 올린다.
```
web: uvicorn app.main:app --host=0.0.0.0 --port=${PORT:-5000}
```

## 2. Heroku 로 배포하기

* [Heroku](https://www.heroku.com/)
* [Heroku Procfile 문서](https://devcenter.heroku.com/articles/procfile#the-web-process-type)

1. Heroku 계정을 생성한다.
2. [앱 대시보드](https://dashboard.heroku.com/apps) 에서 New > Create new app 을 선택하고 앱 이름과 지역을 선택한 후 create app 을 선택한다.
3. Deploy 탭에서 Deployment method 를 Github 를 선택한다.
4. Connect to Github 패널에서 본인의 github 계정과 연동한 후 repo 이름으로 검색해서 앱과 연결한다.
5. repo 와 연결하고 나면 Automatic deploys 와 Manual deploy 패널로 브랜치를 선택하여 배포할 수 있다.
6. 배포가 완료되면 https://{app-name}.herokuapp.com/ 으로 접근할 수 있다.
