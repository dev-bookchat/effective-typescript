---

## 아이템 9. 타입 단언보다는 타입 선언을 사용하기

- 타입 단언을 하게 되면 잉여속석 체크가 적용되지 않는다.
- 타입 단언은 강제로 타입을 지정했기 떄문에 타입 체커에게 오류를 무시하라라는것이다.
- 타입스크립트는 DOM에 접근할 수 없다 => 이럴땐 타입 단언을 사용해주는게 맞다.
- (!) 을 사용하면 null이 아니라는 단언문으로 해석된다.
- 단언문은 컴파일 과정 중에 제거되어 타입 체커는 알 수 없다.
- null이 아니라고 확신 할 수 있을때 사용한다.
- 타입 단언문 타입 간에 변환을 할 수 없다.
- HTMLElement는 HTMLElement | null 의 서브타입이므로 동작한다.
- Person 과 HTMLElement는 서로의 서브타입이 아니기 떄문에 변환이 불가능하다.
- 모든 타입은 unknown의 서브타입이므로 unknown이 포함된 단언문은 항상 동작한다.

1. 타입 단언 (as Type) 보다는 타입 선언(: Type)을 사용한다.

---

## 아이템 10. 객체 래퍼 타입 피하기

- javaScript의 기본형 (string,number,boolean,null,undefined,symbol)
- 기본형은 불변형, 메서드를 가지지 않는다 로 객체와 구분할 수 있다.
- string.charAt()등으로 메서드를 가지고 있는것 처럼 보인다.
- string기본형은 메서드가 없지만 String객체 타입이 서로 자유롭게 변환된다.
- string기본형에 chartAt메서드를 사용할때 js는 String객체로 래핑(wrap)하고 메서드를 호출하고 래핑한 객체를 버린다.
- 객체 래퍼 타입의 자동 변환
- number-Number / boolean-Boolean / symbol-Symbol / 기본형-래퍼타입
- js에서 래퍼타입으로 기본형 값에서 메서드를 사용 할 수 있다.
- TypeScript는 기본형과 객체 래퍼 타입을 별도로 모델링한다.
- string을 사용할때는 특히 유의해야함 -> String이라고 잘못 타이핑 하기 쉽다.
- string을 매개변수로 받는 메서드에 String객체를 전달 하면 문제가 생긴다.
- (pharse:String) | 잘못된 예시
- 'String'형식의 인수는 'string'형식의 매개변수에 할당 될 수 없습니다. error
- striing은 String에 할당 할 수 있지만, String은 string에 할당 할 수 없다.
- TypeScript가 제공하는 타입선언은 전부 기본형 타입으로 되어있다.

1. 객체 래퍼타입은 지양하고 , 기본형 타입을 사용해야 한다. (String 대신 string, Number대신 number등)

---

## 아이템 11. 잉여 속성 체크의 한계 인지하기 (이해가 전반적으로 되지 않음..ㅠ)

- TypeScript는 해당타입의 속성이 있는지, '그 외의 속성은 없는지' 확인한다.
- TypeScript에서 잉여 속성 체크 과정이 수행된다.
- document나 new HTMLAnchorElement는 객체 리터럴이 아니기 떄문에 잉여 속성 체크가 되지 않는다.
- 잉여 속성 체크는 타입 단언문을 사용할 때에도 적용되지 않는다. => 타입 선언문을 사용해야하는 이유 중 하나.
-
