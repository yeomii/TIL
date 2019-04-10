# Git CheatSheet

## 특정 파일만 이전 커밋 상태로 되돌리기
```sh
# commit id 를 지정해서 되돌리기
$ git checkout <commit-id> <file-to-revert>

# 바로 이전 커밋으로 되돌리기
$ git checkout HEAD^ <file-to-revert>
```
