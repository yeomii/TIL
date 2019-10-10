# Git CheatSheet

## 특정 파일만 이전 커밋 상태로 되돌리기
```sh
# commit id 를 지정해서 되돌리기
$ git checkout <commit-id> <file-to-revert>

# 바로 이전 커밋으로 되돌리기
$ git checkout HEAD^ <file-to-revert>
```

## 연속된 특정 커밋들을 다른 브랜치 기반으로 옮기기
* rebase
    * https://stackoverflow.com/questions/2369426/how-to-move-certain-commits-to-be-based-on-another-branch-in-git/11965051
```
git rebase --onto <branch-name> <commit-one> <commit-two>
```
    
* 브랜치 옮기기
```
git branch -f <branch-name> <new-tip-commit>
```

## 브랜치의 가장 최근 커밋을 삭제하기 (local 에만 남아있는 경우)
```
# 이전 커밋으로 reset
$ git reset HEAD^
# 변경된 내용 버리기
$ git checkout -- .
```
