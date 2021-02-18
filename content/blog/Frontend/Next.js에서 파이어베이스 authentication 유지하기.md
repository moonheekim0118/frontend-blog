---
title: Next.js에서 파이어베이스 authentication 유지하기

date: 2021-02-19 01:00:50

category: Frontend

draft: false
---

이번에는 프론트엔드 - 리액트 / next.js 그리고 백엔드 파이어베이스로 이루어진 프로젝트를 진행하다가 마주한 파이어베이스의 auth 지속성과 next.js 의 충돌을 해결한 과정을 정리해보고자 합니다. <br/>

<br/>

## 문제 파악하기

파이어베이스 공식문서에는 아래와 같이 나와 있습니다.

> Firebase JS SDK를 사용하면 인증 상태를 유지하는 방식을 지정할 수 있습니다. 로그인한 사용자가 명시적으로 로그아웃할 때까지 무기한 유지할지, 창을 닫으면 상태를 삭제할지, 아니면 페이지 새로고침 시 삭제할지 지정할 수 있습니다.

<br/>

따라서 파이어베이스가 제공하는 인증 지속성에는 세가지 유형이 있습니다.

1. **NONE** <br/>
   새로고침 시 인증이 삭제됩니다.
2. **SESSION** <br/>
   현재 탭에서 인증이 유지되고, 탭이 닫히면 인증이 삭제됩니다.
3. **LOCAL** <br/>
   브라우저 탭이 닫혀도 인증이 유지됩니다.

그리고 저는 로그인 하고 나서 탭이 닫힐 때 까지 인증이 지속되도록, 즉 SESSION 지속 방법으로 구현하고자 했습니다.

파이어베이스 공식문서에 따르면 아래와 같은 코드를 파이어베이스 설정시에 넣어주면 된다고 하더군요.

```
firebase.auth().setPersistence(firebase.auth.Auth.Persistence.SESSION)
```

<br/>

그래서 기쁜마음으로 `firebase.js` 내부에서 위와 같이 설정을 해줬더니 아래와 같은 에러가 떴습니다.

```
 [Error]: The current environment does not support the specified persistence type.] {
  code: 'auth/unsupported-persistence-type',
  message: 'The current environment does not support the specified persistence type.',
```

간략하게 말하면 **현재 환경은 SESSION 지속성을 지원하지 않는다**는 것입니다. <br/>

왜일까요 ?

`Next.js는 서버사이드 렌더링`을 해주기 때문입니다. 이전 포스트에서 document가 undefined가 되었던 것과 마찬가지로, 클라이언트 서버가 아닌, Node js 환경에서는 오로지 **NONE** 지속성만 지원해주기 때문에 이런 문제가 발생하게 됩니다.

따라서 파이어베이스 인증 지속성에 관한 설정 없이 인증을 지속시켜야했습니다. <br/>

클라이언트에서 설정하고 서버 측에서 이 설정을 확인할 수 있는 것은 딱 하나 ..바로 브라우저에 저장되는 **세션 쿠키 (Session Cookie)** 입니다. 아래에서 어떻게 해결하는지 더 깊게 알아보겠습니다.

<br/>

<br/>

## 들어가기 전에 , 쿠키 이해하기

해결 방법으로 들어가기 전에 간단하게 쿠키에 대해 정리해보도록 하겠습니다. <br/>

### 세션 쿠키에 대하여

저는 해당 프로젝트에서 세션쿠키를 이용합니다. <br/>

세션 쿠키는 브라우저 탭이 닫히면 브라우저 쿠키 스토어에서 삭제됩니다. 이에반해 `지속 쿠키`의 경우는 명시한 시간동안 쿠키 스토어에 지속됩니다.

_세션 쿠키 역시 Max-Age 옵션을 통해 지속 시간을 지정할 수 있습니다._

<br/>

### 쿠키의 흐름

1. 클라이언트 측에서 HTTP 요청을 통해 쿠키 생성을 서버측으로 요청합니다.
2. 서버는 받은 쿠키의 옵션을 지정하고, Set-Cookie 헤더를 통해 쿠키를 저장하여 응답합니다.
3. 이제 쿠키는 브라우저 쿠키 스토어에 저장됩니다.
4. 이제 서버측에서 세션쿠키를 참고하며 비즈니스 로직을 수행합니다.

<br/>

### 쿠키 옵션

서버에서 쿠키를 생성할 때 쿠키 옵션을 설정할 수 있습니다. 주로 사용되는 옵션들만 간략히 살펴보도록 하겠습니다.

- Max-Age
  - 쿠키 만료 시간
- Domain
  - 쿠키가 보내질 도메인 주소를 입력합니다. 생략된 경우 자동으로 현재 주소로 보내집니다.
- Path
  - Domain 내부에서 쿠키가 유효한 URL을 지정해줍니다. 주로 '/' 으로 지정해주면 됩니다.
- HttpOnly
  - 자바스크립트 코드가 `document.cookie` 속성을 통해 쿠키에 접근하는 것을 막아줍니다.
- Secure
  - https 요청에 한해서 브라우저가 쿠키를 서버로 전송하는 것을 허용해줍니다.

<br/>

## 문제 해결하기

아주 간단하게 해결 프로세스를 설명하자면 아래와 같습니다.

1. 유저가 로그인을 합니다.
2. 로그인된 정보를 통해 쿠키를 생성하여 브라우저에 저장해줍니다.
3. 서버사이드 렌더링 시, 저장된 쿠키를 통해 인증 여부를 판단하여 props로 보내줍니다.

<br/>

그렇다면 가장 큰 문제는 어떻게 **저장된 쿠키**와 **특정 유저 사이**의 관계를 알 수 있느냐가 됩니다.

<br/>

### 파이어베이스의 토큰 이용하기

파이어베이스는 사용자 인증 후에 응답으로 아래와 같이 **ID 토큰**을 발급해줍니다. 이 ID 토큰으로 로그인한 사용자를 식별 할 수 있습니다. </br>

```javascript
const response = await auth.signInWithEmailAndPassword(email, password)
const token = response.user.getIdToken()
```

<br/>

그렇다면 `2. 로그인된 정보를 통해 쿠키를 생성하여 브라우저에 저장` 에서 **로그인된 정보 === ID 토큰** 이 됩니다.

그렇다면 ID 토큰 값에 따라 특정 유저 정보를 어떻게 판별하고 인증 할 수 있을까요? 이를 위해서는 **파이어베이스 admin SDK**를 별도로 설정해주어야 합니다. <br/>

파이어베이스 admin SDK 설정은 [이 곳](https://firebase.google.com/docs/admin/setup?hl=ko) 에 매우 자세하게 설명되어 있습니다.

<br/>

### 프로세스에 살 붙이기

지금까지의 프로세스는 아래와 같습니다.

1. 유저가 로그인을 합니다.
2. 로그인 후에 발급받은 ID 토큰을 통해 admin SDK로 사용자를 인증하여 쿠키를 생성하여 브라우저에 저장해줍니다.
3. 서버사이드 렌더링 시, 저장된 쿠키를 통해 인증 여부를 admin SDK로 판단하여 props로 보내줍니다.

<br/>

여기에서 1번과 2번 과정을 세분화 시켜보도록 하겠습니다. <br/>

- 유저가 로그인을 합니다. 로그인 후에 토큰을 발급 받고, 발급받은 토큰을 서버측으로 요청을 보내는 함수에 토큰을 파라미터 값으로 넣어 호출해줍니다.

```javascript
export const signIn = async (email: string, password: string) => {
  try {
    const auth = firebase.auth();
    const response = await auth.signInWithEmailAndPassword(email, password);
    if (response && response.user) {
      // 발급받은 토큰을 서버측으로 보내주는 함수 호출
      await postUserToken(await response.user.getIdToken());
    }
    return { isError: false, errorMessage: '' };
  } catch (error) {
    return { isError: true, error.code };
  }
};
```

<br/>

### 여기서 잠깐 ! Next.js API Route 이용하기

쿠키를 안전하게 생성해주기 위해서는 서버 측의 도움이 필요합니다. 이를 위해서 저는 Next.js 에서 제공하는 API route를 이용했습니다. <br/>

Next.js에서 `/pages/api/**` 아래에 있는 파일은 Next가 알아서 page가 아니라 **엔드포인트(Endpoint)**로 인식을 합니다. 따라서 서버사이드 번들링에만 포함을 시키고 클라이언트 사이드 번들에는 포함하지 않습니다. 즉 서버측의 엔드포인트가 됩니다. <br/>

따라서 앞으로`/pages/api` 에 엔드포인트를 생성하여 쿠키를 처리해보도록 하겠습니다. <br/>

_Next.js의 API route에 대해서는 [공식문서](https://nextjs.org/docs/api-routes/introduction)에서 더 자세히 확인하시는 것을 추천드립니다._ <br/>

<br/>

**발급 받은 토큰으로 쿠키를 생성해주도록 서버측으로 api 요청(POST)을 보냅니다.**

- 쿠키를 생성해주는 요청이므로 POST 요청입니다.
- next pages에 `/api/auth` 를 추가하여 이곳에 api 요청을 보냅니다.
- 환경변수에 현재 프론트엔드 서버 주소를 `BASE_API_URL`로 설정해줍니다. 현재는 개발모드이므로 로컬주소를 저장했습니다.
- data로 token을 삽입하여 보내줍니다.

```javascript
// 받은 토큰으로 쿠키 생성 api 요청
export const postUserToken = async token => {
  const path = '/api/auth'
  const url = process.env.BASE_API_URL + path
  const data = { token: token }
  const headers = {
    'Content-Type': 'application/json',
  }
  const response = await axios.post(url, data, { headers })
  return response
}
```

<br/>

**`/api/auth` 에서 가져온 토큰을 통해 쿠키를 생성하여 응답을 보내줍니다.**

- 파이어베이스 Admin을 통해 토큰을 인증받아서, 쿠키를 생성할 수 있습니다.
  - 주의점 : 파이어베이스 Admin 관련 함수는 Next.js가 알아서 클라이언트에 빌드하지 않으므로, 서버측에서만 실행가능합니다.
- 여기서 expiresIn 은 10분으로 , 토큰이 최초에 생성된 시간에서 10분이 경과하면 다시 로그인을 해야하도록 구현 했습니다.
- 쿠키가 생성되면 헤더에 쿠키를 담아서 보냅니다.

```typescript
import { serialize } from 'cookie'
import getFirebaseAdmin from '../../../firebase/admin'

const EXPIRE = 60 * 60 // 세션 쿠키 만료 기간

const auth = async (req, res) => {
  try {
    const admin = await getFirebaseAdmin()
    const expiresIn = EXPIRE * 1000 // 1hour
    if (req.method === 'POST') {
      let idToken = req.body.token // 토큰 가져오기
      const decodedIdToken = await admin.auth().verifyIdToken(idToken) // 파이어베이스 토큰 인증
      let cookie
      if (new Date().getTime() / 1000 - decodedIdToken.auth_time < EXPIRE) {
        cookie = await admin.auth().createSessionCookie(idToken, { expiresIn })
      } else {
        res.status(401).send('Recent Sign In Required!')
      }

      // 쿠키 생성 완료 -> 토큰 인증 완료 -> authentication 완료
      if (cookie) {
        const options = {
          httpOnly: true,
          secure: process.env.NODE_ENV === 'production', // production 시 secure
          path: '/',
        }
        res.setHeader('Set-Cookie', serialize('user', cookie, options)) // 쿠키 set
        res.status(200).send({ response: 'Succesfull logged in' })
      } else {
        // Authentication 잘못됨
        res.status(401).send('Invalid Authentication')
      }
    }
  } catch (error) {
    res.status(500).send('server Error')
  }
}

export default auth
```

<br/>

이제 인증 정보가 필요한 페이지에서 서버사이드렌더링 설정을 통해서 , 서버측에서 쿠키를 인증해야합니다. <br/>

이전에는 토큰을 인증하여 토큰을 쿠키로 변환했으니, 쿠키를 토큰으로 인증하여 토큰 정보를 가져오는 것도 파이어베이스 admin 을 통해서 할 수 있습니다. <br/>

**verifyCookie.ts**

- 받아온 쿠키를 파이어베이스 admin 을 통해서 인증하고, 인증 여부와 인증된 사용자 정보를 반환해줍니다.
- 쿠키가 인증되면 `{ bAuth:true , useremail:이메일주소}` 를 반환해줍니다.

```typescript
import 'firebase/auth'
import getFirebaseAdmin from '../../firebase/admin'

const verifyCookie = async cookie => {
  try {
    const admin = await getFirebaseAdmin()
    if (!admin) return null
    let userId = ''
    let bAuth = false
    const decodedClaims = await admin.auth().verifySessionCookie(cookie, true)
    if (decodedClaims) {
      bAuth = true
      userId = decodedClaims.userId
    }
    return {
      authenticated: bAuth,
      userId,
    }
  } catch (error) {
    return null
  }
}

export default verifyCookie
```

<br/>

이제 쿠키를 인증하는 함수도 만들었으니, `getServerSideProps` 함수를 통해 서버사이드 렌더링 시 서버측에서 쿠키를 인증하도록 구현해보겠습니다. <br/>

여러 페이지에서 사용되므로 ,하나의 함수로 빼내어서 getServerSideProps 함수의 리턴값으로 넣어줍니다.

**getServerSideProps in pages**

```javascript
import getAuthentication from '../libs/getAuthentication'

export const getServerSideProps = context => getAuthentication(context)
```

**getAuthentication 함수**

- `nookies`의 `parseCookies` 메서드를 통해서 context 내부에 담겨있는 req안에 있는 쿠키를 파싱합니다.
- verifyCookie함수로 쿠키를 인증하고, 결과를 props로 내보냅니다.

```javascript
import { parseCookies } from 'nookies'
import 'firebase/auth'
import verifyCookie from '../remotes/verifyCookie'

const getAuthentication = async context => {
  let propsObject = {
    authenticated: false,
    userId: '',
  }
  const cookies = parseCookies(context)

  if (cookies.user) {
    const authentication = await verifyCookie(cookies.user)
    propsObject.authenticated = authentication
      ? authentication.authenticated
      : false
    propsObject.userId = authentication ? authentication.userId : ''
  }

  return {
    props: propsObject,
  }
}

export default getAuthentication
```

<br/>

이렇게 하면 `props.authenticated` 를 통해 로그인 여부를 파악 할 수 있습니다.

<br/>

### 로그아웃 처리하기

그렇다면 로그아웃은 어떻게 처리해야할까요? 파이어베이스에서 `logout`을 해준다고 해서 우리가 브라우저에 저장한 세션쿠키가 자동으로 삭제되지는 않습니다 . <br/>

따라서 로그아웃 - 세션삭제도 직접 설정해주어야 합니다.<br/>

하지만 유의사항이 있습니다. 보안상 `http-Only` 쿠키로 설정을 해주었기 때문에, 자바스크립트에서 쿠키를 직접 삭제할 수 는 없습니다. <br/>

따라서 다시 **Next.js 의 API 라우트**를 이용하여 서버측에서 처리해주도록 했습니다. <br/>

사용자가 로그아웃을 요청하면

1. 먼저 파이어베이스의 로그아웃 함수를 실행합니다.
2. `/api/removeAuth` 엔드포인트로 쿠키 삭제 요청을 보냅니다.

<br/>

코드를 통해 자세히 알아보도록 하겠습니다. <br/>

**signOut**

- `firebase.auth()` 의 `signOut()` 함수를 호출합니다.
- `removeCookie()` 함수를 호출합니다. 이 함수는 쿠키를 삭제해주는 엔드포인트로 요청을 보내는 함수입니다.

```typescript
export const signOut = async (): Promise<ReqResult> => {
  try {
    const auth = firebase.auth();
    await auth.signOut();
    await removeCookie(); // remove token
    return { isError: false };
  } catch (error) {
    const errorMessage = errorExTxt(error.code); // get Correct ErrorMessage
    return { isError: true, errorMessage };
  }
```

<br/>

**removeCookie**

- axios 요청으로 만들어놓은 엔드포인트에 요청을 보냅니다. 직접적인 삭제 요청은 아니므로 post 메서드로 처리했습니다. 자세한건 아래에서 더 보도록 하겠습니다.

```typescript
const removeCookie = async (): Promise<AxiosResponse<any> | Error> => {
  try {
    const path = '/api/removeAuth'
    const url = process.env.BASE_API_URL + path
    const headers = {
      'Content-Type': 'application/json',
    }
    const response = await axios.post(url, { headers }) // remove token
    return response
  } catch (error) {
    return error
  }
}
```

<br/>

**/api/removeAuth**

- 다른 미들웨어를 사용하지 않고, 쿠키를 삭제하는 방법은 아래와 같습니다. 즉 위에서 쿠키를 생성해줬던 것처럼, `Set-header`를 이용하여, 같은 이름(user) 의 쿠키를 생성(=갱신)해주되, 내용은 비어있고 `maxAge:-1` 로 생성해주면 쿠키가 생성됨과 동시에 만료됩니다.

```typescript
import { serialize } from 'cookie';
import type { NextApiRequest, NextApiResponse } from 'next';

// remove Session-Cookie
const removeAuth = (req: NextApiRequest, res: NextApiResponse) => {
  res.setHeader(
    'Set-Cookie',
    serialize('user', '', {
      maxAge: -1,
      path: '/',
    })
  );
  res.status(200).send('ok');
};

export default removeAuth;
```

<br/>

이렇게 로그아웃에 대한 로직을 구현할 수 있습니다.

<br/>

### Authenticated 정보 클라이언트 사이드에서 받아오기

프로젝트를 `Static-Generation`으로 유지하고 싶다면 `getServerSideProps`의 사용을 피해야합니다.<br/>

저 같은경우는 이번 프로젝트는 getServerSideProps를 사용하지 않기로 결정했기 때문에, 아래 방법으로 변경했습니다. <br/>

이를 위해서는 서버사이드에서 쿠키 정보를 파싱하여 분석하는게 아니라, 렌더링 이후 `useEffect`에서 Next.js API 엔드포인트로 요청을 보내는 방법이 있습니다. 자세한 과정은 아래와 같습니다. <br/>

1. 페이지가 렌더링이 됩니다.
2. useEffect를 이용해 엔드포인트에 요청을 보내서 세션쿠키가 유효한지 확인합니다.

 <br/>

코드를 보며 더 자세히 알아보도록 하겠습니다. <br/>

**api/loginCheck**

- 서버에서 `req.cookie`에 담겨진 쿠키를 파싱한 후, 아까 위에서 작성했던 `verifyCookie` 함수를 통해 세션쿠키의 유효성을 검증하여 authenticated 여부를 반환해줍니다. 이 때 verifyCookie의 설정을 통해 파이어베이스에서 유저 정보를 가져올 수도 있습니다.

```javascript
import verifyCookie from 'libs/verifyCookie'
import type { NextApiRequest, NextApiResponse } from 'next'

// check and verify Session Cookie
const loginCheck = async (req: NextApiRequest, res: NextApiResponse) => {
  try {
    let authInfo = {
      isLoggedIn: false,
    }
    const cookie = req.cookies
    if (cookie.user) {
      const authentication = await verifyCookie(cookie.user)
      if (authentication) {
        authInfo = authentication
      }
    }
    res.status(200).json(authInfo)
  } catch (error) {
    res.status(401).json({ isLoggedIn: false })
  }
}

export default loginCheck
```

<br/>

**useUser : 로그인 여부를 받아오고 , 원한다면 리다이렉트도 해줍니다**

- 이제 위의 엔드포인트에 요청을 보내주는 로직을 만듭니다. **swr**을 사용하여 API에 정보(로그인 여부)를 가져옵니다. swr을 사용하시지 않으신다면, fetch 나 axios를 사용하셔도 무관합니다. 여러 페이지에서 사용될 수 있으므로 커스텀 훅스로 재사용성을 높였습니다.
- 만약, Auth 상태에서만 접근하거나 Not Auth 상태에서만 접근할 수 있는 페이지라면 리다이렉트를 할 수 있도록 `redirectTo` porps를 통해 리다이렉션 주소를 받아옵니다. `redirectIfFound`는 인증된 경우 리다이렉션을 하는가, 아니면 그 반대인가를 알려줍니다.

```typescript
import { useEffect } from 'react'
import Router from 'next/router'
import useSWR from 'swr'
import { UserType } from 'types/User'

interface Props {
  /** path for redirection */
  redirectTo?: string
  /** true if it should be redirected when user is found */
  redirectIfFound?: boolean
}

const useUser = ({ redirectTo, redirectIfFound = false }: Props = {}): {
  user: UserType
  mutateUser: (
    data?: any,
    shouldRevalidate?: boolean | undefined
  ) => Promise<any>
} => {
  const dispatch = useLoginInfoDispatch()
  const { data: user, mutate: mutateUser } = useSWR('/api/loginCheck')

  useEffect(() => {
    // if data is not yet here
    if (!redirectTo || !user) return

    /** when it needs to be redirected */
    if (
      (redirectTo && !redirectIfFound && !user.isLoggedIn) ||
      (redirectIfFound && user.isLoggedIn)
    ) {
      Router.push(redirectTo)
    }
  }, [user, redirectTo, redirectIfFound])

  return { user, mutateUser }
}

export default useUser
```

<br/>

위와 같은 방법으로 `getServerSideProps`를 사용하지 않고도, 클라이언트 측에서 API에 요청을 보내어 세션쿠키의 유효성을 검증할 수 있습니다. 하지만 렌더링 이후에 따로 데이터를 fetching 하여 받아오기 까지 시간이 걸리므로 주의해야합니다.

<br/>

<br/>

### 마치며

이번 프로젝트를 하면서 가장 고민을 많이 할 수 밖에 없었던 문제였습니다. 서버를 알아서 처리해주는 파이어베이스를 믿고 있다가, 서버사이드렌더링을 하는 Next.js 에 의해 의외의 문제를 마주하게 되었으니까요.<br/>

하지만 이번 기회를 통해 Next.js의 API routes에 대해서도 배웠고, 쿠키의 작동방식에대해서 더 깊게 배우게 된 것 같아서 뿌듯합니다. 특히나 Netx Js 의 API routes는 파이어베이스를 사용하시는 분들이라면 정말 요긴하기 쓸 수 있을 것 같습니다. ! 예를들어 파이어베이스의 admin SDK는 Next.js가 클라이언트 측에 빌드하지 않습니다. 그래서 Module not found 에러가 뜨는데, 이 때 admin SDK 함수를 사용하기 위해서는 Next.jS API에 fetch를 한번 보내주고, 해당 API에서 SDK 관련 함수를 호출해주면 됩니다. 이와 관련한 내용도 따로 포스팅하겠습니다 <br/>

이번 프로젝트를 진행하면서 어떻게 구현해야하나 고민하면서 [Next.js의 github 코드 내부의 example 폴더](https://github.com/vercel/next.js/tree/canary/examples)를 정말 많이 읽었고 많은 부분에서 도움을 얻을 수 있었습니다. Next는 정말 공식문서도, example도 잘 되어있으니 꼭 참고하셨으면 좋겠습니다..넥스트 최고..사랑합니다 vercel...

<br/>

## Reference

[HTTP State Management using Cookies](https://medium.com/@kmtsandeepanie/http-state-management-using-cookies-ebab25d5c7e6)

[MDN Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)

[Next.js github with iron session example](https://github.com/vercel/next.js/tree/canary/examples/with-iron-session)

[Next.js Firebase authentication— Including SSR](https://medium.com/javascript-in-plain-english/next-js-firebase-authentication-including-ssr-1045b097ee18)

[next firebase ssr by colin](https://github.com/colinhacks/next-firebase-ssr)
