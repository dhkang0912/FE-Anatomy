- [this](#this)
  - [3-2. 명시적으로 this를 바인딩하는 방법](#3-2-명시적으로-this를-바인딩하는-방법)
    - [(1) call 메서드](#1-call-메서드)
    - [(2) apply 메서드](#2-apply-메서드)
    - [(3) call / apply 메서드의 활용](#3-call--apply-메서드의-활용)
      - [1. 유사배열객체(array-like object)에 배열 메서드를 적용](#1-유사배열객체array-like-object에-배열-메서드를-적용)

# this
## 3-2. 명시적으로 this를 바인딩하는 방법
- 상황별로 this에 바인딩되는 규칙을 깨고 this에 별도의 대상을 바인딩하는 경우도 있음

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

