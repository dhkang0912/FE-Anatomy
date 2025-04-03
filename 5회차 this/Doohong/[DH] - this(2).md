- [this](#this)
  - [3-2. 명시적으로 this를 바인딩하는 방법](#3-2-명시적으로-this를-바인딩하는-방법)
    - [(1) call 메서드](#1-call-메서드)
    - [(2) apply 메서드](#2-apply-메서드)
    - [(3) call / apply 메서드의 활용](#3-call--apply-메서드의-활용)
      - [1. 유사배열객체(array-like object)에 배열 메서드를 적용](#1-유사배열객체array-like-object에-배열-메서드를-적용)
      - [2. 생성자 내부에서 다른 생성자를 호출](#2-생성자-내부에서-다른-생성자를-호출)
      - [3. 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용](#3-여러-인수를-묶어-하나의-배열로-전달하고-싶을-때---apply-활용)
    - [(4) bind 메서드](#4-bind-메서드)

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

    func(1, 2, 3, 4); 
    // this: window (또는 undefined in strict mode), 출력: window 1 2 3 4

    var bindFunc1 = func.bind({ x: 1 });
    bindFunc1(5, 6, 7, 8);
    // this: { x: 1 }, 출력: { x: 1 } 5 6 7 8

    var bindFunc2 = func.bind({ x: 1 }, 4, 5);
    bindFunc2(6, 7);
    // this: { x: 1 }, 출력: { x: 1 } 4 5 6 7

    bindFunc2(8, 9);
    // this: { x: 1 }, 출력: { x: 1 } 4 5 8 9

    ```