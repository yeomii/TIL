# 간단하게 HTTP POST 요청을 날리는 html 코드

* html 파일을 브라우저에서 열고 Post 버튼을 누르면 `localhost:3000` 으로 post 요청을 전송한다.
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Post Back Test App</title>
  </head>
  <body>
    <div id="root">
        <p>Hello</p>
        <button onclick="post()">
            Post
        </button>
    </div>
  </body>
  <script>
        console.log("test");
        function post() {
            console.log("post");
            var form = document.createElement('form');
            form.setAttribute('method', 'post');
            form.setAttribute('action', 'http://localhost:3000');
            document.charset = "utf-8";

            var value = '';
            
            var hiddenField = document.createElement('input');
            hiddenField.setAttribute('type', 'hidden');
            hiddenField.setAttribute('name', 'testField');
            hiddenField.setAttribute('value', value);

            form.appendChild(hiddenField);
            document.body.appendChild(form);
            form.submit();
        }
  </script>
</html>
```

* node package 중 하나인 `http-echo-server` 를 띄워 쉽게 요청 전송을 테스트할 수 있다.
```bash
$ npx http-echo-server
```