# Modern React with Redux

본 문서는 Udemy의 [Modern React with Redux 강좌](https://www.udemy.com/react-redux/) 를 듣고 내용을 정리한 것이다.

---

## 01 Let's Dive In!

### Codepen 으로 React 코드 맛보기
[예제 Codepen gist](https://gist.github.com/yeomii/4aad9c1ff1c1024a07687ec57e06a154)

### React 란?
* React 는 Javascript library 이다.
* React 의 목적은 HTML 컨텐츠를 브라우저를 통해 유저에게 보여주고, 유저와의 상호작용을 처리하는 것이다.
* React 의 component 는 javascript 의 function 또는 class 를 통해 만들 수 있다.
* JSX 는 HTML 과 유사하며, js 코드 내에 위치할 수 있다. HTML 처럼 react app 의 컨텐츠를 나타내는 js `syntax extension` 이다.
* React 라이브러리는 component 를 정의하고 여러 component 가 함께 동작할 수 있도록 한다.
* ReactDOM 라이브러리는 DOM 내에서 component 가 보여지도록 하는데 책임이 있다.

### React Project 만들기
* 진행 순서
    * node js 설치 > create-react-app 설치 > 프로젝트 생성 > 프로젝트 빌드
#### node js 설치
```sh
# node 설치 여부 확인
$ node -v 
```
* 버전이 낮거나 설치되어있지 않다면 [링크](https://nodejs.org/en/download/)에서 최신 버전을 받아 설치한다.
* 여러 버전의 노드를 설치하고 싶은 경우 아래와 같이 nvm 을 사용하여 설치할 수도 있다.
```sh
# nvm 설치 https://github.com/creationix/nvm
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
# 설치 가능한 lts node 버전 리스트업
$ nvm ls-remote --lts
# latest lts 버전 노드 설치
$ nvm install --lts
# 설치 성공했는지 버전 확인
$ node -v 
# 노드 버전이 여러개라면 원하는 버전을 기본으로 지정해준다
$ nvm alias default v10.15.3
```
#### create-react-app 설치
```sh
# global 하게 create-react-app 패키지 설치
$ npm install -g create-react-app
```
#### 프로젝트 생성 및 빌드
* 프로젝트 생성
```sh
# first-react-app 이라는 이름으로 새 프로젝트 생성
$ create-react-app first-react-app
# create-react-app 을 설치하지 않고 npx 를 사용할 수도 있다.
$ npx create-react-app first-react-app
```
* 프로젝트 빌드
```sh
$ cd first-react-app
$ npm start
```

### react 와 babel
* js 버전 변화
    * ES5 > ES2015 > ES2016 > ... > ES2019
* ES5 버전은 모든 브라우저에 호환성이 있는 반면, ES2015 버전은 거의 대부분의 브라우저에 호환성이 있고, ES2016 이후의 버전은 브라우저 호환성이 좋지 않다.
* babel 은 js 컴파일러로, ES2015 이상의 버전으로 작성된 js 코드를 현재 또는 오래된 브라우저가 이해할 수 있는 js 코드로 변환해준다.

### 프로젝트 구조 
* src/
    * 실제로 작성할 코드가 들어가는 폴더
* public/
    * 이미지와 같은 정적 파일이 들어가는 폴더
* node_modules/
    * 프로젝트 의존성이 들어가는 폴더
* package.json
    * 프로젝트 의존성을 설정하고 기록하는 파일
* package-lock.json
    * 설치된 패키지의 정확한 버전을 기록하는 파일
* README.md

### index.js 시작하기
@ src/index.js 
```js
// Import React and ReactDOM libraries
import React from 'react';
import ReactDOM from 'react-dom';

// Create a react component
const App = () => {
    return <div>Hi there!</div>;
}

// Take the react component and show it on the screen
ReactDOM.render(
    <App />,
    document.querySelector('#root')
);
```

#### import vs require
* js 에서 라이브러리를 import 할 때, import 또는 require 키워드를 사용한다.
```js
// ES2015 Module
import React from 'react';
// CommonJS Module
const React = require('react');
```

#### React Component
* js 의 function 또는 class 로 작성할 수 있다.
* JSX 를 사용해서 HTML 을 생성하여 유저에게 보여줄 수 있다.
* event handler 를 사용해서 유저로부터 피드백을 받아올 수 있다.

---

## 02 Building Content with JSX
* TODO
