# git 관련

## VScode에서 원격 저장소 URL 변경하기

기존 git remote 확인하기
```
git remote -v
```

remote repository 변경하기
```
git remote set-url origin https://GIT_REPO@example.com/GIT_REPO/repo.git
```

변경 확인
```
git remote -v
```