---
title: <리액트 에러 해결>unmount된 컴포넌트에 state update를 시도 할 경우

date: 2021-03-29 16:30:50

category: error

draft: false
---

개발을 하다가 아래와 같은 에러가 나왔다. <br/>

```
"Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in a useEffect cleanup function.."
```

<br/>

에러의 원인 / 해결책이 모두 에러메시지에 적혀있었다.

**1.에러 원인** <br/>
이미 unmount 된 컴포넌트에서 state를 변경하려고 했기 때문에 발생한 에러다. 언마운트 된 컴포넌트에서는 리액트가 상태를 변경할 수 없으므로 메모리 누수가 발생할 것 이라고 한다. <br/>
<br/>

**2.해결책** <br/>
useEffect cleanUp 함수를 통해 비동기 작업을 취소해야 한다고 한다.
<br/>
<br/>

## 에러가 발생한 상황

나는 위 에러가 매번 발생한게 아니라 가끔씩 발생했는데, 찾아본 결과 아래와 같은 상황에서 발생 했다. <br/>

1. API 요청을 넣었다.
2. API 요청 상태 (SUCCESS, FAIL) 에 따라서 상태를 변경해주거나, 노티피케이션을 띄워주는 useEffect가 있다.
3. API 요청이 끝나기도 전에 해당 페이지에서 벗어날 경우 -> **에러 발생 !**

<br/>

즉, API 요청이 끝난 후 상태 (result.type) 에 따라서 state 가 변경될 useEffect가 있음에도 불구하고 컴포넌트가 unmount 되서 발생한 에러이다. <br/>

에러가 발생한 코드는 아래와 같다. <br/>

```typescript
const GoogleLoginButton = (): React.ReactElement => {
  const notiDispatch = useNotificationDispatch()
  const [googleAuthResult, googleAuthFetch, googleAuthSetDefault] = useApiFetch(
    googleSignIn
  )

  // 이 부분이 문제 !

  useEffect(() => {
    switch (googleAuthResult.type) {
      case SUCCESS:
        Router.push(routes.HOME)
        break
      case FAILURE:
        if (googleAuthResult.error) {
          notiDispatch(showError(googleAuthResult.error))
          googleAuthSetDefault()
        }
    }
  }, [googleAuthResult])

  const googleSignInHandler = useCallback(() => {
    googleAuthFetch({ type: REQUEST })
  }, [])

  return (
    <S.Container type="button" onClick={googleSignInHandler}>
      <S.Logo>{googleLogo}</S.Logo>
      <S.Title>{GOOGLE_LOGIN_CAPTION}</S.Title>
    </S.Container>
  )
}
export default GoogleLoginButton
```

<br/>

## 해결방법

위에서 사용한 `useApiFetch` 라는 커스텀 훅스를 변경해주어서 해결했다. useApiFetch 는 fetch Request 를 params 와 함께 실행하면 해당되는 비동기 요청을 실행하고, 실행 결과는 Result 에 저장해주는 훅스이다. 성공시 SUCCESS를, 실패시 FAILURE를 type으로 저장한다. <br/>

즉, 비동기 요청으로 변경된 `googleAuthResult` 에 따라서 **useEffect 내부 연산이 실행되어야 하는데 언마운트 되어서 발생한 문제**이므로, 에러메시지에서 알려준대로 **cleanUp 함수를 통해 useEffect 내부 연산이 실행될 가능성을 제거**해주면 된다. <br/>

즉 `googleAuthResult`의 type 을 ' ' 으로 변경해주므로 해결했다.

  <br/>

```typescript
.... ,

const useApiFetch = <T = null>(
  apiRequest: (...args: any[]) => APIResponse<T>
) => {
  const initialState: Result<T> = {
    type: '',
  };
  const [result, dispatch] = useReducer<Reducer<T>>(reducer, initialState);
  const fetched = useRef<boolean>(false);

  useEffect(() => {
    return () => {
      setDefault(); // 해결 !
    };
  }, []);

  useEffect(() => {
    if (result.type === REQUEST && !fetched.current) {
      fetchData<T>(apiRequest, dispatch, result.params);
      fetched.current = true;
    }
    if (result.type === SUCCESS || result.type === FAILURE) {
      fetched.current = false;
    }
  }, [result, apiRequest]);

  const setDefault = useCallback(() => dispatch({ type: '' }), []);

  return [result, dispatch, setDefault] as const;
};

export default useApiFetch;
```

<br/>

<br/>

## 나가며

생각보다 굉장히 간단하게 해결 되는 문제였다 . <br/>

만약 위와 같이 비동기 연산에 대한 커스텀 훅스가 없을 경우에는, 컴포넌트 내부에서 클린 업 함수를 통해 변경될 state를 변경해주면 된다.

<br/>

<br/>

## Reference

[React state update on an unmounted component](https://www.debuggr.io/react-update-unmounted-component/#:~:text=If%20we%20look%20again%20at,in%20a%20useEffect%20cleanup%20function.)
