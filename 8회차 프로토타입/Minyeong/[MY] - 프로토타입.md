# Chapter 6. 프로토타입
- [Chapter 6. 프로토타입](#chapter-6-프로토타입)
  - [6-1. 프로토타입의 개념 이해](#6-1-프로토타입의-개념-이해)
    - [6-1-1. constructor, prototype, instance](#6-1-1-constructor-prototype-instance)

> 자바스크립트 : 프로토타입 기반 언어
>   - 어떤 객체를 원형으로 삼고 복제(참조) = 클래스 기반 언어의 상속

## 6-1. 프로토타입의 개념 이해
### 6-1-1. constructor, prototype, instance
![alt text](06/프로토타입도식.png)  
```js
var instance = new Constructor();
```
- Constructor는 생성자 함수
- new Constructor() 호출 → 새로운 instance 생성
- 이 instance에는 **__proto__(숨겨진 프로퍼티)** 자동 부여
- ✨ `__proto__`는 Constructor의 prototype(프로퍼티) 참조
  - ```js
    Constructor.prototype === instance.__proto__  // true
    ```
> 🤔 prototype? __proto__ ?
>
> - **prototype** : 객체
>   - prototype 내부에 인스턴스가 사용할 메서드 저장
> - **__proto__**(던더 프로토, double underscore proto) : 객체
>   - 인스턴스는 __proto__를 통해 prototype의 메서드에 접근 가능
>      <details>
>      <summary>ES5, ES6에서의 __proto__</summary>
>
>      - ES5.1
>          - [[prototype]] 명칭으로 정의
>          - __proto__는 브라우저가 [[prototype]] 구현한 대상일 뿐
>           - instance.__proto__로 직접 접근 허용X
>           - `Object.getPrototypeOf(instance)`, `Refelect.getPrototypeOf(instance)` 통해 접근 가능
>       - ES6
>         - 대부분의 브라우저가 계속 직접 접근하자 레거시 코드에 대한 호환성 유지 차원에서 정식 인정
>              - 레거시 코드 : 오래되었거나 현재 기준에서 권장되지 않는 방식으로 작성된 코드
>         - 브라우저에서의 호환성을 고려한 지원일 뿐 권장X
>         - Object.getPrototypeOf(instance);
>         - Object.setPrototypeOf(instance, prototype);
>         - Object.create(prototype);
>   </details>

*생성자 함수와 프로토타입 예시*
```js
function Person(name) {
  this._name = name;
}
Person.prototype.getName = function () {
  return this._name;
};

const suzi = new Person("Suzi");

// 1) 이렇게 쓰면?
suzi.__proto__.getName(); // undefined

// 2) 이렇게 쓰면?
suzi.getName(); // "Suzi"
```
- 가정 : Person 생성자 함수의 prototype에 getName 메서드 지정
-  Person의 인스턴스는 __proto__ 프로퍼티를 통해 get Name 호출 가능
- 반환 값 다른 이유
  - 1)의 this = suzi.__proto__
    - 이 객체 내부에는 name 프로퍼티 없음
    - 찾고자 하는 식별자 없으므로 undefined 반환
    - 