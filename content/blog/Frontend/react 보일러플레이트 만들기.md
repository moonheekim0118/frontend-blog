\---

title: CRA 없이 리액트 bolierplate 만들기

date: 2020-09-10 22:40:50

category: Frontend

draft: false

\---

## Category

1. [포스팅의 목적 - Goal](#goal)
2. [왜 CRA를 납두고? - Create React App은 편리하다, 하지만..](#intro)
3. [초기 환경 세팅](#first)
4. [Babel 세팅](#babel)
5. [Webpack 세팅](#webpack)
6. [ESLint 세팅](#eslint)
7. [최종 파일 구조](#final)
8. [리퍼런스](#refer)

<br/>

<br/>

## Goal <a name="goal"/>

- Webpack의 기능과 사용법을 안다.
- Babel의 기능과 사용법을 안다.
- EsLint의 기능과 사용법을 안다.
- Create React App 없이 나만의 리액트 bolierplate를 만들어본다.

<br/>

## Create React App은 편리하다,하지만.. <a name="intro"/>

​ cra로 리액트 프로젝트를 시작하는 것은 정말 편하다. 리액트가 알아서 복잡한 바벨과 웹팩 환경을 세팅해주므로 개발자는 리액트 코드만 작성하면 알아서 자동 빌드해주고 서버까지 띄워주기 때문이다. <br/>

​ 하지만 eject를 해서 웹팩 설정을 수정해야하는 순간 cra가 얼마나 많은 것들을 감추고 있었는지 깨닫게 되고, 내가 '다 차려진 밥상 위에 숟가락만 얹히고' 있었다는 것을 깨닫게 된다. <br/>

​ 따라서 이 포스팅을 통해서 직접 리액트 프로젝트 세팅을 해봄으로써 바벨과 웹팩, 그리고 esLint의 사용법을 간단하게 정리하여 리액트에 대한 이해를 더 넓혀보고자 한다. <br/>

<br/>

<br/>

## 초기 환경 세팅 <a name="first"/>

1. 먼저 npm init을 이용하여 package.json 파일을 생성한다.
   - package.json 파일은 현재 Node 프로젝트에 대한 정보를 담고 있다.

```javascript
npm init
```

<br/>

2. 리액트를 사용하기 위해 react와 react-dom 모듈과, react-hot-loader를 설치해준다.

   - hot loader는 코드에 변경사항이 생겼을 때, 새로고침 하지 않고도 변경된 부분만 교체해주는 라이브러리이다.

   - hot loader는 배포시에는 사용되지 않으므로 개발용 (-D) 으로 설치해준다.
   - hot loader 는 웹팩 설정시에도 사용된다.

```
npm i react react-dom
npm i -D react-hot-loader
```

<br/>

3. 아래와 같이 폴더 구조를 잡아준다.

```
.
+-- public
| +--index.html
+-- src
| +--index.js
| +--App.js
| +--App.css
```

- public 폴더에는 모든 static assets을 넣을 것이다.

  - index.html 파일은 바로 리액트가 렌더링 하기 위해 최초로 로딩할 html 파일이다.

- src 폴더에는 컴포넌트들과, 실제로 DOM에 렌더링 해주는 index.js 파일이 들어간다.

 <br/>

4. public / index.html 파일을 아래와 같이 작성해준다.
   - script src를 이용해 번들링된 js 파일과 연결시켜준다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Bolierplate</title>
  </head>
  <body>
    <div id="root"></div>
    <script src="../dist/bundle.js"></script>
    <!--번들링된 js 파일과 연결해준다-->
  </body>
</html>
```

<br/>

5. src / index.js 파일을 아래와 같이 작성해주어 App 컴포넌트를 index.html의 id == root인 DOM에다 렌더링 해준다.

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App.js'
ReactDOM.render(<App />, document.getElementById('root'))
```

<br/>

6. src / App.js 파일을 아래와 같이 작성해준다.
   - 프로젝트의 최상위 컴포넌트 App 에 hot을 불러온 후 내보낼 때 hot(module)(App) 형식으로 작성하여 hot loader를 적용해준다.

```jsx
import React, { Component } from 'react'
import { hot } from 'react-hot-loader'
import './App.css'

class App extends Component {
  render() {
    return (
      <div className="App">
        <h1>Hello World!</h1>
      </div>
    )
  }
}

export default hot(module)(App)
```

<br/>

<br/>

## Babel 세팅 <a name="babel"/>

- 바벨은 브라우저가 읽지 못하는 최신 자바스크립트 문법 및 JSX 코드를 ES5 이하의 자바스크립트 코드로 변환시켜주는 컴파일러이다.

<br />

1. 바벨을 패키지 및 모듈을 설치한다.
   - babel/core는 바벨 메인 패키지이다.
   - preset은 plugin의 모음을 뜻한다.
   - @babel/preset-env는 최신 자바스크립트 문법을 특정 브라우저(따로 target을 지정하지 않을 경우 모든 브라우저를 디폴트로 한다)에서 호환가능하도록 설정해주는 옵션이다.
   - @babel/preset-react는 리액트(JSX)를 지원해주도록 설정해주는 옵션이다.

```
npm i -D @babel/core @babel/preset-env @babel/preset-react
```

<br />

2. 프로젝트 최상위 루트에 .babelrc 파일을 생성해주고 아래와 같이 작성해줌으로써, env와 react preset을 사용할 것이라고 설정해준다.

```javascript
{
    "presets": ["@babel/env", "@babel/preset-react"]
}
```

###

## Webpack 세팅 <a name="webpack"/>

자 이렇게 바벨까지 다 세팅해주었다면 웹팩을 설치 및 설정하여 js 코드를 빌드 해주어야 한다. <br/>

- 웹팩은 여러개의 js 파일을 하나의 js 파일로 묶어주는 모듈 번들러이다.
- 리액트는 첫 로딩시 , 번들된 js 파일이 연결된 html 파일을 로드하므로 웹팩이 필요하다.

<br/>

1. 웹팩 설정을 위해 필요한 모듈을 설치해준다.
   - webpack-cli 는 빌드 할 때 webpack 관련된 CLI (커맨드라인 인터페이스)을 제공해준다.
   - webpack-dev-server는 소스파일이 변경될 시, 변경된 모듈만 새로 번들링하여 서버로 띄워주는 역할을 한다.

```
npm i -D webpack webpack-cli webpack-dev-server
```

<br />

2. 웹팩에서 사용할 loader 모듈들을 설치해준다.
   - 웹팩은 loader를 이용하여 각각 다른 파일의 번들을 처리한다. 예를들면 CSS파일을 번들링하고 싶다면 그에 맞는 로더를 설치해야 한다.
   - babel loader는 일부 브라우저에서 지원하지 않는 JSX 코드와 ES6 이상의 문법을 ES5 이하 버전의 JS 코드로 변환 시켜준다.
     - babel loader 옵션의 presets는 아까 바벨 설정 시 다운받은 preset을 적용한다.

```
npm i -D style-loader css-loader babel-loader
```

<br />

3. 최상위 루트에 webpack.config.js 파일을 생성해주고 아래와 같이 작성해준다.

```javascript
const path = require('path')
const webpack = require('webpack')

module.exports = {
  mode: 'development', // 배포시에는 production으로 변경해야한다.
  devtool: 'eval',
  resolve: {
    // 웹팩이 알아서 경로 / 확장자를 처리할 수 있게 도와주는 옵션
    extensions: ['.js', '.jsx'],
  },
  entry: './src/index.js', // 웹팩이 읽어들일 파일
  module: {
    // js 모듈의 변환을 rules에 맞게 처리해준다.
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /(node_modules|bower_components)/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env', '@babel/preset-react'],
          plugins: 'react-hot-loader/babel',
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  output: {
    // 번들링된 js 파일이 저장될 위치와 파일명
    path: path.join(__dirname, 'dist/'),
    publicPath: '/dist/',
    filename: 'bundle.js',
  },
  devServer: {
    contentBase: path.join(__dirname, 'public/'),
    port: 3000,
    publicPath: 'http://localhost:3000/dist/',
    hotOnly: true,
  },
  plugins: [new webpack.HotModuleReplacementPlugin()],
}
```

- mode
  - 배포시에는 production 으로 변경해줘야 한다.
  - mode에 따라 번들링의 결과물이 달라진다.
- resolve
  - 웹팩이 알아서 경로 / 확장자명을 처리할 수 있게 해준다.
  - 따라서 모듈 import할 때 확장자명을 생략 가능 (css 제외)
- entry
  - 웹팩이 읽어들일 파일의 경로
- module
  - 프로젝트 내의 여러 타입의 모듈을 정의 / 처리 해주기 위한 규칙(rules)를 설정하는 옵션
- rules 에서 test와 exclude 옵션
  - exclude 옵션 바깥에 존재하는 모든 (test에 정의된) 확장자명의 문법을 정의된 로더를 사용하여 변환하겠다는 설정
- 로더가 두개 이상일 경우 use 이용하여 두 로더를 묶어준다.
- output
  - 번들된 파일이 저장될 경로와 파일명을 정의해준다.
- output의 publicPath
  - 번들된 파일이 저장될 경로 정의
  - webpack-dev-server가 번들된 파일을 가져올 경로 정의
- devServer
  - webpack-dev-server에 대한 설정 - 포트번호/ 경로
- contentBase
  - public/ 에 있는 모든 파일을 서버에 띄워준다
- devServer의 publicPath
  - 서버에게 번들된 파일이 어디있는지 알려준다.
- 핫 로더 적용
  - devServer에서 hotOnly:true 설정과 plugins: [new webpack.HotModuleReplacementPlugin()] 를 이용해 핫 모듈 플러그인의 인스턴스를 생성해줌으로써 핫 로더를 적용해준다.

<br/>

<br/>

## ESLint 세팅 <a name="eslint"/>

- 코드 Linting이란 특정 스타일 규칙을 준수하지 않는 소스코드를 찾는데 사용되는 방식이고, Linter는 이러한 Linting을 수행하는 도구이다.
- 자바스크립트는 컴파일 과정이 없기때문에, Linter가 내장되어 있지 않다. 따라서 이러한 자바스크립트를 실행하지 않고도 구문 오류등을 찾아내주는 것이 ESLint이다.

<br/>

1. ESLint와 ESLint react 플러그인을 설치해준다.

```javascript
npm i -D eslint eslint-plugin-react
```

<br />

2. 프로젝트의 최상위 루트에 .eslintrc 라는 ESLint 설정 파일을 생성해주고 아래와 같이 설정해준다.

   - 일단 필수로 설정해주어야 할 것은 plugin:react/recommended와 sourceType: module이다. 이를 설정하지 않으면 import 문법과 const 문법을 에러 처리한다.

   - 다른 여러 설정은 앱 확장과 더불어 [유저가이드](https://eslint.org/docs/user-guide/configuring ) 를 따라서 설정해주면 된다.

```javascript
{
     "env": {
         "browser": true,
         "node":true,
         "es6": true
    },
    "plugins": ["react"],
    "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
    ],
    "parserOptions": {
        "ecmaVersion": 7,
        "sourceType": "module",
        "ecmaFeatures": {
            "jsx": true
        }
    }
}

```

<br/>

<br/>

## 최종 파일 구조 <a name="final"/>

```
.
+-- dist
| +-- bundle.js
+-- public
| +-- index.html
+-- src
| +-- App.css
| +-- App.js
| +-- index.js
+-- .babelrc
+-- .eslintrc
+-- .gitignore
+-- package.json
+-- package-lock.json
+-- webpack.config.js
```

이렇게 웹팩과 바벨 그리고 ESLint를 사용하여 나만의 리액트 bolierpalte를 만들어보았다. 만들면서 든 생각은 웹팩 설정이 참 까다롭긴해도 , 이렇게 한번 알아두면 나중에 CRA에서 eject로 수정할 때에도, 혹은 커스텀하여 리액트 프로젝트 세팅을 할 때에도 편하다는 것이다.

## Reference <a name="refer"/>

- [Creating a React App...From Scratch By Jedai Saboteur](https://blog.usejournal.com/creating-a-react-app-from-scratch-f3c693b84658)

- [Webpack official docs](https://webpack.js.org/)

- [Babel official docs](https://babeljs.io/docs/en/)

- [ESLint UserGuide](https://eslint.org/docs/user-guide/configuring)

- [How to Setup ESLint and Prettier for Your React Apps](https://thomlom.dev/setup-eslint-prettier-react/)

- [Setting up ESLint in React By Ross Whitehouse](https://medium.com/@RossWhitehouse/setting-up-eslint-in-react-c20015ef35f7)

- [리액트 프로젝트 초기 셋팅(NO CRA!) By yhe228](https://velog.io/@yhe228/%EB%A6%AC%EC%95%A1%ED%8A%B8-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%B4%88%EA%B8%B0%EC%85%8B%ED%8C%85NO-CRA)

- [React 개발 환경을 구축하면서 배우는 Webpack 기초 By jeff0720](https://velog.io/@jeff0720/React-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD%EC%9D%84-%EA%B5%AC%EC%B6%95%ED%95%98%EB%A9%B4%EC%84%9C-%EB%B0%B0%EC%9A%B0%EB%8A%94-Webpack-%EA%B8%B0%EC%B4%88)

<br/>
<br/>
