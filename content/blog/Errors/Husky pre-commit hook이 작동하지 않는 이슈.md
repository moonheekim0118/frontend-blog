---
title: Husky pre-commit hook이 작동하지 않는 이슈

date: 2021-05-09 16:30:50

category: error

draft: false
---

## 문제

어느때와 같이 pretty-quick을 husky pre-commit hook으로 저장해놓았는데, 이상하게 아무리 커밋을 해도 hook이 돌아가지 않았다.

그래서 별의 별 삽질을 다 해봤는데, 결론은 최근에 업데이트된 husky 6버전에서 변경사항이 적용되어서 그랬던 것이다.

혹시 모르니 어떻게 해결했는지 뭐가 문제인지 짧게나마 기록해보겠다.

### 뭐가 문제인가

husky 레포지토리를 이리저리 탐색하고 내린 결론들이다.

기존에는 아래와 같이 package.json 파일에 husky hook을 설정해주었다.

```json
  "husky": {
    "hooks": {
      "pre-commit": "pretty-quick --staged"
    }
  }
```

하지만, 새로운 husky 6 버전에서는 더이상 위와 같은 설정이 적용되지 않는다고 한다.<br/>

나는 git hook이 아예 생성되지 않는 문제인 줄 알았는데 이슈를 샅샅히 찾아보니 설정 방법이 변경된 것 뿐이었다.

<br/>

### 해결방법 1) version4 로 다운그레이드 한다.

당연히 새로운 버전에서 문제가 생겼으므로 다운그레이드를 하면된다. 하지만 추가사항이 있다.

```
npm unistall husky
npm install -D husky@4 // 다운그레이드
```

이렇게만 하고 훅을 등록해도 제대로 동작하지 않는다. 아래 커맨드를 추가적으로 실행해주어야 한다.

```
 git config --unset core.hookspath
```

<br/>

<br/>

### 해결방법 2) 새로운 설정 방법을 적용해준다.

새로운 설정방법도 간단하다.

1. npx husky 를 설치해준다.

```
npx husky install
```

2. git hook을 등록하기 위해서, package.json의 스크립트에 아래와 같이 추가해준다.

```
{
  "scripts": {
    "prepare": "husky install"
  }
}
```

3. 원하는 hook을 추가해준다.

```
npx husky add .husky/pre-commit "npm run prettier"
```

위의 커맨드를 실행하면 .husky 폴더에 아래와 같은 파일이 생성된다.

```
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run prettier

```

<br/>

## 참고

https://typicode.github.io/husky/#/?id=migrate-from-v4-to-v6

https://github.com/typicode/husky/issues/896
