## 아이템 9. 타입 단언보다는 타입 선언을 사용하기

```ts
interface Person {
  name: string;
}

const alice: Person = { name: "Alice" }; // 타입은 Person
const bob = { name: "Bob" } as Person; // 타입은 Person
```

이 두 가지 방법은 결과가 같아 보이지만 그렇지 않다.

- 첫 번째 `alice: Person`은 변수에 **타입 선언**을 붙여서 그 값이 선언된 타입임을 명시
- 두 번째 `as Person`은 **타입 단언**을 수행

<br />

### 타입 단언보다 타입 선언을 사용하는 게 나은 이유

```ts
const alice: Person = {}; // 오류 발생, 'Person' 유형에 필요한 'name' 속성이 '{}'유형에 없습니다.
const bob = {} as Person; // 오류 없음
```

타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사한다. 첫 번째 예제에서는 그러지 못했기 때문에 타입스크립트가 오류를 표시하고, 타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것

- **타입 단언**이 꼭 필요한 경우가 아니라면, 안전성 체크도 되는 **타입 선언**을 사용하는 것이 좋다.

```ts
const people = ["alice", "bob", "jan"].map((name) => ({ name } as Person));
// 반환되는 값을 Person[]으로 단언, 그러나 좋지 않은 예제

const people = ["alice", "bob", "jan"].map((name) => ({} as Person)); // 런타임시에 오류 없음
```

최종적으로 원하는 타입을 직접 명시하고, 타입스크립트가 할당문의 유효성을 검사

```ts
const people: Person[] = ["alice", "bob", "jan"].map(
  (name): Person => ({ name })
);
```

- 함수 호출 체이닝이 연속되는 곳에서는 체이닝 시작에서부터 명명된 타입을 가져야 정확한 곳에 오류가 표시된다.

<br/>

### 타입 단언이 꼭 필요한 경우

- **타입 단언**은 타입 체커가 추론한 타입보다 개발자가 판단하는 타입이 더 정확할 때 의미가 있다.
- 예를 들어, 타입스크립트는 DOM에 접근할 수 없기 때문에 DOM 엘리먼트에 대해 타입 단언을 해주는 것

```ts
document.querySeletor("#myButton").addEventListener("click", (e) => {
  e.currentTarget; // 타입은 EventTarget
  const button = e.currentTarget as HTMLButtonElement;
  button; // 타입은 HTMLButtonElement
});
```

<br/>

### 자주 쓰이는 특별한 문법(!)을 사용해서 `null`이 아님을 단언

```ts
const eNull = document.getElementById("foo"); // 타입은 HTMLElement | null
const el = document.getElementById("foo")!; // 타입은 HTMLElement
```

- 접미사로 쓰이는 `!`는 그 값이 `null`이 아니라는 단언문으로 해석

---

> 최근 프로덕트 코드에서 서버로부터 받아오는 값을 잘 보여주기 위해 가공해야했는데 `Array.prototype.find`를 써서 원치않는 `undefined`가 발생하는 경우가 생겨서 **타입 단언**을 사용했다. 이에 대해 얘기를 나눴는데 `undefined`가 없을 거라고 단정짓지 않고 `if`문을 써서 해당 경우는 `throw New Error()`를 하는 게 낫다는 의견을 들었다. 서버를 믿지말고, 나를 믿지 않을 것!

<br/>
<br />

## 아이템 10. 객체 래퍼 타입 피하기

- 자바스크립트에는 **자동 형반환**이라는 속성이 있는데, 예를 들어 기본형 `string`과 객체 타입 `String`을 서로 자유롭게 변환한다. `string` 기본형에 `charAt` 같은 메서드를 사용할 때, 자바스크립트는 기본형을 `String` 객체로 래핑(wrap)하고 메서드를 호출하고, 마지막에 래핑한 객체를 버린다.

 <br/>

### 객체 래퍼 타입의 자동 변환

```js
> x = "hello"
> x.language = "English"
'English'

> x.language
undefined
```

이 코드는 실제로 `x`가 `String` 객체로 변환된 후 `language` 속성이 추가되었고, `language` 속성이 추가된 객체는 버려진 것

<br />

- 다른 기본형 `number, boolean, symbol, bigint`에도 객체 래퍼 타입이 존재 (`null`과 `undefined`에는 없음)
- 타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링

  - `string`과 `String`
  - `number`와 `Number`
  - `boolean`과 `Boolean`
  - `symbol`과 `Symbol`
  - `biginit`과 `Biginit`

<br />

### 타입스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용

대부분의 라이브러리와 마찬가지로 타입스크립트가 제공하는 타입 선언은 전부 **기본형 타입**으로 되어 있다.

<br />
<br />

## 아이템 11. 잉여 속성 체크의 한계 인지하기

_아이템 7. 타입이 값들의 집합이라고 생각하기_ 와 이어지는 내용

타입이 명시된 변수에 객체 리터럴을 할당할 때 타입스크립트는 해당 타입의 속성이 있는지, 그리고 **그 외의 속성은 없는지** 확인

```ts
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present", // 오류, 객체 리터럴은 알려진 속성만 지정할 수 있으며 'Room'형식에 'elephant' 속성이 없습니다.
};

const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
const r: Room = obj; // 정상
```

- `obj` 타입은 `Room` 타입의 부분 집합을 포함하므로, `Room`에 할당 가능하며 타입 체커도 통과

<br />

### 잉여 속성 체크 이용의 한계

- 객체 리터럴이 아닌 경우에는 **잉여 속성 체크**가 되지 않는다.

```ts
interface Options {
  title: string;
  darkMode?: boolean;
}

const o1: Options = document; // 정상

const o2: Options = {
  darkmode: true,
  title: "Ski Free",
}; // 에러, 'Options' 형식에  'darkmode'가 없습니다.
```

<br />

- 잉여 속성 체크는 타입 단언문을 사용할 때 적용되지 않는다.

```ts
const o = { darkmode: true, title: "Ski Free" } as Options; // 정상
```

```ts
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}

const opts = { logScale: true };
const o: LineChartOptions = opts;
// 에러, '{ logScale: true }' 유형에
// 'LineChartOptions' 유형과 공통적인 속성이 없습니다.
```

구조적인 관점에서 선택적 속성만 가지는 '약한(weak)' 타입에도 비슷하게 체크가 동작한다. 공통 속성 체크는 잉여 속성 체크와 마찬가지로 오타를 잡는데 효과적이며 구조적으로 엄격하지 않다. 하지만 잉여 속성 체크와 다르게, 약한 타입과 관련된 할당문마다 수행된다. 임시 변수를 제거하더라도 공통 속성 체크는 여전히 동작한다.

---

- 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행된다.
- 잉여 속성 체크에는 한계가 있다는 걸 인지하며 사용해야한다.
