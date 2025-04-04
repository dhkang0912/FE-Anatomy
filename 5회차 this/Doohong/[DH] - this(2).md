- [this](#this)
  - [3-2. 명시적으로 this를 바인딩하는 방법](#3-2-명시적으로-this를-바인딩하는-방법)
    - [(1) call 메서드](#1-call-메서드)
    - [(2) apply 메서드](#2-apply-메서드)
    - [(3) call / apply 메서드의 활용](#3-call--apply-메서드의-활용)
      - [1. 유사배열객체(array-like object)에 배열 메서드를 적용](#1-유사배열객체array-like-object에-배열-메서드를-적용)
      - [2. 생성자 내부에서 다른 생성자를 호출](#2-생성자-내부에서-다른-생성자를-호출)
      - [3. 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용](#3-여러-인수를-묶어-하나의-배열로-전달하고-싶을-때---apply-활용)
    - [(4) bind 메서드](#4-bind-메서드)
      - [1. name 프로퍼티](#1-name-프로퍼티)
      - [2. 상위 컨텍스트의 this를 내부함수나 콜백함수에 전달하기](#2-상위-컨텍스트의-this를-내부함수나-콜백함수에-전달하기)
    - [(5) 화살표 함수의 예외사항](#5-화살표-함수의-예외사항)
    - [(6) 별도의 인자로 this를 받는 경우 (콜백 함수 내에서의 this)](#6-별도의-인자로-this를-받는-경우-콜백-함수-내에서의-this)
  - [3-3. 정리](#3-3-정리)
  - [Q\&A](#qa)
    - [Q1. call, apply, bind의 차이는 무엇인가요?](#q1-call-apply-bind의-차이는-무엇인가요)
    - [Q2. call이나 apply를 사용해 유사 배열 객체에 배열 메서드를 적용할 수 있는 이유는?](#q2-call이나-apply를-사용해-유사-배열-객체에-배열-메서드를-적용할-수-있는-이유는)
    - [Q3. arguments와 나머지 매개변수(...args)의 차이점은 무엇인가요?](#q3-arguments와-나머지-매개변수args의-차이점은-무엇인가요)
    - [Q4. bind를 사용하는 목적은 무엇인가요?](#q4-bind를-사용하는-목적은-무엇인가요)
    - [Q5. 배열 메서드에서 콜백 함수의 this를 유지하려면 어떻게 해야 하나요?](#q5-배열-메서드에서-콜백-함수의-this를-유지하려면-어떻게-해야-하나요)

# this
## 3-2. 명시적으로 this를 바인딩하는 방법
- 상황별로 this에 바인딩되는 규칙을 깨고 this에 별도의 대상을 바인딩하는 경우도 있음

- call/apply 메서드 장점 : 명시적으로 별도의 `this`를 바인딩하면서 함수 또는 메서드를 실행
- call/apply 메서드 단점 : `this`를 예측하기 어렵게 만들어 코드 해석을 방해
  - 그럼에도 `ES5 이하 환경`에서는 대안이 없어 실무에서 광범위하게 활용됨

### (1) call 메서드
`Function, prototype.call(thisArg[, arg1[, arg2[, ...]]])`
- **call 메서드 : 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령**
  - call 메서드의 첫번째 인자 : this로 바인딩
  - 이후 인자들 : 호출할 함수의 매개변수
  - 함수를 그냥 실행할 때 this가 전역객체를 참조하는 것과 달리 `call 메서드`를 이용해 직접 지정할 수 있음

  <br>

  - 예시 : call 메서드
    ```js
    var func = function (a, b, c) {
        console.log(this, a, b, c);
    };

    func(1, 2, 3); // 일반함수, Window {...} 1 2 3 (브라우저 기준 전역 객체)
    func.call({ x: 1 }, 4, 5, 6); // call 바인딩, { x: 1 } 4 5 6


    ```
    ```js
    var obj = {
        a: 1,
        method: function (x, y) {
            console.log(this.a, x, y);
        }
    };

    obj.method(2, 3); // 1(this) 2 3
    obj.method.call({ a: 4 }, 5, 6); // 4(call 바인딩된 this) 5 6

    ```

### (2) apply 메서드
`Function.prototype.apply(thisArg, argsArray])`
- **apply 메서드 : call 메서드와 기능적으로 완전히 동일**
  - **첫번째 인자** : this로 바인딩
  - **두번째 인자** : `배열`로 받음, 배열의 요소들을 호출할 함수의 매개변수로 지정

  <br>

  - 예시 : apply 메서드
    ```js
    var func = function (a, b, c) {
        console.log(this, a, b, c);
    };

    // 첫번째 인자 외에 다른 인자들을 매개변수로 받음
    func.apply({ x: 1 }, [4, 5, 6]); // { x: 1 } 4 5 6

    var obj = {
        a: 1,
        method: function (x, y) {
            console.log(this.a, x, y);
        }
    };

    // 첫번재 인자 = this 바인딩
    // 두번째 인자 = 배열로 매개변수를 받음
    obj.method.apply({ a: 4 }, [5, 6]); // 4 5 6

    ```

<br>

### (3) call / apply 메서드의 활용
#### 1. 유사배열객체(array-like object)에 배열 메서드를 적용
- 예시 : call/apply 메서드의 활용, `유사배열객체에 배열 메서드를 적용`
    ```js
    var obj = {
        0: 'a',
        1: 'b',
        2: 'c',
        length: 3
    };

    // 배열 메서드인 push를 객체 obj에 적용하여 'd' 추가
    Array.prototype.push.call(obj, 'd');
    console.log(obj);
    // { 0: 'a', 1: 'b', 2: 'c', 3: 'd', length: 4 }

    // slice 배열 메서드를 적용해 객체를 배열로 전환
    // slice = 인덱스 시작, 마지막 값을 받아 배열 요소를 추출하는 메서드
    // => 만약 아무 값도 넘기지 않으면 원본 배열 얕은 복사
    var arr = Array.prototype.slice.call(obj);
    console.log(arr);
    // ['a', 'b', 'c', 'd'] => 객체를 얕은 복사하여 배열로 반환

    ```
    - 객체에는 배열 메서드를 직접 적용할 수 없음
    - `배열 구조와 유사한 객체의 경우 call, apply 메서드를 통해 배열 메서드를 차용할 수 있음`
      - 배열 구조와 유사한 객체 : 키가 0 또는 양의 정수, length 프로퍼티의 값이 0 또는 양의 정수인 경우
  
<br>

- 예시 : call/apply 메서드의 활용, `arguments, NodeList에 배열 메서드를 적용`
    ```js
    function a() {
        // arguments = {0:1, 1:2, 2:3, length:3}, 자동으로 일반함수에서 arguments 유사 배열 객체 생성
        // 유사 배열 객체를 slice 배열 메서드로 배열로 반환
        var argv = Array.prototype.slice.call(arguments);
        argv.forEach(function (arg) {
            console.log(arg);
        });
    }

    a(1, 2, 3);
    // 출력
    // 1
    // 2
    // 3

    document.body.innerHTML = '<div>a</div><div>b</div><div>c</div>';

    var nodeList = document.querySelectorAll('div');
    // nodeList라는 유사배열객체를 배열로 반환
    var nodeArr = Array.prototype.slice.call(nodeList);
    nodeArr.forEach(function (node) {
        console.log(node);
    });
    // 출력
    // <div>a</div>
    // <div>b</div>
    // <div>c</div>

    ```
    - **arguments : 모든 일반 함수(화살표 함수 제외)에서 사용할 수 있는 유사 배열 객체**
      - 함수가 호출될 때 전달된 모든 인자를 저장
      - 매개변수를 선언하지 않아도 `arguments`로 전달된 인자를 전부 접근 가능
      - 최근에는 `slice.call`로 매개변수 반환하기 보다는 `나머지 매개변수`를 사용
        - **나머지 매개변수(Rest Parameter) : 함수에 전달된 나머지 인자들을 배열로 자동 저장해주는 문법**
            ```js
            function showAll(...args) {
            console.log(args);
            }

            showAll(1, 2, 3); // [1, 2, 3]

            ```
            - `...args`를 통해 나머지 매개변수를 배열로 담아서 활용
            - 항상 매개변수 마지막 위치에 존재해야함
            - 화살표 함수도 사용 가능

    <br>
    
    - **nodeList : HTML 요소들을 리스트 형태로 담고 있는 유사 배열 객체**

<br>

- 예시 : call/apply 메서드의 활용, `문자열에 배열 메서드 적용 예시`
    ```js
    var str = "abc def";

    // push는 문자열에 쓸 수 없음 (문자열은 불변, length는 읽기 전용)
    Array.prototype.push.call(str, 'pushed string');
    // Error: Cannot assign to read only property 'length' of object '[object String]'

    // concat은 배열 메서드이지만 문자열에 사용 가능, 하지만 제대로된 결과를 받을 수 없음
    var result1 = Array.prototype.concat.call(str, 'string');
    console.log(result1);
    // [String ["abc def"], "string"]

    // every → 빈 문자가 아닌지 검사
    var isAllNotEmpty = Array.prototype.every.call(str, function (char) {
    return char !== '';
    });
    console.log(isAllNotEmpty); // false

    // some → 빈 문자가 하나라도 있는지 검사
    var hasEmpty = Array.prototype.some.call(str, function (char) {
    return char === '';
    });
    console.log(hasEmpty); // true

    // map → 각 문자에 "!" 추가
    var newArr = Array.prototype.map.call(str, function (char) {
    return char + '!';
    });
    console.log(newArr);
    // ["a!", "b!", "c!", " !", "d!", "e!", "f!"]

    // reduce → 인덱스를 함께 이어 붙이기
    var newStr = Array.prototype.reduce.call(str, function (acc, char, i) {
    return acc + char + i;
    }, '');
    console.log(newStr);
    // a0b1c2 3d4e5f6

    ```
    - 원본 문자열에 변경을 가하는 메서드(push, pop, shift, unshift, splice 등)는 에러가 남
      - 문자열의 `length` 프로퍼티가 `읽기 전용`이기 때문
    - concat처럼 대상이 반드시 배열이어야 하는 경우 에러는 나지 않지만 `제대로 된 결과가 나오지 않음`
    - call/apply의 형변환은 this를 특정 값으로 바인딩하는 의도와 달라졌지만 slice 메서드는 오직 배열 형태로 `복사`하기 위해 차용됐음  
    => 다른 사람들은 의도 파악이 힘들 수 있음  
    => `Array.from`으로 대체

<br>

- **Array.from : 유사배열객체 또는 순회 가능한 모든 종류의 데이터 타입을 배열로 전환하는 메서드**
  - 예시 : call/apply 메서드의 활용, `ES6의 Array.from 메서드`
    ```js
    var obj = {
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3
    };

    var arr = Array.from(obj);
    console.log(arr); // ['a', 'b', 'c']

    ```

<br>

#### 2. 생성자 내부에서 다른 생성자를 호출
- 생성자 내부에서 다른 생성자와 공통된 내용이 있을 경우 `call, apply`를 이용하여 다른 생성자를 호출하면 반복을 간단하게 줄일 수 있음

<br>

- 예시 : call/apply 메서드의 활용, `생성자 내부에서 다른 생성자를 호출`
  - `Student`, `Employee` 생성자 함수 내부에서 `Person` 생성자 함수를 호출하여 인스턴스 속성을 정의
    ```js
    function Person(name, gender) {
    this.name = name;
    this.gender = gender;
    }

    function Student(name, gender, school) {
    // 인자를 개별로 전달하며 생성자 내부의 this를 상속시킴
    Person.call(this, name, gender); // 부모 생성자 호출
    this.school = school;
    }

    function Employee(name, gender, company) {
    // 인자를 배열로 전달하며 생성자 내부의 this를 상속시킴
    Person.apply(this, [name, gender]); // 배열 형태로 전달
    this.company = company;
    }

    var by = new Student('보영', 'female', '단국대');
    var inho = new Employee('재난', 'male', '구골');

    console.log(by);   // Student { name: '보영', gender: 'female', school: '단국대' }
    console.log(inho); // Employee { name: '재난', gender: 'male', company: '구골' }

    ```
    - `Person.call(this, ...)` : 인자를 개별로 전달하며 생성자 내부의 this를 상속시킴
    - `Person.apply(this, [...])` : 인자를 배열로 전달하며 생성자 내부의 this를 상속시킴
    - `this.property = value` : 생성자 함수 내부에서 인스턴스에 프로퍼티 추가

<br>

#### 3. 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용
- 여러 개의 인수를 받는 메서드에게 `하나의 배열로 인수들을 전달`하고 싶을 때 `apply` 사용

<br>

- 배열에서 최대/최솟값을 구해야할 때 apply를 사용하면 간단히 구현 가능
  - 예시 : call/apply 메서드의 활용, `직접 최대/최솟값 구하는 코드`
    ```js
    var numbers = [10, 26, 3, 16, 45];
    var max = numbers[0];
    var min = numbers[0];

    numbers.forEach(function (number) {
        if (number > max) {
        max = number;
        }
        if (number < min) {
        min = number;
        }
    });

    console.log(max, min); // 45 3

    ```

    <br>

  - 예시 : call/apply 메서드의 활용, `여러 인수를 받는 메서드(Math.max/Math.min)에 apply 적용`
    ```js
    var numbers = [10, 20, 3, 16, 45];

    var max = Math.max.apply(null, numbers);
    var min = Math.min.apply(null, numbers);

    console.log(max, min); // 45 3

    ```

  <br>

  - 예시 : call/apply 메서드의 활용, `ES6 펼치기 연산자 (spread operator) 활용`
    - **동일하게 하나의 배열 안 여러 값들을 인수로 전달하는 방법**
    ```js
    const numbers = [10, 20, 3, 16, 45];

    const max = Math.max(...numbers);
    const min = Math.min(...numbers);

    console.log(max, min); // 45 3

    ```

<br>

### (4) bind 메서드
`Function.prototype, bind(thisArg, arg11, arg2[....]]])`
- **bind 메서드 : ES5에서 추가된 기능, `call`과 비슷하지만 즉시 호출하지 않고 넘겨받은 `this` 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드**
  - 다시 새로운 함수를 호출할 때 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달했던 인수들의 뒤에 이어서 등록됨
  - `bind 메서드의 두가지 목적`
    1. 함수에 `this`를 미리 적용하는 것
    2. 부분 적용 함수를 구현하는 것

  <br>

  - 예시 : bind 메서드, `this 지정과 부분 적용 함수 구현`
    ```js
    var func = function (a, b, c, d) {
    console.log(this, a, b, c, d);
    };

    // bind 메서드로 바인딩 전 => this는 전역객체를 가르킴
    func(1, 2, 3, 4); 
    // this: window (또는 undefined in strict mode), 출력: window 1 2 3 4

    // func에 this를 x:1로 바인딩한 새로운 함수가 담김
    // this만 바인딩
    var bindFunc1 = func.bind({ x: 1 });

    // bindFunc1을 호출하여 this 바인딩된 결과 출력됨
    bindFunc1(5, 6, 7, 8);
    // this: { x: 1 }, 출력: { x: 1 } 5 6 7 8

    // func에 this를 바인딩하고 추가로 앞 2개의 매개변수도 저장
    // this 바인딩과 함께 부분 적용 함수 구현
    var bindFunc2 = func.bind({ x: 1 }, 4, 5);

    // 이미 bindFunc2에서 this와 앞 2개의 매개변수가 저장되어 나머지 매개변수 2개만 입력해도 온전히 출력됨
    bindFunc2(6, 7);
    // this: { x: 1 }, 출력: { x: 1 } 4 5 6 7

    bindFunc2(8, 9);
    // this: { x: 1 }, 출력: { x: 1 } 4 5 8 9

    ```
    - this만 바인딩 : `var bindFunc1 = func.bind({ x: 1 });`
      - `bind`를 활용하여 this를 바인딩한 함수를 사용할 수 있음
    - 부분 적용 함수 구현 : `var bindFunc2 = func.bind({ x: 1 }, 4, 5);`
      - `this` 바인딩과 함께 매개변수를 지정하여 부분 적용 함수를 구현할 수 있음

#### 1. name 프로퍼티
- **bind 메서드 적용 함수의 독특한 성질**
  - `name 프로퍼티`에 동사 bind의 수동태인 `bound`라는 접두어가 붙음
  - 특정 함수의 `name 프로퍼티`가 `bound xxx`인 경우 : 함수명이 `xxx`인 원본 함수에 `bind` 메서드를 적용한 새로운 함수라는 의미
  - call, apply보다 코드를 추적하기 수월함

  ```js
  var func = function (a, b, c, d) {
    console.log(this, a, b, c, d);
  };

  var bindFunc = func.bind({ x: 1 }, 4, 5);

  console.log(func.name);      // func

  // bind 메서드를 활용하여 name 프로퍼티에 bound라는 접두어가 붙음
  console.log(bindFunc.name);  // bound func

  ```

<br>

#### 2. 상위 컨텍스트의 this를 내부함수나 콜백함수에 전달하기
- 메서드 내부함수에서 상위 컨텍스트 메서드의 `this`를 바라보게 하는 우회법
  - 기존 나온 `self` 변수에 상위 컨텍스트 메서드의 `this` 할당
  - 그 외 call, apply, bind를 활용하여 가능

- 예시 : 내부함수에 this 전달, `call vs bind`
  - `call 메서드` 활용
    ```js
    var obj = {
      outer: function () {
        console.log('outer this:', this); // obj

        var innerFunc = function () {
          console.log('innerFunc this:', this); // obj (call로 바인딩)
        };

        // 호출 시 바인딩
        innerFunc.call(this); // 명시적으로 this 바인딩
      }
    };

    obj.outer();

    ```
  - `bind` 메서드 활용
    ```js
    var obj = {
      outer: function () {
        var innerFunc = function () {
          console.log('innerFunc this:', this); // obj (bind로 바인딩)
        }.bind(this); // 미리 this를 바인딩한 새 함수 생성하여 innerFunc에 할당 => 할당 시 바인딩

        console.log('outer this:', this); // obj
        innerFunc();
      }
    };

    obj.outer();

    ```
  - 공통점 : 결국 `this`를 외부 컨텍스트의 `this`와 동일하게 바인딩함
  - 차이점
    - `call` : 호출 시점에 `this`를 바인딩
      - 호출 시점에 바인딩
    - `bind` : `새로운 함수`를 만들어 `this`를 미리 고정
      - 할당 시점에 바인딩

<br>

- 예시 : bind 메서드, `내부 함수에 this 전달`
  - 콜백 함수를 인자로 받는 함수나 메서드 중 기본적으로 콜백 함수 내에서의 `this`에 관여하는 함수 또는 메서드에서도 `bind`를 이용하여 `this` 값을 설정할 수 있음
  ```js
  var obj = {
    logThis: function () {
      console.log(this);
    },

    logThisLater1: function () {
      // 메서드로 함수를 호출하였기 때문에 this = obj
      // 따라서 this.logThis로 호출 가능
      // this.logThis 함수를 실행시키는 순간 => 일반 함수 실행, this = 전역객체
      setTimeout(this.logThis, 500); // ❌ this가 바뀜
    },

    logThisLater2: function () {
      // 일반함수로 호출하면서 bind를 통해 this를 바인딩 => this가 logThisLater2 함수 호출 시 this(obj)로 고정됨
      setTimeout(this.logThis.bind(this), 1000); // ✅ this 고정
    }
  };

  obj.logThisLater1(); //  Window (또는 undefined in strict mode)
  obj.logThisLater2(); //  obj {logThis: f, ...}

  ```
  - 내부 함수에서 함수를 호출하는 경우 일반 함수로 호출됨 => `this`가 전역객체를 가르킴
  - `bind`로 `this`를 바인딩하는 경우 외부 컨텍스트의 this나 원하는 값을 this로 바인딩할 수 있음

### (5) 화살표 함수의 예외사항
- ES6에서 새롭게 도입된 화살표 함수는 실행 컨텍스트 생성 시 `this`를 바인딩하는 과정이 제외됨
- 함수 내부에 `this`가 없고 스코프체인 상 가장 가까운 `this`에 접근
  - 스코프체인 상 가까운 변수를 참조하는 것과 동일

<br>

- 예시 : 화살표 함수 내부에서의 this
  ```js
  var obj = {
    outer: function () {
      var innerFunc = () => {
        console.log('innerFunc this:', this);
      };
      innerFunc();
    }
  };

  obj.outer();

  // 출력
  innerFunc this: { outer: f }

  ```
  - `obj.outer()`에 따라 outer 함수의 this는 obj가 됨
  - 내부 함수가 화살표 함수로 작성되어 스코프 체인을 따라 obj가 this에 할당됨
  - call, apply, bind를 적용할 필요없이 간결하게 화살표 함수로 스코프 체인에 따른 this를 할당 받음

### (6) 별도의 인자로 this를 받는 경우 (콜백 함수 내에서의 this)
- 콜백 함수를 인자로 받는 메서드 중 일부는 추가로 `this`로 지정할 객체를 `인자로 지정(thisArg)`할 수 있는 경우가 있음
  - 주로 여어 내부 요소에서 같은 동작을 반복 수행하는 `배열 메서드`에서 자주 발생

<br>

- 예시 : thisArg를 받는 경우, `forEach 메서드`
  ```js
  var report = {
    sum: 0,
    count: 0,

    add: function () {
      // args 변수에 arguments를 배열로 변환하여 할당
      var args = Array.prototype.slice.call(arguments); // arguments = add 함수의 모든 인자

      // forEach(콜백함수, thisArg)로 this 유지
        // 콜백함수, 이후 this로 thisArg가 설정됨을 확인 가능
      // 배열을 순회하면서 콜백 함수를 실행 => report.sum과 report.count가 배열을 순회하며 차례대로 수정됨
      args.forEach(function (entry) {
        this.sum += entry;
        ++this.count;
      }, this); 
    },

    average: function () {
      return this.sum / this.count;
    }
  };

  report.add(60, 85, 95);
  console.log(report.sum, report.count, report.average()); // 240 3 80
  ```
  - `forEach` : forEach(콜백함수, thisArg) 형태로 this를 지정할 수 있음

<br>

- 예시 : 콜백 함수와 함께 thisArg를 인자로 받는 메서드
  ```js
  // ✅ 배열(Array) 메서드

  Array.prototype.forEach(callback[, thisArg])
  // 배열의 모든 요소를 순회하며 callback 실행 (return 없음)

  Array.prototype.map(callback[, thisArg])
  // 배열의 각 요소에 callback을 적용한 새 배열 반환

  Array.prototype.filter(callback[, thisArg])
  // callback 결과가 true인 요소만 모아 새 배열 반환

  Array.prototype.some(callback[, thisArg])
  // 하나라도 callback이 true면 true 반환 (OR)

  Array.prototype.every(callback[, thisArg])
  // 모든 요소가 callback 조건을 만족하면 true 반환 (AND)

  Array.prototype.find(callback[, thisArg])
  // 조건을 만족하는 첫 번째 요소 반환 (없으면 undefined)

  Array.prototype.findIndex(callback[, thisArg])
  // 조건을 만족하는 첫 번째 요소의 인덱스 반환 (없으면 -1)

  Array.prototype.flatMap(callback[, thisArg])
  // map 후 결과를 1단계 평탄화한 새 배열 반환

  // ✅ 배열 유사 객체 → 배열로 변환

  Array.from(arrayLike[, mapFn[, thisArg]])
  // 유사 배열/이터러블을 배열로 변환 (옵션으로 mapFn 사용 가능)

  // ✅ Set 메서드

  Set.prototype.forEach(callback[, thisArg])
  // Set의 모든 요소를 순회하며 callback 실행

  // ✅ Map 메서드

  Map.prototype.forEach(callback[, thisArg])
  // Map의 모든 key-value 쌍을 순회하며 callback 실행

  ```
  - [, thisArg] : 대괄호 안에 들어있는 것은 선택 사항이라는 의미로 thisArg는 선택 사항

<br>

## 3-3. 정리
- 명시적 `this` 바인딩이 없는 한 늘 성립되는 원리
- ✨자바스크립트에서의 `this` 정리
  1. 전역 공간에서의 `this`
     - 전역 객체를 참조함  
     - 브라우저: `window`  
     - Node.js: `global`

  <br>

  2. 메서드로서 호출한 함수의 `this`
     - 호출한 **객체(메서드명 앞에 있는 것)**가 `this`가 됨  
     - 예: `obj.method()` → `this === obj`


  <br>


  3. 함수로서 호출한 함수의 `this`
     - 기본적으로 **전역 객체**를 참조함  
     - `strict mode`에선 `undefined`
     - 메서드의 **내부 함수도 마찬가지**

      ```js
      var obj = {
        method: function () {
          function inner() {
            console.log(this); // window 또는 undefined
          }
          inner();
        }
      };
      obj.method();
      ```

  <br>


  4. 콜백 함수 내부의 `this`
     - 콜백을 호출하는 **"제어권을 가진 함수"가 정의한 방식**에 따름
     - 정의된 방식이 없으면 기본값인 **전역 객체(window)** 참조
       ```js
       [1, 2, 3].forEach(function (el) {
         console.log(this); // 기본적으로 window
       });
       ```

     - `thisArg`로 명시적으로 지정 가능
       ```js
       [1, 2, 3].forEach(function (el) {
         console.log(this); // { x: 1 }
       }, { x: 1 });
       ```

  <br>

  5. 생성자 함수에서의 `this`
     - `new` 키워드로 호출 시, 생성될 **인스턴스 객체**를 참조

       ```js
       function Person(name) {
         this.name = name;
       }
       const p = new Person('보영');
       console.log(p.name); // '보영'
       ```


<br>

- 위 규칙에 부합하지 않는 명시적 `this` 바인딩

  - this와 관련된 함수 메서드들 요약

    1. `call`, `apply`
       - **`this`를 명시적으로 지정하고** 함수를 **즉시 실행**
       - 차이점: 인자 전달 방식
         ```js
         func.call(thisArg, arg1, arg2, ...);
         func.apply(thisArg, [arg1, arg2, ...]);
         ```

    <br>

    2. `bind`
       - **`this`를 지정하고**, 선택적으로 인자를 고정해서  
         **새로운 함수를 반환** (실행은 나중에)
         ```js
         const bound = func.bind(thisArg, arg1, arg2);
         bound(); // 실행은 나중에
         ```

    <br>

    3. 콜백 기반 메서드에서의 `thisArg`
       - `forEach`, `map`, `filter`, `some`, `every`, `find`, `reduce`,  
         `Set.prototype.forEach`, `Map.prototype.forEach` 등은  
         **콜백 내부에서 사용할 `this`를 두 번째 인자로 전달**할 수 있음

          ```js
          const obj = { prefix: '▶' };
          ['a', 'b', 'c'].forEach(function (item) {
            console.log(this.prefix + item);
          }, obj);
          // ▶a ▶b ▶c
          ```

<br><br>

## Q&A

### Q1. call, apply, bind의 차이는 무엇인가요?

`call`, `apply`, `bind`는 모두 함수의 `this`를 명시적으로 바인딩할 수 있는 메서드입니다.  
가장 큰 차이는 **실행 여부와 인자 전달 방식**입니다.  
`call`은 this를 바인딩하고 함수에 인자를 **나열**하여 전달하며 **즉시 실행**합니다.  
`apply`는 `call`과 똑같지만 인자를 **배열**로 전달합니다.  
반면 `bind`는 함수 자체를 실행하지 않고, this와 일부 인자를 바인딩한 **새로운 함수를 반환**합니다. 이 반환된 함수는 나중에 실행할 수 있습니다.  
따라서 즉시 실행이 필요하다면 `call`이나 `apply`를, 나중에 사용할 함수를 만들고 싶다면 `bind`를 사용하는 것이 적절합니다.

---

### Q2. call이나 apply를 사용해 유사 배열 객체에 배열 메서드를 적용할 수 있는 이유는?

자바스크립트에서 배열 메서드는 기본적으로 `Array.prototype`에 정의되어 있으며, 배열 객체에서만 사용할 수 있습니다. 하지만 배열처럼 보이는 객체(예: arguments, NodeList, 문자열 등)도 인덱스와 length 프로퍼티를 가지고 있다면, `call`이나 `apply`를 통해 배열 메서드를 빌려와 사용할 수 있습니다.  
예를 들어, `Array.prototype.slice.call(arguments)`를 사용하면 `arguments`라는 유사 배열 객체를 진짜 배열로 복사할 수 있습니다. 이 방식은 배열 메서드의 `this`를 유사 배열 객체로 바꿔주는 것으로, 배열처럼 작동하게 만들어주는 유용한 트릭입니다.

---

### Q3. arguments와 나머지 매개변수(...args)의 차이점은 무엇인가요?

`arguments`는 모든 일반 함수 내부에서 자동으로 생성되는 유사 배열 객체입니다. 함수에 전달된 모든 인자를 순서대로 가지고 있으며, 배열처럼 인덱스로 접근할 수 있지만 진짜 배열은 아니기 때문에 map, forEach 같은 배열 메서드는 바로 사용할 수 없습니다.  
반면 `...args`는 ES6에서 도입된 문법으로, 함수 매개변수 중 나머지 인자들을 **배열 형태로 수집**합니다. 이 `...args`는 진짜 배열이기 때문에 배열 메서드를 바로 사용할 수 있고, 화살표 함수에서도 사용할 수 있습니다. 실무에서는 가독성과 사용성을 고려해 나머지 매개변수를 사용하는 것이 더 일반적입니다.

---

### Q4. bind를 사용하는 목적은 무엇인가요?

`bind`는 함수를 **즉시 실행하지 않고**, 특정 `this` 값과 일부 인자를 바인딩한 **새로운 함수를 반환**합니다. 이렇게 반환된 함수는 나중에 원하는 시점에 실행할 수 있습니다.  
`bind`의 주요 용도는 두 가지입니다. 첫째, 특정 this를 고정하고 싶을 때입니다. 예를 들어 콜백 함수에서 this가 변경되는 것을 방지할 수 있습니다. 둘째, 부분 적용 함수를 만들 때입니다. 예를 들어 `func.bind(null, 1, 2)`처럼 일부 인자를 미리 채워두고 나머지 인자는 실행 시점에 전달하도록 구성할 수 있습니다. `bind`는 특히 이벤트 핸들러나 setTimeout 등에서 많이 사용됩니다.

---

### Q5. 배열 메서드에서 콜백 함수의 this를 유지하려면 어떻게 해야 하나요?

배열 메서드 중에는 `forEach`, `map`, `filter` 등처럼 콜백 함수를 사용하는 것들이 있습니다. 이때 콜백 함수 내부의 `this`는 기본적으로 전역 객체를 가리키거나 `undefined`가 되는데, 원하는 객체를 `this`로 사용하고 싶다면 메서드의 두 번째 인자로 `thisArg`를 전달하면 됩니다.  
예를 들어, `[1, 2, 3].forEach(callback, thisArg)`처럼 쓰면, `callback` 함수 안에서의 `this`는 `thisArg`로 지정한 객체가 됩니다. 이렇게 하면 내부에서 외부 객체의 속성 등을 안전하게 참조할 수 있습니다.  
또는 콜백 함수를 화살표 함수로 작성하거나, `callback.bind(this)`로 바인딩한 함수로 전달하는 것도 this를 고정하는 좋은 방법입니다.

