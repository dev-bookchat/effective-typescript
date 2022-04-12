# 이펙티브 타입스크립트 아이템 9~11

## 아이템 9. 타입 단언보다는 타입 선언을 사용하기

```tsx
const alice: Person = { name: 'Alice' } // 타입 선언
const bob = { name: 'Bob' } as Person // 타입 단언
```

- 타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사한다. 반면, 타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것이다.
- 타입 단언이 꼭 필요한 경우가 아니라면, 안전성 체크도 되는 타입 선언을 사용하는 것이 좋다.
- 타입 단언이 꼭 필요한 경우
  - (ex) DOM 엘리먼트
    ```tsx
    document.querySelector('#myButton').addEventListener('click', (e) => {
      e.currentTarget // 타입은 EventTarget
      const button = e.currentTarget as HTMLButtonElement
      button // 타입은 HTMLButtonElement
    })
    ```
    - 타입스크립트는 DOM에 접근할 수 없기 때문에 #myButton이 버튼 엘리먼트인지 알지 못한다. 우리는 타입스크립트가 알지 못하는 정보를 가지고 있기 때문에 여기서는 타입 단언문을 쓰는 것이 타당하다.
- 접미사로 쓰이는 `!` 는 null이 아니라는 단언문으로 해석
- 모든 타입은 unknown의 서브타입이기 때문에 unknown이 포함된 단언문은 항상 동작한다. 하지만 unknown을 사용한 이상 적어도 무언가 위험한 동작을 하고 있다는 걸 알 수 있다.
  ```tsx
  const el = document.body as unknown as Person
  ```
- 화살표 함수의 반환 타입 선언하기
  ```tsx
  const people: Person[] = ['alice', 'bob', 'jan'].map((name): Person => ({ name }))
  ```

<br />

## 아이템 10. 객체 래퍼 타입 피하기

- 자바스크립트에는 메서드를 가지는 String ‘객체’ 타입이 정의되어 있다. string ‘기본형’에는 메서드가 없지만, string 기본형에 charAt 같은 메서드를 사용할 때, 자바스크립트는 기본형을 String 객체로 래핑하고 메서드를 호출하고 마지막에 래핑한 객체를 버린다.
  ```tsx
  > x = "hello"
  > x.language = 'English'
  'English' // x가 String 객체로 변환된 후 language 속성 추가
  > x.language
  undefined // language 속성이 추가된 객체 버려짐
  ```
  - (cf) 몽키-패치(monkey-patch): 런타임에 프로그램의 어떤 기능을 수정해서 사용하는 기법. 자바스크립트에서는 주로 프로토타입을 변경하는 것이 이에 해당.
- 타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링한다.
- string을 String으로 잘못 타이핑하지 않도록 주의해야 한다.
  - string을 매개변수로 받는 메서드에 String 객체를 전달하는 순간 문제가 발생한다.
    ```tsx
    function isGreeting(phrase: String) {
      return ['hello', 'good day'].includes(phrase)
      // Argument of type 'String' is not assignable to parameter of type 'string'.
      // 'string' is a primitive, but 'String' is a wrapper object. Prefer using 'string' when possible.
    }
    ```
  - string은 String에 할당할 수 있지만 String은 string에 할당할 수 없다.
- 기본형 타입은 객체 래퍼에 할당할 수 있기 때문에 타입스크립트는 기본형 타입을 객체 래퍼에 할당하는 선언을 허용한다. 하지만 오해하기 쉽기 때문에 기본형 타입을 사용하는 것이 낫다.
  - new 없이 BigInt와 Symbol을 호출하는 경우는 기본형을 생성하기 때문에 사용해도 좋다.

<br />

## 아이템 11. 잉여 속성 체크의 한계 인지하기

- 타입이 명시된 변수에 객체 리터럴을 할당할 때 타입스크립트는 해당 타입의 속성이 있는지, 그리고 ‘그 외의 속성은 없는지’ 확인한다.
- 잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는 것을 알아야 타입스크립트 타입 시스템에 대한 개념을 정확히 잡을 수 있다.
- 아래의 예제에서는 구조적 타입 시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록 ‘잉여 속성 체크’라는 과정이 수행되었다.

  ```tsx
  interface Room {
    numDoors: number
    ceilingHeightFt: number
  }

  const r: Room = {
    numDoors: 1,
    ceilingHeightFt: 10,
    elephant: 'present',
    // Type '{ numDoors: number; ceilingHeightFt: number; elephant: string; }' is not assignable to type 'Room'.
    // Object literal may only specify known properties, and 'elephant' does not exist in type 'Room'
  }
  ```

- 통상적인 할당 가능 검사에서는 obj가 Room 타입의 부분 집합을 포함하므로 에러가 발생하지 않는다.

  ```tsx
  interface Room {
    numDoors: number
    ceilingHeightFt: number
  }

  const obj = {
    numDoors: 1,
    ceilingHeightFt: 10,
    elephant: 'present',
  }

  const r: Room = obj
  ```

- 엄격한 객체 리터럴 체크
  - 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행된다.
  - 잉여 속성 체크를 이용하면 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써 속성 이름의 오타 같은 실수를 잡을 수 있다.
  - 객체 리터럴이 아닌 경우, 잉여 속성 체크가 적용되지 않는다. 즉, 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다.
  - 타입 단언문을 사용할 때에도 잉여 속성 체크가 적용되지 않는다.
  - 객체 리터럴을 변수에 할당할 때, 잉여 속성 체크를 회피하는 방법
    - 인덱스 시그니처를 통해 추가적인 속성 예상하기 <- 과연 이것은 적절한 방법일까..?
    ```tsx
    interface Options {
      darkMode?: boolean
      [otherOptions: string]: unknown
    }
    const o: Options = { darkmode: true } // 정상
    ```
- 선택적 속성만 가지는 ‘약한’ 타입

  - 모든 속성이 선택적인 타입은 구조적 관점에서 모든 객체를 포함할 수 있다. 이런 약한 타입에 대해서 타입스크립트는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행한다.
  - 잉여 속성 체크와 마찬가지로 오타를 잡는 데 효과적이다.
  - 잉여 속성 체크와 다르게, 약한 타입과 관련된 할당문마다 수행된다. 임시 변수를 제거하더라도 공통 속성 체크는 여전히 동작한다. **← ⁇⁇⁇**

  ```tsx
  interface LineChartOptions {
    logscale?: boolean
    invertedYAxis?: boolean
    areaChart?: boolean
  }

  const opts = { logScale: true }
  const o: LineChartOptions = opts
  // Type '{ logScale: boolean; }' has no properties in common with type 'LineChartOptions'.
  ```
