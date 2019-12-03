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

## 리모트 브랜치가 삭제된 상태에서 로컬 브랜치 지우기 
* git flow 의 release 를 캔슬하는 경우 아래 명령어로 release 를 로컬에서 삭제할 수 있다
```
$ git branch -D release/19.12.03
```
* 이후에 remote 서버에서 해당 브랜치가 지워진 후 같은 브랜치 이름으로 다시 푸시를 하려고 하면 다음과 같은 에러 메시지를 볼 수 있다.
```
$ git flow release publish 19.12.03
Branch 'origin/release/19.12.03' already exists. Pick another name.
```
* 로컬에 remote-tracking 브랜치가 남아있기 때문인데, 아래와 같이 삭제해주면 된다.
```
$ git branch -dr origin/release/19.12.03
```
* remote 서버의 브랜치를 직접 삭제하려면 아래와 같은 명령어를 사용하면 된다.
```
$ git push origin --delete <branch>  # Git version 1.7.0 or newer
$ git push origin :<branch>          # Git versions older than 1.7.0
```
* https://stackoverflow.com/questions/2003505/how-do-i-delete-a-git-branch-locally-and-remotely
