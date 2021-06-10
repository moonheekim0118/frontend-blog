---
title: 리액트의 useCallback, 언제 써야할까

date: 2021-06-10 11:00:50

category: frontend

draft: false
---

리액트 Hooks 로 개발을 하다보면, 컴포넌트가 리렌더링 될 때 컴포넌트 내부에 있는 모든 일반함수가 리렌더링 되는 것을 알 수 있다.

이것은 매우 비효율적인 작업이기 때문에 리액트에서는 useCallback 이라는 api 를 제공해준다. 그래서 나는 이제껏 컴포넌트 내부에서 정의되는 대부분의 함수에 useCallback을 사용해왔다. 왜냐하면, 불필요한 리렌더링을 막고싶어서이다.

하지만 때때론, useCallback으로 함수를 감싸는게 더 부담되는 작업일 수 있다. 그렇게 되면 useCallback으로 감싸진 함수는 결국 오버엔지니어링의 산물이 되고, 성능 개선하려다가 성능 저하만 야기시키는 셈이 된다.

그러면 언제 useCallback을 써야하고, 언제 쓰지 말아야할까?

<br/>

<br/>

## useCallback이 하는 일

- **함수를 메모라이징** 해준다. 따라서 하나의 컴포넌트가 리렌더링 되었을 때, 해당 함수가 새로 생성되지 않고 이전에 메모라이징 된 함수가 사용된다.
- **Dependency** 를 추가하여, Dependency에 추가된 값이 변할 때 `함수가 업데이트` 될 수 있도록 변경할 수 있다. 함수를 메모라이징 해놓는다면 함수 내부에서 사용되는 상태값 역시 메모라이징 된 상태이다. 따라서, 상태값에 따라 함수 연산을 달리 해야한다면 Dependency 배열에 상태값을 넣어주면 된다.
- 결국, 우리는 하나의 함수를 정의 할 때 Dependecy를 할당하고, useCallback 을 사용함으로써 메모라이징 연산을 추가하게 된다. 또한, useCallback과 함께라면, 원본 함수는 가비지 컬렉팅 되지 않고, 메모리에서 계속해서 한 공간을 차지하게 된다. 따라서, useCallback API를 사용하는 것은, 성능 최적화에 기여하겠지만, 분명 **그 만큼의 비용이 드는 성능최적화**인 것이다.

<br/>

<br/>

## useCallback을 써야하는 경우

**렌더링 성능이 최적화 된 자식 컴포넌트에 함수를 Props로 넘겨주는 경우**

여기서 렌더링이 최적화 된이란, *자식 컴포넌트가 Props 변경의 영향을 많이 받지 않아야하는 경우*를 일컫는다. 즉, 컴포넌트를 React.memo로 감싸고 있는 경우가 그러하다.

<br/>

<br/>

## useCallback을 정말 절대 쓰지 말아야 하는 경우

만약 연산이 복잡한 함수를 useCallback으로 감싸고 해당 컴포넌트에 렌더링이 자주 일어난다면 성능 최적화를 위한 것이라고 볼 수 있겠다. 하지만 아래와 같은 경우는 정말 useCallback을 사용하는게 괜한 메모리낭비이므로 사용하지 말도록 하자.

**단순히 함수 내부에서 setState나 dispatch 함수를 호출하는 경우**

```jsx
cosnt handleChange = useCallback((newState)=>{ setState(newState) },[]);
```

왜냐하면, 이미 리액트 자체에서 useState 와 useDispatch 에 대한 성능 최적화를 보장해주고 있기 때문이다. 렌더링이 새로 되어도 setState함수와 dispatch 함수는 새로 생성되지 않는다.

당연하다. 그들도 Hooks API 이기 때문이다.

<br/>

<br/>

## 결론

### useCallback 혹은 useMemo를 쓰자!🥳

- 자식 컴포넌트에 함수를 props로 넘겨주는데 , 해당 자식 컴포넌트에 _넘겨주는 함수_ 때문에 불필요한 리렌더링이 일어난다고 판단될 경우
- 함수 자체가 매우 복잡하거나, 비용이 많이 드는 경우

### useCallback 굳이 써야하니 ? 🙄

- 일반 함수의 경우, toggle이나 incrememt 같은 단순한 연산들은 굳이 useCallback을 쓸 필요가 있나 재고해 볼 필요가 있다. 이 경우 그냥 일반 함수로 표기해도 성능상 별 문제가 없다.
- 오히려 함수가 복잡하지 않으면 useCallback을 쓰지 않는 편이 성능상 좋다. 계속해서 함수가 메모리에 남기 때문이다.

### useCallback 절대 쓰지마! 🤮

- setState나 dispatch를 단순 호출할 경우. 불필요한 연산을 더하는 셈이므로 절대 쓰지마!

<br/>

<br/>

## 참고

[When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

[AHA Programming 💡](https://kentcdodds.com/blog/aha-programming)
