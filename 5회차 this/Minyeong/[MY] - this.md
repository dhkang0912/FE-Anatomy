*this*

- [Chapter 3. this](#chapter-3-this)
  - [3-1. 상황에 따라 달라지는 this](#3-1-상황에-따라-달라지는-this)
    - [3-1-1. 전역 공간에서의 this - 전역 객체](#3-1-1-전역-공간에서의-this---전역-객체)
    - [3-1-2. 메서드로 호출할 때 그 메서드 내부에서의 this](#3-1-2-메서드로-호출할-때-그-메서드-내부에서의-this)
    - [3-1-3. 함수로서 호출할 때 그 함수 내부에서의 this](#3-1-3-함수로서-호출할-때-그-함수-내부에서의-this)
    - [3-1-4. 콜백 함수 호출 시 그 함수 내부에서의 this](#3-1-4-콜백-함수-호출-시-그-함수-내부에서의-this)
    - [3-1-5. 생성자 함수 내부에서의 this](#3-1-5-생성자-함수-내부에서의-this)
- [Q\&A](#qa)
  - [1. 자바스크립트에서 this가 다른 객체지향 언어와 다르게 작동하는 이유는 무엇인가요?](#1-자바스크립트에서-this가-다른-객체지향-언어와-다르게-작동하는-이유는-무엇인가요)
  - [2. 함수로 호출한 경우와 메서드로 호출한 경우 this가 어떻게 달라지나요?](#2-함수로-호출한-경우와-메서드로-호출한-경우-this가-어떻게-달라지나요)
  - [3. 메서드 내부의 중첩 함수에서 this가 의도와 다르게 전역 객체를 가리키는 문제를 어떻게 해결할 수 있나요?](#3-메서드-내부의-중첩-함수에서-this가-의도와-다르게-전역-객체를-가리키는-문제를-어떻게-해결할-수-있나요)
  - [4. 콜백 함수에서 this가 달라지는 이유가 무엇인가요?](#4-콜백-함수에서-this가-달라지는-이유가-무엇인가요)
  - [5. 생성자 함수를 호출하는 방법은 무엇이고, 또 this는 어떤 객체를 가리키게 되나요?](#5-생성자-함수를-호출하는-방법은-무엇이고-또-this는-어떤-객체를-가리키게-되나요)

# Chapter 3. this
다른 객체지향 언어에서의 this : 클래스로 생성한 인스턴스 객체
- 클래스에서만 사용할 수 있어 혼란의 여지가 없거나 많지 않음

**자바스크럽트에서의 this는?**  
어디든 이용 가능  
함수와 객체(메서드)의 구분이 느슨하기 때문에 이를 구분하기 위한 유일한 기능이기도 함

## 3-1. 상황에 따라 달라지는 this
함수를 어떤 방식으로 호출하느냐에 따라 값이 달라짐
- this는 기본적으로 **실행 컨텍스트가 생성될 때** 함께 결정됨
- 실행 컨텍스트는 **함수를 호출할 때** 생성됨

<br/>

### 3-1-1. 전역 공간에서의 this - 전역 객체
전역 컨텍스트를 생성하는 주체 : 전역 객체
- **전역 객체** : 자바스크립트 런타임 환경에 따라 다른 이름 및 정보 가짐
- 브라우저 환경(window), Node.js 환경(global)
  ![alt text](03-1/전역공간this.png)
  <details>
  <summary>전역 공간에서만 발생하는 성질</summary>

  전역 변수를 선언하면 자바스크립트 엔진은 이를 전역 객체의 프로퍼티로 할당
  - 자바스크립트의 모든 변수는 특정 객체(실행 컨텍스트의 LexicalEnvironment)의 프로퍼티로서 동작하기 때문
    - 어떤 변수를 호출하면 LexiclaEnvironment를 조회해 일치하는 프로퍼티가 있으면 해당 값 봔환
    - GlobalEnv가 전역 객체를 참조하는데, 전역 컨텍스트의 LexicalEnvironment가 이 GlobalEnv를 참조(전역 객체 그대로 참조)
    ```js
    var a = 1;
    console.log(a);        // 1
    console.log(window.a); // 1
    console.log(global.a); // 1
    console.log(this.a);   // 1
    ```
    - var로 선언하지 않고 window의 프로퍼티에 직접 할당해도 똑같이 동작하나 삭제 할 때 다름
    - 전역 변수로 선언한 경우 삭제 안 됨 : 사용자가 의도치 않게 삭제하는 것을 방지하기 위해 → 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의
    ```js
    var a = 1;
    delete window.a;                  // false
    console.log(a, window.a, this.a); // 1 1 1
    
    var b = 2;
    delete b;                         // false
    console.log(b, window.b, this.b); // 2, 2, 2

    window.c = 3;
    delete window.c;                  // true
    console.log(c, window.c, this.c); // ReferenceError: c 정의 안 됨

    window.d = 4;
    delete d;                         // true
    console.log(d, window.d, this.d); // ReferenceError: d 정의 안 됨
    ```

  </details>

<br/>

### 3-1-2. 메서드로 호출할 때 그 메서드 내부에서의 this
**함수 vs 메서드**  
- 공통점 : 미리 정의한 동작을 수행하는 코드 뭉치
- 차이점 : 독립성 유무
  - 함수 : 그 자체로 독립적인 기능 수행
  - 메서드 : 자신을 호출한 대상 객체에 관한 동작 수행
    - 점 또는 대괄호를 사용해 함수 이름(프로퍼티) 앞에 객체가 명시된 경우
    - ✨ this는 마지막에 명시된 직전의 객체
    ```js
    // 함수로서 호출
    var func = function (x) {
        console.log(this, x)
    };
    func(1);          // window {...} 1

    // 메서드로서 호출
    var obj = {
        method: func
    };
    obj.method(2);    // {method: f} 2
    obj['method'](2); // {method: f} 2
    ```
<br/>

### 3-1-3. 함수로서 호출할 때 그 함수 내부에서의 this
**함수 내부에서의 this - 전역 객체**  
어떤 함수를 함수로서 호출할 경우 this가 지정되지 않음  
- this는 호출한 주체에 대한 정보가 담김
- 함수로서 호출은 호출 주체를 명시하지 않은 것이기 때문에 정보를 알 수 없음
- 따라서 전역 객체를 가리키게 됨
<br/>

**메서드의 내부함수에서의 this**  
```js
var obj1 = {
    outer : function () {
        console.log(this);       // (1)
        var innerFunc = function () {
            console.log(this);   // (2) (3)
        }
        innerFunc();

        var obj2 = {
            innerMethod : innerFunc
        };
        obj2.innerMethod();
    }
};
obj1.outer()
```
① 객체를 생성(내부엔 outer 프로퍼티가 있고 익명함수 연결됨)해 변수 obj1에 할당  
② obj1.outer() 호출  
- obj1.outer 함수의 실행 컨텍스트가 생성되면서 호이스팅, 스코프 체인 정보 수집, this 바인딩
- outer 앞에 .이 있었기 때문에 메서드로 호출 → this에 obj1 바인딩
- (1) : obj1 객체 정보 출력  

③ 호이스팅된 innerFunc은 지역변수로 outer 스코프 내에서만 접근 가능, 익명함수 할당  
④ innerFunc() 호출
- innerFunc 함수의 실행 컨텍스트가 생성되면서 호이스팅, 스코프 체인 정보 수집, this 바인딩
- 함수명 앞에 .이 없으므로 함수로서 호출 → this 지정 안 됨, 스코프 체인상 최상위 객체인 전역 객체(window) 바인딩
- (2) : window 객체 정보 출력

⑤ 호이스팅된 변수 obj2도 지역변수, innerFunc와 연결된 innerMethod가 프로퍼티로 있는 객체 할당  
⑥ obj2.innerMethod() 호출
- obj2.innerMethod 함수의 실행 컨텍스트가 생성
- 함수명 앞에 .이 있었기 때문에 메서드로 호출 → this에 obj2 바인딩
- (3) : obj2 객체 정보 출력  

<br/>

**메서드의 내부함수에서의 this를 우회하는 방법** 
호출 주체가 없을 때 자동으로 전역 객체를 바인딩 하지 않고 호출 당시 주변 환경의 this를 그대로 상속받기 위한 방법
- ES5까지는 내부함수에 this 상속할 방법 없음
- 변수 활용 : self, _this, that, _ 등
  - 상위 스코프의 this를 저장해 내부함수에서 활용하려는 수단
```js
var obj = {
    outer : function () {
        console.log(this);       // (1) {outer: f}
        var innerFunc1 = function () {
            console.log(this);   // (2) Window {...}
        }
        innerFunc1();

        var self = this;
        var innerFunc2 = function () {
            console.log(self)    // (3) {outer: f}
        };
        innerFunc2();
    }
};
obj.outer()
```

<br/>

**this를 바인딩하지 않는 함수 - 화살표 함수**   
ES6에서는 함수 내부에서 this가 전역 객체를 바라보는 문제를 보완하고자 화살표 함수 도입
- **화살표 함수** : 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠짐 → 상위 스코프의 this를 그대로 활용
```js
var obj = {
    outer : function () {
        console.log(this);       // (1) {outer: f}
        var innerFunc = () => {
            console.log(this);   // (2) {outer: f} 
        }
        innerFunc();
    }
};
obj.outer()
```
<br/>

### 3-1-4. 콜백 함수 호출 시 그 함수 내부에서의 this
**콜백 함수** : 제어권을 다른 함수(또는 메서드)에게 넘겨주는 함수
- 콜백 함수는 다른 함수의 내부 로직에 따라 실행
- ✨ this도 기본적으로 전역 객체를 참조하나 제어권을 받은 함수에서 별도로 this가 될 대상을 지정하면 그 대상을 참조하게 됨
```js
setTimeout(function () {console.log(this);}, 300);  // (1)

[1, 2, 3, 4, 5].forEach(function (x) {              // (2)
    console.log(this, x)
})

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector('#a')
    .addEventListener('click', function (e) {        // (3)
        console.log(this, e);
    })
```
(1) : setTimeout 함수는 300ms만큼 시간 지연 뒤에 콜백 함수 실행 → 0.3초 뒤 전역 객체 출력  
(2) : forEach 메서드는 배열의 각 요소를 차례대로 하나씩 꺼내어 그 값을 콜백 함수의 첫번째 인자로 삼아 함수 실행 → 전역 객체와 배열의 각 요소가 총 5회 출력  
(3) : addEventListener는 지정한 HTML 엘리먼트에 'click' 이벤트가 발생할 때마다 그 이벤트 정보를 콜백 함수의 첫 번째 인자로 삼아 함수 실행 → 버튼을 클릭하면 지정한 엘리먼트와 클릭 이벤트에 관한 정보가 담긴 객체 출력
- (1), (2)의 내부에서는 콜백 함수를 호출할 때 대상이 될 this를 지정하지 않기 때문에 전역 객체 참조
- (3)은 자신의 this를 상속하도록 되어 있어 . 앞부분이 this가 됨

<br/>

### 3-1-5. 생성자 함수 내부에서의 this
**생성자 함수** : 어떤 공통된 성질을 지니는 객체를 생성하는 데 사용하는 함수
- 객체지향 언어에서의 생성자(🎨클래스, class), 클래스를 통해 만든 객체(🐟인스턴스, instance)
- 프로그래밍적으로 생성자는 구체적인 인스턴스를 만들기 위한 틀
- 자바스크립트에서는?
  - **생성자** :`new` 명령어 + 함수
  - **인스턴스** : 생성자 함수로 호출되어 그 내부에서의 this
  - 생성자 함수를 호출하면 생성자의 prototype 프로퍼티를 참조하는 `__proto___`(프로퍼티 있는 인스턴스) 만들고, 미리 준비된 공통 속성 및 개성을 해당 객체(this)에 부여 
    ```js
    var Cat = function (name, age) {
        this.bark = '미야옹';
        this.name = name;
        this.age = age;
    };
    var choco = new Cat('초코', 3);
    var cheeze = new Cat('치즈', 5);
    console.log(choco, cheeze);

    /*
    결과
    Cat {bark: '미야옹', name: '초코', age: 3};
    Cat {bark: '미야옹', name: '치즈', age: 5};
    */
    ```
*정리*  
생성자(클래스) = 팔레트, 붕어빵 틀 / 객체(인스턴스) = 짠 물감, 붕어빵 

---

# Q&A
## 1. 자바스크립트에서 this가 다른 객체지향 언어와 다르게 작동하는 이유는 무엇인가요?
자바스크립트에서는 함수와 객체의 구분이 느슨하고, 클래스 기반이 아닌 프로토타입 기반 언어이기 때문에 this가 더 유연하게 설계되었습니다.  
다른 언어에서는 this가 주로 클래스의 인스턴스를 가리키지만, 자바스크립트에서는 함수 호출 방식에 따라 this가 결정됩니다.  
(예를 들어 같은 함수라도 전역 공간에서 호출하면 전역 객체를 가리키고, 객체의 메서드로 호출하면 해당 객체를 가리키는 등 동적 바인딩을 따릅니다.)

## 2. 함수로 호출한 경우와 메서드로 호출한 경우 this가 어떻게 달라지나요?
함수로 호출한 경우에는 this가 전역객체를 가리킵니다.  
브라우저에서는 window, node.js에서는 global을 가리키게 됩니다.  
반면, 메서드로 호출할 경우 this는 그 메서드를 호출한 객체를 가리킵니다. 함수명 앞에 점이나 대괄호를 통해 객체가 명시되어있느냐에 따라 this의 바인딩 대상이 달라집니다.

## 3. 메서드 내부의 중첩 함수에서 this가 의도와 다르게 전역 객체를 가리키는 문제를 어떻게 해결할 수 있나요?
변수를 사용하거나 화살표 함수를 통해 해결할 수 있습니다.  
ES5에선 상위 스코프의 this를 변수에 할당하고, ES6에선 화살표 함수를 사용해 상위 스코프의 this를 상속받을 수 있습니다.

## 4. 콜백 함수에서 this가 달라지는 이유가 무엇인가요?
콜백 함수는 제어권을 넘겨 받은 함수의 로직에 따라 this를 바인딩 하기 때문입니다.  
setTimeout이나 forEach와 같이 호출자가 명확하지 않다면 기본적으로 전역 객체를 가리키게 됩니다.  
반면, 이벤트 리스너나 일부 메서드와 같이 호출 시에 this를 명확히 지정해 준다면 그 대상을 참조하게 됩니다.

## 5. 생성자 함수를 호출하는 방법은 무엇이고, 또 this는 어떤 객체를 가리키게 되나요?
생성자 함수를 호출할 때는 new 키워드를 사용합니다.
new를 붙여 호출하면 자바스크립트는 내부적으로 새로운 객체를 생성하고, 생성자 함수 내부의 this는 그 새로 생성된 객체를 가리키게 됩니다.