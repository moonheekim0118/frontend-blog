---
title: <Trouble shooting> Next.js 에서 document is not defined 문제 해결하기

date: 2021-02-02 01:00:50

category: Frontend

draft: false
---

<br/>

새로 진행하고 있는 `반려견 산책로 리뷰 서비스` 에서 모달창을 구현할 일이 생겼습니다. 그래서 자연히 리액트의 **Portal**을 사용하기로 했습니다. Portal은 현재 컴포넌트가 삽입된 위치에 종속되지 않고, 컴포넌트를 특정 엘리먼트에 붙여줄 수 있도록 도와주기 때문입니다. Portal의 일반적인 사용 방식은 아래와 같고 아래의 코드가 문제를 발생시켰습니다. <br/>

### ✔️ 문제 코드

```javascript
const Modal = () => {
  return ReactDOM.createPortal(
    <>
      {root element에 붙여줄 코드 작성}
    </>,
    document.getElementById('modal-root') // Element
  )
}
```

Portal사용시에는 `document.getElementById` 를 통해서 해당 컴포넌트를 붙여 줄 엘리먼트를 먼저 찾아줘야합니다. <br/>

하지만 이렇게 코드를 작성했더니 next는 바로 `document is not defined`라는 에러를 뱉어냈습니다.

음? 클라이언트 서버에서 어떻게 document를 찾을 수 없다는건가? 하고 생각할 수 있습니다만, Next.js의 동작원리를 알면 매우 당연한 에러임을 알게 됩니다.

<br/>

### ✔️ Next.js의 동작원리 - 서버사이드 렌더링

<br/>

Next.js의 주된 목적 중 하나는 `초기 렌더링 속도의 향상`이며, 초기의 서버사이드 렌더링은 이 목적을 달성해줍니다. 먼저 클라이언트 사이드 렌더링(CSR)과 동작 방식을 비교해보도록 하겠습니다. <br/>

### 클라이언트 사이드 렌더링 (CSR)

1. 유저가 특정 url을 통해 데이터를 요청
2. 요청을 받은 서버는 HTML 파일 ( JS 링크 포함) 을 브라우저(클라이언트 서버)에 전송
3. 브라우저에서 HTML 파일을 다운로드
4. 브라우저에서 프레임워크 혹은 라이브러리를 구동하여 코드 실행
5. **렌더링이 시작되고 이벤트 발생 준비 완료**, _유저와 인터렉션 가능 , 이제 유저가 뷰를 볼 수 있음_

<br/>

### 서버 사이드 렌더링 (SSR)

1. 유저가 특정 url을 통해 데이터를 요청
2. 서버(next에서는 node 서버)가 렌더링 가능한 html 파일(+ 서버에서 설정시 백엔드 서버측 데이터 포함 가능 ) 을 만들어서 브라우저로 전송
3. 브라우저가 받은 HTML을 다운로드 하여 **렌더링이 시작되고 이벤트 발생 준비 완료** , _이제 유저가 뷰를 볼 수 있음_
4. 자바스크립트 코드를 다운로드
5. 브라우저가 프레임워크를 구동하여 _유저와 인터렉션 가능_

> 참고
>
> - Next는 page 에 해당하는 js 코드를 빠른 빌드가 목적인 dev 환경에서는 미리 만들어 놓지 않고, 빠른 초기 렌더링이 목적인 production 환경에서는 미리 만들어 놓습니다.

<br/>

Document를 포함한 DOM API가 정의되는 시점은 **뷰와 함께 유저와 인터렉션이 가능한 코드가 모두 클라이언트에 로드 되었을 때** 입니다. <br/> 하지만 **우리가 정의한 document를 포함한 코드는 DOM API가 아직 정의 되기 전인 노드서버 측에서 먼저 접근되기 때문에** undefined 에러가 나오는 것입니다.

즉 SSR 단계에서 아직 렌더링이 끝나지 않은 상태에서 정의되지 않은 DOM API에 접근하려다보니 발생한 에러였습니다. <br/>

그렇다면 이를 어떻게 해결해야 할지 고민했습니다. 제가 생각한 방법은 아래와 같습니다. <br/>

### ✔️ 해결방법

<br/>
접근해야 하는 Element를 담을 State를 null로 정의해놓고, useEffect 훅스를 통해서 컴포넌트 렌더링이 완료 된 후에 document에 접근하도록 하였습니다. 컴포넌트 렌더링이 완료 된 후에 실행되기 때문에 클라이언트 단에서 실행되는 것이 보장되었기 때문입니다.

그리고 위와 같은 로직이 앞으로 재사용 될 가능성이 있다고 보았기에 커스텀 훅스인 useElement 로 아래와 같이 구현하였습니다. <br/>

### useElement

```typescript
import { useState, useEffect } from 'react'

const useElement = (id: string) => {
  const [element, setElement] = useState<Element | null>()

  useEffect(() => {
    setElement(document.querySelector(`#${id}`))
  }, [])

  return element
}

export default useElement
```

<br/>

Props 로 id 를 받아와서 다른 컴포넌트에서도 재사용 가능하도록 하였고, element를 반환해주어서 Modal 컴포넌트에서는 아래와 같이 사용됩니다. <br/>

### 사용 예제

```typescript
mport React from 'react';
import ReactDOM from 'react-dom';
import useElement from '../../hooks/useElement';
import styled from '@emotion/styled';

const Modal = () => {
  const root = useElement('modal-root');

  return root
    ? ReactDOM.createPortal(
        <>
       	 {root element에 붙여줄 코드 작성}
        </>,
        root
      )
    : null;
};
```

이렇게하면 document에 접근하는 시점이 SSR이 끝난 후임이 보장되어 에러가 나지 않습니다.

<br/>

### ✔️ 마치며

사실 처음에는 대체 왜 document가 클라이언트 단에서 접근이 되지 않는지 이해가 가지 않았습니다. 하지만 곧 이 문제 해결을 위해 Next.js 의 SSR 구동방식을 알아보면서 너무 당연한 에러였음을 알게 되었습니다. 너무 당연하게도 Next의 SSR 기능에 의해 dom api 정의의 시점과 Next 에서 해당 코드를 생성하는 시점이 맞지 않았던거죠. <br/>

또한 그동안 희미하게만 알고 있던 Next.js의 초기 SSR 방식을 조금 깊이 알게 되어서 매우 뿌듯하기도 하고, 앞으로 갈길이 멀다고 느끼기도 합니다. 사실 이전에는 서버사이드렌더링 === 백엔드에서 데이터를 초기에 미리 가져와서 뷰를 빠르게 구성하기 위함 이라고만 생각했습니다. <br/>물론 이도 맞지만, 애초에 HTML 파일을 렌더링 가능한 상태로 프론트 서버단에서 만들어와주기 때문에 초기 구동 자체가 빠름을 이해 할 수 있었습니다.

추후에는 꼭 Next.js의 동작 원리에 대해 더 깊이 공부하고자 합니다.

 <br/>
