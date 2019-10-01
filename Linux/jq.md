# jq

jq 는 json 문자열 처리를 위한 커맨드라인 도구다.

https://stedolan.github.io/jq/

## 몇가지 사용 예

* pretty-print

```
$ echo '{"userId":1,"id":1,"title":"delectus aut autem","completed":false}' | jq -C '.'
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}
```

* one-line print
```
$ echo '{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}' | jq -c '.'
{"userId":1,"id":1,"title":"delectus aut autem","completed":false}
```

* get specific field
```
$ echo '{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}' | jq '.title'
"delectus aut autem"
```

* flatten nested json
```
echo '
{
  "user": {
    "properties": {
      "age": {
        "type": "keyword"
      },
      "gender": {
        "type": "keyword"
      },
      "id": {
        "type": "keyword"
      },
      "location": {
        "type": "keyword"
      }
    }
  }
}
' | jq -C '[leaf_paths as $path | {"key": $path | join("."), "value": getpath($path)}] | from_entries'

{
  "user.properties.age.type": "keyword",
  "user.properties.gender.type": "keyword",
  "user.properties.id.type": "keyword",
  "user.properties.location.type": "keyword"
}
```

[reference](http://codesd.com/item/flatten-nested-json-using-jq.html)
