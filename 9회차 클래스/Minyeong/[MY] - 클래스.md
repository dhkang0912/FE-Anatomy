# Chapter 7. 클래스
- [Chapter 7. 클래스](#chapter-7-클래스)
  - [7-1. 클래스와 인스턴스의 개념 이해](#7-1-클래스와-인스턴스의-개념-이해)
  - [7-2. 자바스크립트의 클래스](#7-2-자바스크립트의-클래스)

> 👀 **들어가기 전**
>
> 자바스크립트는 프로토타입 기반 언어
> - '상속' 개념 존재X
> - ES6에서 클래스 문법 추가
>   - 일정 부분 프로토타입을 활용하고 있음


## 7-1. 클래스와 인스턴스의 개념 이해
- 일반적인 개념 이해
  - **클래스(class)**
    - 어떤 개체의 공통 속성을 모아 정의한 추상적인 개념
    - super-, sub-를 접목해 상위 클래스(superclass)/하위 클래스(subclass)로 표현
      - 하위 클래스는 상위 클래스를 포함하면서 더 구체적인 개념 추가
      - 즉, 클래스는 하위로 갈수록 상위 클래스의 속성을 상속하면서 더 구체적인 요건 추가/변경
    <details>
    <summary>클래스 간의 상하관계</summary>
    
    ![alt text](07/클래스간상하관계.png)
    - 음식 : 과일의 superclass
    - 과일 : 음식의 subclass, 귤류의 superclass
    - 귤류 : 음식의 sub-subclass
    </details>
  - **인스턴스(instance)**
    - 어떤 클래스의 속성을 지니는 실존하는 개체
- 접근 방식
  - 현실 세계상
    - 인스턴스 → 클래스 
    - 개체들이 존재한 상태에서 이를 구분짓기 위해 클래스 도입
    - 하나의 개체가 같은 레벨에 있는 서로 다른 여러 클래스의 인스턴스일 수 있음
  - 프로그래밍 언어상
    - 클래스 → 인스턴스 
    - 클래스를 바탕으로 인스턴스 생성
    - 한 인스턴스는 하나의 클래스만을 바탕으로 생성
      - 다중상속을 지원하든 안 하든 인스턴스 입장에서 클래스는 '직계존속'
  - 비교
    - 공통점 : 공통 요소를 지니는 집단을 분류하기 위한 개념
    - 차이점
      - 현실 - 인스턴스들로부터 공통점 발견해 클래스 정의(추상적 개념)
      - 프로그래밍 - 클래스가 먼저 정의되어야만 그로부터 공통적인 요소 지니는 개체 생성 가능(추상적인 대상일 수도, 구체적인 개체일 수도 있음)

## 7-2. 자바스크립트의 클래스
- 생성자 함수를 호출하면 인스턴스 생성
  - 생성자 함수 = 일종의 클래스
  - 생성자 함수.prototype 객체 내부 요소 = 인스턴스에 상속
    - 프로토타입 체이닝에 의한 참조이나 결과적으론 상속과 동일하게 동작
    - 인스턴스에 상속(참조)되는지 여부에 따라 static member와 prototype method(instance member)로 나뉨
  ![alt text](07/프로토타입에클래스개념적용.png)

*클래스 관점에서 바라본 프로토타입 시스템 예시*
```js
// 생성자
var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
};

// (프로토타입) 메서드
Rectangle.prototye.getArea = function () {
  return this.width * this.height;
};

// 스태틱 메서드
Rectangle.isRectangle = function (instance) {
  return instance instanceof Rectangle && 
    instance.with > 0 && instance.height > 0;
}

var rect1 = new Rectangle(3, 4)
console.log(rect1.getArea()); // 12 (O)
console.log(rect1.isRectangle(rect1)); // Error (X)
console.log(Rectangle.isRectangle(rect1)); // true
```
- 프로토타입 메서드
  - `rect1.getArea()`
    - `rect1.(__proto__).getArea()`
    - this = rect1
  - 인스턴스에서 직접 호출할 수 있는 메서드
- 스태틱 메서드
  - `rect1.isRectangle(rect1)`
    - rect1 (X) → rect1.`__proto__` (X) → rect1.`__proto__.__proto__`(= Object.prototype) (X)
    - Uncaught TypeError : not a function
  - 인스턴스에서 직접 접근할 수 없는 메서드
  - 생성자 함수를 this로 해야만 호출 가능

![alt text](07/인스턴스에서직접접근여부.png)  


> 🤔 **자바스크립트에서의 클래스는?**
> - 구체적인 인스턴스가 사용할 메서드를 정의한 '틀'의 역할을 담당하는 목적 => **추상적 개념**
> - 클래스 자체를 this로 해 직접 접근해야만 하는 스태틱 메서드를 호출할 때의 클래스 => **하나의 개체**로 취급