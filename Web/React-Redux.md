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

### JSX 란?
* jsx 는 html 로 보이지만 브라우저가 이해할 수 있는 언어가 아니다.
* jsx 는 유저 브라우저에서 직접 해석되지 않고 babel 을 통해 ES5 js 로 변환된다.
* [babel 웹페이지](https://babeljs.io/repl) 에서 어떻게 변환되는지 확인할 수 있다.
    * react 코드
    ```jsx
    const App = () => {
        return <div>Hi there!</div>;
    }
    ```
    * ES5 로 변환된 js 코드
    ```js
    var App = function App() {
    return React.createElement("div", null, "Hi there!");
    };
    ```

### JSX vs HTML
* element 에 style 을 적용할 때 다른 문법을 사용해야 한다
    * HTML
    ```html
    <div style="background-color: red;"></div>
    ```
    * JSX
    ```jsx
    // 바깥 {} 괄호는 js 변수를 참조한다는 표현이고, 안쪽 {} 괄호는 js object 를 표현한다.
    <div style={{backgroundColor: 'red'}}></div>
    ```
* element 에 class 를 추가할 떄 다른 문법을 사용해야 한다
    * HTML
    ```html
    <div class="root"></div>
    ```
    * JSX
    ```jsx
    // class (element attribute) 대신 className 을 사용해야 한다
    // class 는 es2015 부터 js 키워드로 사용되기 때문
    <div className="root"></div>
    ```
* JSX 는 js 변수 또는 함수를 참조할 수 있다
    * HTML
    ```html
    <label>hello</label>
    ```
    * JSX
    ```jsx
    // class (element attribute) 대신 className 을 사용해야 한다
    // class 는 es2015 부터 js 키워드로 사용되기 때문```
    const text = 'hello';
    <label>{text}</label>
    ```
    * string 이 와야하는 곳에는 js object 를 참조할 수 없다

---

## 03 Props

* Component 를 작성할 때 신경써야 할 것
    * Nesting
    * Reusability
    * Configuration

### semantic-ui
* 오픈소스 css framework
* 자세한 설명은 TODO

* 간단히 시작하기
```html
<html>
    <head>
        ...
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/semantic-ui/2.4.1/semantic.min.css" />
    </head>
    ...
</head>
```

### faker.js
* 테스트용으로 필요한 가짜 데이터를 자동으로 생성해주는 오픈소스 js 라이브러리

* 간단히 시작하기
```sh
# react 프로젝트에 faker 라이브러리 설치
$ npm install --save faker
```
```jsx
import faker from 'faker';
...
const App = () => { return <img alt="avatar" src={} />; }
...
```

### Reusable, Configurable 한 컴포넌트 만들기
* 중복되는 JSX 코드를 찾는다
* 해당 JSX 블럭의 목적에 맞게 이름을 짓는다
* 새로운 컴포넌트가 들어갈 새로운 파일을 만든다. 이 때 파일이름은 컴포넌트와 동일해야 한다.
* 새로 만든 파일에 컴포넌트를 새로 만들고 JSX 코드를 옮긴다
* react 의 props 시스템을 사용해서 컴포넌트를 configurable 하게 만든다.

* 예제
    * before
    ```js
    // index.js
    import React from 'react';
    import ReactDOM from 'react-dom';
    import faker from 'faker';

    const App = () => {
        return (
            <div className="ui container comments">
                <div className="comment">
                    <a href="/" className="avatar">
                        <img alt="avatar" src={faker.image.avatar()} />
                    </a>
                    <div className="content">
                        <a href="/" className="author">
                            Sam
                        </a>
                        <div className="metadata">
                            <span className="date">Today at 06:00PM</span>
                        </div>
                        <div className="text">Nice!</div>
                    </div>
                </div>
            </div>
        );
    };

    ReactDOM.render(<App />, document.querySelector('#root'));
    ```

    * after
    ```jsx
    // @index.js
    import React from 'react';
    import ReactDOM from 'react-dom';
    import faker from 'faker';
    import CommentDetail from './CommentDetail';

    const FakeCommentDetail = () => {
        return <CommentDetail 
            author={faker.name.firstName()} 
            dttm={faker.date.recent().toString()}
            avatar={faker.image.avatar()}
            comment={faker.lorem.sentence()}
        />
    };

    const App = () => {
        return (
            <div className="ui container comments">
                <FakeCommentDetail />
                <FakeCommentDetail />
            </div>
        );
    };

    ReactDOM.render(<App />, document.querySelector('#root'));

    // @CommentDetail.js
    import React from 'react';

    const CommentDetail = props => {
        return (
            <div className="comment">
                <a href="/" className="avatar">
                    <img alt="avatar" src={props.avatar} />
                </a>
                <div className="content">
                    <a href="/" className="author"> {props.author} </a>
                    <div className="metadata">
                        <span className="date">{props.dttm}</span>
                    </div>
                    <div className="text">{props.comment}</div>
                </div>
            </div>
        );
    }

    export default CommentDetail;
    ```

### props
* 부모 컴포넌트에서 자식 컴포넌트로 데이터를 넘기기 위한 시스템
* 자식 컴포넌트를 설정하거나 커스텀하기 위한 목적으로 사용

* 부모 컴포넌트에서 자식 컴포넌트로 데이터를 넘기려면 jsx 태그에 attribute 로 달아주면 된다
* 자식 컴포넌트에서 데이터를 참조하려면 props 를 인자로 받아서 부모 컴포넌트에서 넣어준대로 키밸류로 참조할 수 있다.
* props 로 데이터를 전달하는 방법
    * 부모 컴포넌트에서 jsx 태그의 attribute 로 데이터를 넘기는 방법 ( props.{keyName} )
    ```jsx
        // 부모 컴포넌트
        <Comment comment="Nice!" />
        // 자식 컴포넌트
        const Comment = props => { return <div> {props.author} </div> };
    ```
    * 부모 컴포넌트의 jsx 태그 내용에 자식 컴포넌트의 jsx 태그를 쓰는 방법 ( props.children )
    ```jsx
        // 부모 컴포넌트
        <Card>
            <Comment/>
        </Card>
        // 자식 컴포넌트 
        const Card = props => { return <div> {props.children} </div>}
    ```

---


