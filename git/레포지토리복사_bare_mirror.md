# 레포지토리 복사
- 특정 레포지토리의 파일을 그대로 복사해서 가져오는 것이 아니라
- commit history까지 모두 복사해서 가져오는 방법은?

## source 레포 가져오기
- 어차피 복사하고나면 필요 없는거 다 가져올 필요가 있을까. 
- 이를 위해 git 메타 데이터만 가져오는 방법이 있다. 
```bash
$ git clone --bare {원본레포지토리URL}
```

## destination 레포에 붙여넣기
- 이제 붙여넣으면 된다.
- source 디렉터리로 이동한 후,
- mirror 옵션을 넣어서 push하면 된다.
```bash
$ cd {source}
$ git push --mirror {목적지레포지토리URL}
```


---

### 참고
- [공식문서](https://docs.github.com/ko/repositories/creating-and-managing-repositories/duplicating-a-repository)
