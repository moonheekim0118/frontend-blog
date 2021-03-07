---
title: Emotion CSS prop 사용하기

date: 2021-03-07 15:50:50

category: Frontend

draft: false
---

이번에 프로젝트에서는 좀 색다르게 styled-component 대신 emotion을 사용하고 있습니다. 그런데 최근까지는 계속 emotion Styled를 사용하다보니 styled-component와의 큰 차이점을 별로 못느끼고 있었습니다. <br/>

그러던 도중 atom 컴포넌트로 만든 button 이나 Icon 과 같은 컴포넌트를 리팩토링하다가..props로 color, hoverColor, fontSize..등등 별의 별 스타일링 요소를 다 받게되는 아주 더러운 코드를 보고(...) 이를 간결화 할 방법을 물색하던 도중 Emotion의 CSS prop 기능을 발견하게 되었습니다. <br/>

그래서 CSS prop에 대해 간략하게 소개하고, 설정 및 사용방법을 포스팅해보고자 합니다. <br/>

## CSS Prop 이란

- Emotion에서 제공해주는 Element 스타일링 도구입니다.
- 컴포넌트에 Props 삽입하듯이, Element나 Styled Component에 css prop를 삽입해서 스타일링을 해줍니다.
- 기존에 styling이 적용된 컴포넌트에 삽입 시, 기존 styling을 override 합니다.

<br/>

## Emotion Styled와 CSS prop 뭘 써야하나 ?

- 둘 다 emotion에서 제공해주는 스타일링 도구이고, 사용성이 좋습니다만, 저는 이렇게 구분합니다. <br/>

#### Styled - 대부분의 컴포넌트 / 스타일링이 외부 요소에 의해 바뀔 일이 없는 컴포넌트

#### CSS prop - 스타일링이 사용되는 곳에 따라 바뀌어야 하는 컴포넌트 (atoms)

<br/>

## CSS prop 설정 및 사용법

```
npm i @emotion/react
npm i -D "@emotion/babel-plugin
```

1. `@emotion/react` 의 css 모듈을 통해 css prop를 작성합니다.
2. `@emotion/babel-plugin` 바벨 플러그인 설정을 위해 설치해줍니다.

<br/>

### .babelrc (리액트 without Next.js)

```javscript

{
  "presets": [
    [
      "@babel/preset-react",
      { "runtime": "automatic", "importSource": "@emotion/react" }
    ]
  ],
  "plugins": ["@emotion/babel-plugin"]
}

```

### .babelrc (Next.js)

```javascript
{
  "presets": [
    [
      "next/babel",
      {
        "preset-react": {
          "runtime": "automatic",
          "importSource": "@emotion/react"
        }
      }
    ]
  ],
  "plugins": ["@emotion/babel-plugin"]
}

```

- 위와 같이 바벨 설정을 해주어 css prop를 사용하도록 합니다.

<br/>

<br/>

### 사용하기

```typescript

import { css } from '@emotion/react'

const color = 'darkgreen'
//...
return(
  <div
    css={css`
      background-color: #fff;
	  font-size:1.5rem;`
    }
  >
    예시입니다.
  </div>
)

// .. 혹은

return(
  <div
    css={{
      Background-color: '#fff',
      fontSize:'1.5rem'
    }}
  >
    예시입니다.
  </div>
)
```

- 특정 컴포넌트에 CSS prop를 넘겨줄 때에는 **꼭 className도 함께 넘겨줘야 합니다** .그래야 CSS prop이 적용됩니다. 엘리먼트에 넘겨줄 때는 className을 넘겨주지 않아도 됩니다.
  <br/>
  <br/>

- CSS prop를 넘겨주는 방법은 객체 형식으로 넘겨주는 방식과 CSS 문자열로 넘겨주는 방식이 있습니다

### 예시

```typescript
import React, { memo, ButtonHTMLAttributes } from 'react'
import Link from 'next/Link'
import styled from '@emotion/styled'

// ...

const Button = (props: Props): React.ReactElement => {
  const {
    children,
    className,
    loading,
    disabled = false,
    onClick,
    ...rest
  } = props
  return (
    <Component
      className={className}
      onClick={onClick}
      disabled={disabled}
      {...rest}
    >
      {children}
    </Component>
  )
}

const Component = styled.button`
  font-size: 1.2rem;
  background-color: inherit;
  color: inherit;
  width: 100%;
  border: none;
  border-radius: 15px;
  padding: 13px 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  &:focus {
    outline: none;
  }
`

export default memo(Button)
```

```typescript
// ...

<Button className="reviewBtn" css={buttonStyle} onClick={...}>
    산책로 리뷰 보기
</Button>

const buttonStyle = css`
  background-color: #fff;
  color: ${colorCode['blue']};
  ${baseButtonStyle}

  &:hover {
    box-shadow: 0px 0px 5px 0px rgba(244, 244, 244, 0.75);
  }
`;
```

<br/>

- 위와 같은 버튼 컴포넌트를 여러 컴포넌트에서 사용할 때 CSS prop를 사용하기 딱 좋습니다. 버튼 스타일링이 미세하게 계속 바뀔 수 있기때문입니다.
  <br/>
  <br/>

- 버튼 컴포넌트를 사용할 때 간단히 CSS prop와 ClassName만 넣어주면 스타일링이 override 됩니다.

<br/>

<br/>

### 마치며

이렇게 간단하게 emotion의 기능인 CSS props에 대해 알아보았습니다. <br/>

styled와 CSS props를 적재적소에 잘 사용하면 매우 깔끔하게 스타일링을 구현할 수 있습니다

<br/>

## Reference

[Emotion official Docs - CSS prop](https://emotion.sh/docs/css-prop)
