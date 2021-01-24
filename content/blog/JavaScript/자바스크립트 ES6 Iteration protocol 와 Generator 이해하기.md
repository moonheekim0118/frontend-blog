---
title: 자바스크립트 ES6 Iteration protocol 와 Generator 이해하기

date: 2021-01-24 19:00:50

category: JavaScript

draft: false
---

## 왜 필요한가

이미 자바스크립트에는 for 루프 부터 map, forEach 와 같은 반복문을 만들어주는 컬렉션이 있다. 그렇다면 iterator와 iterable 은뭘까 ?

- **iterator는 프로그래머가 반복문의 결과를 정의할 수 있도록 도와준다.**
- **iterable은 @@iterator를 가지고 있는 객체를 말한다.**

<br/>
그러면 자바스크립트 es6에 포함된 iterator protocol과 iterable protocol 그리고 Generator 에 대해 자세히 뜯어보자 !

<br/>

## 1. Iterator Protocol

- 객체를 next 메서드로 순환 (iterate) 할 수 있는 객체이다.
- value와 done을 프로퍼티로 가진 객체를 반환한다. `{value, done}`
  - 프로퍼티 value : 순환하는 객체에서 리턴되는 value
  - 프로퍼티 done : 순환이 끝났는지 아닌지에 대한 bool값
  - 순환이 끝나고 next 메서드를 실행한다면 ? `{value : undefined ,done: true}`객체를 반환 받는다.

<br/>

**예시 1 )** 문자열에 iterator 프로퍼티를 추가하여 iterator 실행해보기

```javascript
let iterator = '예시'[Symbol.iterator]()
iterator.next() // { value:'예', done: false}
iterator.next() // { value:'시', done: false}
iterator.next() // { value:undefined , done: true}
```

<br/>

**예시 2)** iterator 객체는 아래와 같이 직접 구현 될 수도 있다.

```javascript
let counter = 0
let limit = 3

const iteratorObjectProtocol = {
  next: function() {
    counter++
    if (counter >= limit) return { value: undefined, done: true }
    return { value: counter, done: false }
  },
}

console.log(iteratorObjectProtocol.next())
// { value: 1, done: false}
console.log(iteratorObjectProtocol.next())
// { value: 2, done: false}
console.log(iteratorObjectProtocol.next())
// { value: undefined, done: true}
```

<br/>

## 2. Iterable Protocol

- 객체의 멤버를 순환 할 수 있는 객체이다.
- `Array`와 `Map`과 같은 타입은 iterable 이 구현되어 있는 `built-in iterables`이다.
- `for...of` 를 통해 iterator 값들을 순환 할 수 있고, 위에서 보았듯이 `펼침 연산자 (spread operator)`로 표현 시, 배열형태로 반환 접근 할 수 있다.
- built-in-iterables가 아닌 객체를 iterable 하기 위해서는 object에 `@@iterator` 메소드를 구현해야 한다.

```javascript
object[Symbol.iterator] : iterator 메서드 정의
```

<br/>

> Built in iterables 종류
>
> Array , TypedArray, String, Map, Set

**예시 1)** 위에서 구현한 `iteratorObjectProtocol` 메서드를 `[Symbol.iterator]` 에 정의해보도록 하겠다.

```javascript
let counter = 0
let limit = 3

const iteratorObjectProtocol = {
  next: function() {
    counter++
    if (counter >= limit) return { value: undefined, done: true }
    return { value: counter, done: false }
  },
}

const obj = {
  [Symbol.iterator]: function() {
    return iteratorObjectProtocol
  },
}

console.log([...obj])
// [1,2]
```

- 객체 obj를 펼친 배열 (spread array) 형태로 선언하면 `Symbole.iterator` 프로퍼티에 정의한 `iteratorObjectProtocol`이 실행된 결과물이 배열의 형태로 담겨져 있는 것을 알 수 있다.
- 그렇다면 매번 복잡한 iterator 객체와 내부의 next 메서드를 구현해야할까? 이를 간결하게 만들어주는 것이 바로 ES6의 **Generator 함수**이다.

<br/>

**예시 2 )** Generator 함수를 통해서 간결화하기

```javascript
const obj = {
  [Symbol.iterator]: function*() {
    yield 1
    yield 2
  },
}

for (let value of obj) {
  console.log(value)
}
// 1, 2
```

<br/>

<br/>

## 3. Generator

위의 예시에서 사용한 Generator 에 대해서 더 자세히 알아보도록 하겠다.

<br/>

### Generator 함수

- `Iterable`을 생성하는 함수로, 위에서 우리가 직접 제어문으로 구현한 Iterator 함수를 간결화 시킨 것이라고 보면 된다.
- 제너레이터 함수는 일반 함수와 달리 사용자의 정의에 따라서 함수 코드블록을 일시 중지했다가, 필요한 시점에 재개 할 수 있다.
- 일반 함수 호출시 return으로 값을 반환하지만, 제너레이터 함수는 `제너레이터를 반환`한다.
  - 제너레이터는 `iterable`이면서 동시에 `iterator`인 객체이다.
  - 즉, **@@iterator**를 소유한 iterable이며, 동시에 **next 메서드**를 소유하여, next 메서드 호출 시 { value, done:bool } 객체를 반환한다.

<br/>

**예시 1)** 제너레이터 함수로부터 반환되는 제너레이터

```javascript
function* genFunc() {
  for (let value of [1, 2]) yield value
}

const generatorObj1 = genFunc()

// Iterable 이므로 for..of를 통해서 값 반환
for (let value of generatorObj) {
  console.log(value) // 1 , 2
}

// next 메서드를 가지고 있는 iterator
const generatorObj2 = getFunc()
console.log(generatorObj2.next())
// {value:1, done:false}
console.log(generatorObj2.next())
// {value:2, done:false}
console.log(generatorObj2.next())
// {value:undefined, done:true}
```

<br/>
<br/>

---

이렇게 JavaScript의 ES6의 iterable protocol과 Generator를 알아보았다. <br/>
사실 iterable protocol은 자주 사용할 일은 없지만, 리덕스 사가를 사용한다면 Generator 함수 사용이 불가피하다. <br/>
이전에는 단순히 Generator 함수를 함수의 순서를 제어해주는 함수라고만 생각했는데, iterable protocol과 함께 보면 Generator 함수가 어떻게 동작하고,
반환되는 Generator의 속성에 대해 더 깊이 알 수 있다 <br/>

<br/>

### Reference

[Javascript와 Iterator](https://pks2974.medium.com/javascript%EC%99%80-iterator-cdee90b11c0f)

[Javascript: Iterator and iterable protocols](https://medium.com/@insomniocode/javascript-iterator-and-iterable-protocols-583b700305ce)

[MDN Iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)

[제너레이터와 async/awit](https://poiemaweb.com/es6-generator)
