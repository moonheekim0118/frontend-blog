---
title: Http 요청 시, 프론트엔드에서 Timeout 구현하기

date: 2021-07-05 21:20:50

category: Frontend

draft: false
---

이번에 채용 과제를 진행하면서 학습 및 구현하게 된 사항이다.

이전에는 미처 깨닫지 못하고 있었는데, timeout은 꼭 구현해야 유저의 경험의 퀄리티가 올라가므로 알아두자!

## Timeout이 뭔데?

타임아웃을 정의하자면 아래와 같다.

- 프로그램이 일정 시간 내에 성공적으로 수행하지 않아서, 진행이 자동으로 중단되는 것이다.
- 즉 , 서버측으로 http 요청을 보냈을 때, 서버에 어떤 문제가 생겨서 올바르게 응답을 보내줄 수 없는 경우, 요청을 보낸 측에서 `제한시간 (timeout)` 을 설정하여 해당 시간이 지난 후에는 요청을 알아서 취소하는 것이다.

## 왜 필요한데?

![](https://i.imgflip.com/1j538j.jpg)

왠만한 프론트엔드에서는 Http 요청 상태는 아마 세개의 상태로 분류하고 있을 것이다.

- LOADING
- SUCCESS
- FAIL

이 때, 서버가 친절히 500 응답이라도 보내주면 FAIL 로 처리되지만 그러지 않고 응답을 아예 주지 않는 경우에는 LOADING 상태에서 머무르게 된다. <br/>

따라서 요청을 한 유저의 경우는 요청에 에러가 있는지도 모르고`로딩중입니다` 와 같은 메시지만 보게되어 유저 경험이 현저히 낮아질 수 밖에 없다. <br/>

따라서, 일정 시간이 지나도 서버 측에서 응답을 보내주지 않을 경우 프론트엔드에서 요청을 취소하고 요청 상태를 `fail` 로 처리해줘야 한다. <br/>

## 구현하기

- 편의를 위해 fetch API를 사용하겠다. (다른 라이브러리도 크게 다를 것 없다.)

### 기존 요청 함수

```javascript
async function request() {
  const resposne = await fetch('/api')
  if (!response.ok) {
    throw new Error('서버 에러')
  }
  const data = await response.json()
  return data
}
```

- async ~ await 구문을 통해 비동기 요청을 처리했다.
- 응답받은 response 객체의 ok 가 `false` 라면 에러를 throw 해준다.
- json 데이터를 파싱하여 리턴해준다.

<br/>

### Timeout 적용하기

```javascript
const controller = new AbortController() // 요청 취소 객체
const signal = controller.signal
const timeout = 8000

async function requestWithTimeout(options) {
  const timer = setTimeout(() => controller.abort(), timout)
  const resposne = await fetch('/api', {
    ...options,
    signal,
  })
  if (!response.ok) {
    throw new Error('서버 에러')
  }
  const data = await response.json()
  clearTimeout(timer)
  return data
}
```

- AbortController
  - 해당 인터페이스는 웹 요청 취소를 할 수 있도록 도와준다.
  - `signal 프로퍼티`를 fetch 요청의 옵션으로 추가해준다.
  - controller.abort() 를 호출하면 해당 fetch 요청이 취소된다.
- Timer
  - setTimout 함수를 통해 특정 timout 시간이 지나면, `controller.abort()` 을 실행하여 fetch 요청을 취소하도록 한다.
  - timout 시간 내에 요청을 잘 응답받았다면 `clearTimout` 을 통해 timout을 제거해준다.

---

이렇게 timeout 이 무엇이고, 왜 필요하고, fetch API 를 사용해 구현하는 법 까지 간단하게 알아보았다.
