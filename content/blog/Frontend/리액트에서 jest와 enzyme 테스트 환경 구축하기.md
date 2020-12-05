---
title: 리액트에서 jest와 enzyme 테스트 환경 구축하기 

date: 2020-12-05 10:00:50

category: frontend

draft: false
---

## Goal

- 테스팅 환경을 구축하고 기본적인 테스트코드를 작성해본다.


<br/>




이번에 영화 리뷰 플랫폼 프로젝트를 진행하면서 처음으로 프론트엔드 테스팅을 경험해보았습니다. 환경구축부터 테스트코드를 작성하는 것 까지 모두 처음이었기 때문에 우왕좌왕 했습니다. 그 중에서도 환경 구축하는게 꽤나 까다로웠기때문에 미래의 저를 위해 포스팅을 남기고자 합니다. 

참고로 **타입스크립트 , 리액트 17** 입니다 . 

<br/>

#### jest와 enzyme 설치하기

- jest와 enzyme 그리고 , enzyme adapter 를 설치합니다. 여기서 문제 되는 점은 enzyme가 아직 리액트 17을 지원하지 않는다는 점입니다. 따라서 이를 지원해주는 개인 라이브러리를 사용했습니다.  이는 [해당 깃헙 이슈](https://github.com/enzymejs/enzyme/issues/2429) 에서 힌트를 얻었고 [이 라이브러리](https://www.npmjs.com/package/@wojtekmaj/enzyme-adapter-react-17)를 사용했습니다. 

```
npm i -D jest enzyme @wojtekmaj/enzyme-adapter-react-17
```

<br/>



#### types 를 설치하기

- 타입스크립트 프로젝트이므로 types를 설치해주지 않으면 에디터에서 에러가 납니다.

```
npm i -D @types/jest @types/enzyme  
```

<br/>

#### setUpTest 파일 작성

- 먼저 저는 src 폴더 하위에  __ test __ 라는 폴더를 생성한후 setupTests.ts 파일을 생성하고 아래와 같이 작성 했습니다. 이를 셋업파일로 작성하지 않을 경우 모든 테스트 파일마다 Adapter를 실행해야 합니다. 

```javascript
import { configure } from 'enzyme';
import Adapter from '@wojtekmaj/enzyme-adapter-react-17';

configure({ adapter: new Adapter() });
```

<br/>



#### jest config 파일 작성

- jest.config.js 파일을 생성한 후 아래와 같이 작성해줍니다. setupFilesAfterEnv 항목에는 위에 작성한 setupTests.ts 의 경로를 적어 주세요. <rootDir>를 앞에 붙이고 root를 기준으로 작성하면 됩니다. 

```javascript
module.exports = {
    preset: 'ts-jest',
    transform: {
      '^.+\\.tsx?$': 'babel-jest',
    },
	testEnvironment: "node",
	testMatch: ["**/__tests__/**/*.ts?(x)", "**/?(*.)+(test).ts?(x)"],
  transformIgnorePatterns: ["/node_modules/", "/.next/"],
  setupFilesAfterEnv: ["<rootDir>/src/__test__/setupTests.ts"] 
};
```



<br/>



#### 테스트코드 작성해보기

- 이제 환경 셋팅이 끝났으니 시험삼아 첫 테스트를 해보겠습니다.  주의할점은 `import 'jsdom-global/register'`  를 맨위에 작성해주지 않으면 에러가 뜹니다. 

```typescript
import 'jsdom-global/register'; 
import React from "react";
import { mount } from 'enzyme';
import Component from '../components' // 테스트 하고자하는 컴포넌트를 가져와주세요 

describe('<Component/>', () => {
   let container = null;
   const mockFn = jest.fn(); // 가짜 함수를 만들어줍니다. props로 onClick이 들어가는 경우에 이를 삽입해주면 됩니다. 

   it('renders correctly', ()=>{ // 컴포넌트가 올바르게 렌더링되는지 테스트 해줍니다. 
      container=mount(<Component onClick={mockFn}/>); 
   });

	// 맨 처음 작성된 스냅샷으로 비교해줍니다. 처음 테스트를 실행할 경우 스냅샷 폴더가 생성되								고 새로운 파일이 저장됩니다.`
   it('matches snapshot', ()=>{
    expect(container.html()).toMatchSnapshot();
   });

   it('should call onClick if button is clicked',()=>{ // onClick 버튼 시뮬레이션 
      container.simulate('click');
      expect(mockFn).toHaveBeenCalled(); // 불러와졌는지 체크 합니다. 
   });

   
})
```



<br/>



#### 만약 타입스크립트 문법 에러가 뜬다면 ?

- 위와같이 테스트코드를 작성하는데 타입스크립트 문법관련 에러가 뜬다면 `tsconfig.json` 파일의 컴파일러 옵션을 수정해주세요. 저의 tsconfig.json은 아래와 같습니다.

```javascript
{
  "compileOnSave": false,
  "compilerOptions": {
    "jsx": "preserve",
    "allowJs": true,
    "lib": ["ES2019","dom"],
    "module": "commonjs",
    "target": "ES2019",
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "skipLibCheck": true,
    "strict": false,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "isolatedModules": true
  },
  "exclude": [
    "node_modules"
  ],
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx"
  ]
}
```



<br/>



---

이렇게 jest, enzyme 환경 구축을 해보았습니다.  첫 테스트코드 작성 및 환경 구축이라 어렵고 미숙하기도 했지만, 그만큼 뿌듯했고, 왜 테스트코드를 작성하는지 알 수 있던 시간이었습니다. 테스트 코드를 작성하면서 코드에서 놓쳤던 에러와, 코드의 미숙한 부분들을 다시한번 되짚어 볼 수 있었기 때문입니다. 다음번에는 TDD를 도입하여 더 견고한 소프트웨어를 만들어 보고 싶습니다 ^^



 