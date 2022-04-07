## 아이템 6. 편집기를 사용하여 타입 시스템 탐색하기

- 타입스크립트를 설치하면, 다음 두 가지를 실행할 수 있다.
  - 타입스크립트 컴파일러(tsc) → 주된 목적
  - 단독으로 실행할 수 있는 타입스크립트 서버(tsserver) → 언어 서비스를 제공한다는 점에서 중요
- 언어 서비스란
  - 코드 자동 완성, 명세(사양, specification) 검사, 검색, 리팩터링 포함
- 타입 추론 살펴보기
  - 반환값의 타입을 명시하지 않으면 반환값의 타입을 어떻게 추론하는지 살펴보기
  - 조건문의 분기에서 값의 타입이 어떻게 변하는지 살펴보기
  - 객체의 개별 속성의 타입을 추론하는지 살펴보기
  - 메서드 체인 중간의 추론된 제너릭 타입 살펴보기
  - 타입 오류 살펴보기
    - ex) typeof null이 “object”라 생기는 오류
    - cf) `document.getElementById()` 의 경우 HTMLElement 뿐만 아니라 null도 올 수 있다.
  - 라이브러리 타입 선언 살펴보기

<br />

## 아이템 7. 타입이 값들의 집합이라고 생각하기

- 타입: 할당 가능한 값들의 집합
- null, undefined는 strictNullChecks 여부에 따라 number에 해당될 수도, 아닐 수도 있다.
- 집합의 크기
  - `naver` < `literal(unit)` < `union`
- naver: 아무 값도 포함하지 않는 공집합. 아무 값도 할당할 수 없다.
- literal(unit): 한 가지 값만 포함하는 타입 (ex) type A = ‘A’;
- union: 값 집합들의 합집합. 2개 이상 (ex) type AB = ‘A’ | ‘B’
- 타입스크립트 오류 중 ‘assignable’의 의미
  - (ex) `Typescript: Type 'string | undefined' is not assignable to type 'string'`
  - 집합의 관점에서 ‘~의 원소(값과 타입의 관계)’ 또는 ‘~의 부분 집합(두 타입의 관계)’을 의미한다.
  - 타입 체커의 주요 역할은 하나의 집합이 다른 집합의 부분 집합인지 검사하는 것
- `&` 연산자는 두 타입의 인터센션(교집합)을 계산한다.

  - 타입 연산자는 인터페이스의 속성이 아닌 값의 집합(타입의 범위)에 적용된다.
  - 인터섹션 타입의 값은 각 타입 내의 속성을 모두 포함하는 것이 일반적인 규칙이다. 하지만 두 인터페이스의 유니온에서는 그렇지 않다.
  - 속성에 대한 인터섹션과 인터페이스의 유니온의 차이 이해하기

    - 속성에 대한 인터섹션
      두 집합에서 필요로 하는 요건을 모두 만족시켜야 한다. 두 집합에서 가지고 있는 속성보다 더 많은 속성을 가지는 값도 타입에 속한다. (← 이 말 무슨 뜻인지..?)

      ```tsx
      interface Person {
        name: string
      }

      interface Lifespan {
        birth: Date
        death?: Date
      }

      type PersonSpan = Person & Lifespan

      // Type '{ name: string; }' is not assignable to type 'PersonSpan'. Property 'birth' is missing in type '{ name: string; }' but required in type 'Lifespan'
      const me1: PersonSpan = {
        name: '라도',
      }

      // 정상
      const me2: PersonSpan = {
        name: '라도',
        birth: new Date('22/4/7'),
      }

      // Type '{ name: string; birth: Date; team: string; }' is not assignable to type 'PersonSpan'. Object literal may only specify known properties, and 'team' does not exist in type 'PersonSpan'.
      const me3: PersonSpan = {
        name: '라도',
        birth: new Date('22/4/7'),
        team: 'dev',
      }
      ```

    - 인터페이스의 유니온

      ```tsx
      type K = keyof (Person | Lifespan) // type K = never
      type K2 = keyof (Person & Lifespan) // type K2 = "name" | keyof Lifespan

      // 정상
      const keyOfMe: K2 = "name"

      keyof (A|B) = (keyof A) & (keyof B)
      keyof (A&B) = (keyof A) | (keyof B)
      ```

  - `extends` 를 사용해 서브 타입 만들기
    - 제너릭 타입에서 한정자로 쓰임
  - ‘A는 B를 상속’, ‘A는 B에 할당 가능’, ‘A는 B의 서브타입’은 ‘A는 B의 부분 집합’과 같은 의미이다.
  - 타입스크립트를 집합으로 이해하기
    | 타입스크립트 용어 | 집합 용어 |
    | ---------------------------- | ----------------------------- | -------------------------- |
    | never | ∅(공집합) |
    | 리터럴 타입 | 원소가 1개인 집합 |
    | 값이 T에 할당 가능 | 값 ∈ T (값이 T의 원소) |
    | T1이 T2에 할당 가능 | T1 ⊆ T2 (T1이 T2의 부분 집합) |
    | T1이 T2를 상속 | T1 ⊆ T2 (T1이 T2의 부분 집합) |
    | T1 | T2 (T1과 T2의 유니온) | T1 ∪ T2 (T1과 T2의 합집합) |
    | T1 & T2 (T1와 T2의 인터섹션) | T1 ∩ T2 (T1과 T2의 교집합) |
    | unknown | 전체(universal) 집합 |

<br />

## 아이템 8. 타입 공간과 값 공간의 심벌 구분하기

- 타입스크립트의 심벌은 타입 공간이나 값 공간 중의 한 곳에 존재한다. 이름이 같더라도 어떨 땐 타입으로 쓰이고, 어떨 땐 값으로 쓰일 수 있다.

```tsx
interface Square {
  width: number
  height: number
}

const Square = (width: number, height: number) => ({ width, height })

function calculateArea(shape: unknown) {
  if (shape instanceof Square) {
    shape.width // Property 'width' does not exist on type '{}'
  }
}
```

- instanceof는 자바스크립트의 런타임 연산자이고, 값에 대해서 연산을 한다. 그래서 `instanceof Square`는 타입이 아니라 함수를 참조한다.
- type, interface, 타입 선언(:), 단언문(as) 다음에 나오는 심벌은 타입
- const나 let 선언에 쓰이는 것, `=` 다음에 나오는 모든 것은 값
- class와 enum은 타입과 값 두 가지 모두 가능한 예약어

  - class가 타입으로 쓰일 때는 형태(속성과 메서드)가 사용되는 반면, 값으로 쓰일 때는 생성자가 사용된다.

  ```tsx
  class Square {
    width = 1
    height = 1
  }

  function calculateArea(shape: unknown) {
    if (shape instanceof Square) {
      // 값으로 쓰임
      shape // 정상. 타입은 Square
      shape.width // 정상. 타입은 number
    }
  }
  ```

- 연산자 중 타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 하는 것
  - `typeof`
    - 타입의 관점에서 typeof는 값을 읽어서 타입스크립트 타입을 반환한다. 타입 공간의 typeof는 보다 큰 타입의 일부분으로 사용할 수 있고, type 구문으로 이름을 붙이는 용도로도 사용할 수 있다.
    - 값의 관점에서 typeof는 자바스크립트 런타임의 typeof 연산자가 된다. 값 공간의 typeof는 대상 심벌의 런타임 타입을 가리키는 문자열을 반환하며 타입스크립트의 타입과는 다르다.
    - 클래스는 타입과 값 두 가지로 모두 사용되기 때문에 상황에 따라 다르게 동작
      ```tsx
      const v = typeof Square // 값이 "function" -> 클래스가 자바스크립트에서는 실제 함수로 구현되기 때문
      type T = typeof Square // 타입이 typeof Square
      ```
- 타입의 속성을 얻을 때는 반드시 `obj['field']` 를 사용해야 한다.
- 값으로 쓰이는 this는 자바스크립트의 this 키워드. 타입으로 쓰이는 this는, 일명 ‘다형성 this’라고 불리는 this의 타입스크립트 타입.
- 값에서 &와 |는 AND와 OR 비트 연산. 타입에서는 인터섹션과 유니온
- const는 새 변수를 선언하지만, as const는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꾼다.
- extends는 서브클래스(`class A extends B`) 또는 서브타입(`interface A extends B`) 또는 제너럴 타입의 한정자(`Generic<T extends number>`)를 정의할 수 있다.
- in은 루프(`for (key in obj)`) 또는 매핑된 타입에 등장한다.
- 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다.
- “foo”는 문자열 리터럴이거나, 문자열 리터럴 타입일 수 있다.
