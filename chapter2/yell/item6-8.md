## 아이템 6. 편집기를 사용하여 타입 시스템 탐색하기

- 타입스크립트를 설치하면, 컴파일러 `tsc`와 단독으로 실행할 수 있는 타입스크립트 서버 `tsserver`를 실행할 수 있다.

- VS Code와 같은 편집기를 사용해 타입스크립트의 언어서비스를 사용할 수 있는데, 이 언어서비스에는 코드 자동 완성, 명세 검사, 검색, 리팩터링이 포함된다.

- 타입스크립트 언어 서비스는 편집기에서 모델링된 타입 선언 파일을 볼 수 있다.

<br />

---

<br />

## 아이템 7. 타입이 값들의 집합이라고 생각하기

- 타입은 ‘할당 가능한 값들의 집합'이라고 생각하면 된다. 여기서 집합은 타입의 범위.

- 타입스크립트의 타입은 겹쳐지는 집합(벤 다이어그램)으로 표현된다.

  | **타입**  | **집합 개념**                             |
  | --------- | ----------------------------------------- |
  | never     | 아무 값도 포함하지 않는 공집합            |
  | literal   | 한 가지 값만 포함                         |
  | union     | 두 개 혹은 세 개의 값을 포함, 합집합      |
  | interface | 범위가 무한대, 타입 범위 내의 값들의 집합 |
  | unknown   | 전체 집합                                 |

<br />

---

<br />

## 아이템 8. 타입 공간과 값 공간의 심벌 구분하기

- 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다.
- `class`나 `enum` 같은 키워드는 타입과 값 두 가지로 사용될 수 있다.
- `typeof`, `this` 그리고 많은 다른 연산자들(`&`, `|`)과 키워드들(`extends`, `in`)은 타입 공간과 값 공간에서 다른 목적으로 사용될 수 있다.

  | **타입**                                 | **값**                      |
  | ---------------------------------------- | --------------------------- |
  | 심벌 앞에 `type`이나 `interface`         | 심벌 앞에 `const`이나 `let` |
  | 심벌 뒤에 타입 선언 `:` 또는 단언문 `as` | 심벌 뒤에 `=`               |
