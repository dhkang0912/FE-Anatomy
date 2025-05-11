- [콜백함수](#콜백함수)
  - [1. 콜백 함수란?](#1-콜백-함수란)
  - [2. 제어권](#2-제어권)
    - [2-1. 호출 시점](#2-1-호출-시점)
    - [2-2. 인자](#2-2-인자)
    - [2-3. this](#2-3-this)
  - [3. 콜백 함수는 함수다](#3-콜백-함수는-함수다)

# 콜백함수
## 1. 콜백 함수란?
> **콜백 함수 (callback function)** : 다른 코드의 인자로 넘겨주는 함수
- 콜백 함수를 넘겨받은 코드는 적절한 시점에 이를 실행함
- **일상의 예시 : 알람 시계를 설정**
  - 알람 시계를 설정하는 함수를 호출
  - 호출 당시에는 아무것도 하지 않고 정해진 시간이 됐을 때 `알람이 울리는 결과를 반환`  
  => 시간을 직접 확인하며 깨는 것과 달리 `시계에 요청을 하면서 알람을 울리는 명령의 제어권을 시계에 전달한 것`

<br>

- 콜백 함수는 `제어권`과 관련이 깊음

<br>

> **callback** : 호출하다 + 되돌아오다 = `되돌아 호출해달라는 명령`
>   - 함수 X를 호출하면서 `특정 조건일 때 함수 Y를 실행해서 나에게 알려달라고 요청`하는 것
>   - `요청 받은 함수 X 입장`에서는 `조건`이 갖춰졌는지 여부를 스스로 판단하고 `Y를 직접 호출함`

- 다른 코드(함수 또는 메서드)에게 인자를 넘겨주면서 제어권도 위임한 함수
- 위임받은 코드는 자체적인 내부 로직에 의해 이 콜백 함수를 적절한 시점에 실행할 것

## 2. 제어권
### 2-1. 호출 시점
- 콜백 함수 예제 : `setInterval`
    ```js
    var count = 0;
    var timer = setInterval(function () {
    console.log(count);
    if (++count > 4) clearInterval(timer);
    }, 300);

    ```
    1. count 변수를 선언하고 0으로 할당
    2. timer 변수를 선언하고 setInterval을 실행한 결과를 할당
    3. setInterval을 호출 시 두개의 매개변수 전달
       1. 익명함수
       2. 300
       - `setInterval` 함수 구조
         - `var intervalID scope setInterval(func, delay, parami, param2, ...]);`
         - `scope` : window 객체 또는 Worker의 인스턴스
           - 일반적인 `브라우저 환경에서는 window를 생략, 함수처럼 사용 가능
         - 매개 변수 : func, delay는 필수
           - `func` : 실행되는 함수
           - `delay` : ms 단위의 숫자, 매 ms마다 func이 실행되며 결과는 리턴하지 않음
         - 세번째 매개변수부터 선택
           - `func` 함수 실행 시 매개변수로 전달할 인자
         - `setInterval`를 실행하면 반복적으로 실행되는 내용 자체를 특정할 수 있는 고유한 ID가 반환됨 => 이를 변수에 담아 종료(`clearInterval`)할 수 있게 하기 위함

<br>

- 콜백 함수 예제 : `setInterval` 쉽게 보기
    ```js
    // count 변수를 0으로 초기화
    var count = 0;

    // 콜백 함수(cbFunc) 정의
    var cbFunc = function () {
    console.log(count); // 현재 count 출력
    if (++count > 4) clearInterval(timer); // count를 1 증가시킨 후 4보다 크면 타이머 종료
    };

    // 300ms마다 cbFunc를 호출하는 타이머 설정
    var timer = setInterval(cbFunc, 300);

    // - 실행 결과 예시 -
    // 0 (0.3초)
    // 1 (0.6초)
    // 2 (0.9초)
    // 3 (1.2초)
    // 4 (1.5초)

    ```
    - `timer` 변수에는 `setInterval의 ID`값이 담김
    - `setInterval`에 전달한 첫 번째 인자인 `cbFunc`함수 (콜백 함수)는 0.3초마다 자동으로 실행됨
    - 콜백 함수 내부에서는 `count` 값을 출력하고 `count`를 1만큼 증가시킨 다음 그 값이 4보다 반복 실행을 종료하라고 함

    <br>

    - **코드 실행 방식과 제어권**
    ![Alt text](<images/코드 실행 방식과 제어권.png>)

    <br>

    - **결과** 
      - 콜솔창에는 0.3초에 한 번씩 숫자가 0부터 1씩 증가하며 출력되다가 4가 출력된 이후 종료됨
      - `setInterval`이라는 `다른 코드`에 첫번째 인자로 `cbFunc` 함수를 넘겨줌   
      -> 제어권을 넘겨받은 `setInterval`이 스스로 판단하여 적절한 시점 (0.3초마다) 익명함수를 실행  
      => `제어권을 넘겨받은 코드는 콜백 함수 호출 시점에 대한 제어권을 가짐`

### 2-2. 인자
- 콜백 함수 예제 : `Array.prototype.map`
  ```js
  // [10, 20, 30] 배열에 대해 map 메서드를 사용
  var newArr = [10, 20, 30].map(function (currentValue, index) {
    console.log(currentValue, index); // 현재 요소 값과 인덱스 출력
    return currentValue + 5; // 각 요소에 5를 더한 값을 반환
  });

  console.log(newArr); // 새로운 배열 출력

  // - 실행 결과 -
  // 10 0
  // 20 1
  // 30 2
  // [15, 25, 35]

  ```
  - newArr 변수 선언 후 우항의 결과를 할당
       - 우항은 배열 [10, 20, 30]에 map 메서드 호출
       - 첫번째 매개변수로 익명함수 전달
      > **map 메서드 동작 방식**
      > `Array.prototype.map(callback[, thisArg])`
      > `callback: function(currentValue, index, array)`
      > - 첫번째 인자 : callback 함수
      > - 두번째 인자 : 생략 가능함, 콜백 함수 내부에서 this로 인식할 대상을 특정
      > - 배열의 모든 요소들을 하나씩 꺼내어 `콜백 함수를 반복 호출하고 실행` 결과들을 모아 새로운 배열을 만듦
      > - 콜백 함수의 `첫번째 인자` = 배열의 요소 중 `현재 값`, `두번째 인자` = 현재값의 `인덱스`, `세번째 인자` = map 메서드의 `대상이 되는 배열 자체`가 담김

<br>

- 콜백함수 예제 : Array.prototype.map - 인자의 순서를 임의로 바꾸어 사용한 경우
  ```js
  // 잘못된 매개변수 순서를 가진 map 함수 예제
  var newArr2 = [10, 20, 30].map(function (index, currentValue) {
    console.log(index, currentValue);
    return currentValue + 5;
  });

  console.log(newArr2);

  // -- 실행 결과 --
  // 10 0
  // 20 1
  // 30 2
  //  [5, 6, 7]

  ```
  - 사람은 이름을 통해 인지하여 순서가 바뀌더라도 괜찮을거라고 생각하지만 컴퓨터는 순서를 통해 구분하여 원하는 값이 나오지 않음
  - `currentValue`의 위치가 `두번째라 인덱스 값`을 더하게 됨

<br>

- 이처럼 `콜백 함수의 제어권을 넘겨받은 코드`는 콜백 함수를 호출할 때 `인자에 어떤 값들을 어떤 순서로 넘길 것인지에 대해 제어권을 가짐`

### 2-3. this
- 콜백 함수 예제 : Array.prototype.map - map 메서드 직접 구현
  ```js
  Array.prototype.map = function (callback, thisArg) {
    var mappedArr = [];

    for (var i = 0; i < this.length; i++) {
      var mappedValue = callback.call(thisArg || window, this[i], i, this);
      mappedArr[i] = mappedValue;
    }

    return mappedArr;
  };

  ```
  - this에는 thisArg 값이 있을 경우 그 값을 지정하고 없으면 전역객체를 지정
  - `call/apply 메서드`는 콜백 함수에 this 값을 명시적으로 지정해서 실행하는 메서드
  - `첫번째` 인자 : 메서드의 this가 배열을 가르킬 것이므로 배열의 i번째 요소 값
  - `두번째` 인자 : i 값
  - `세번째` 인자 : 배열 자체
  - `결과` : mappedValue에 담겨 mappedArr의 i번째 인자에 할당
  - 제어권을 넘겨받을 코드에서 `call/apply 메서드의 첫번째 인자`에 콜백 함수 내부에서 this가 될 대상을 명시적 바인딩
  
<br>

- 예시 : 콜백 함수 내부에서의 this
  ```js
  // (1) setTimeout 내부의 this는 window
  setTimeout(function () {
    console.log('(1)', this); 
  }, 300); // (1) Window { ...}

  // (2) forEach 콜백 내부의 this는 undefined (strict mode) 또는 window
  [1, 2, 3, 4, 5].forEach(function (x) {
    console.log(this); 
    // (2) Window { ... }
  });

  // (3) 이벤트 리스너 내부의 this는 해당 DOM 요소
  document.body.innerHTML += '<button id="a">클릭</button>';

  document.querySelector('#a')
    .addEventListener('click', function (e) {
      console.log(this, e);
      // (3) <button id="a">클릭</button>
      // MouseEvent { isTrusted: true, ... }
    });
  ```
  - (1) `setTimeout` 내부에서 콜백 함수 호출 => call 메서드의 첫번째 인자에 전역객체를 넘김
    - 콜백 함수 내부에서의 this가 전역객체를 가르킴

  - (2) `forEach`는 별도의 인자로 this를 받는 경우 해당에 해당하지만 별도의 this를 넘겨주지 않아 전역객체를 가르킴

  - (3) `addEventListener`는 내부에서 콜백 함수를 호출할 때 call 메서드의 첫번째 인자에 `addEventListener` 메서드의 this를 그대로 넘기게 정의되어있음
    - 콜백 함수 내부에서의 this가 `addEventListener`를 호출한 주체인 `HTML` 엘리먼트를 가르킴


## 3. 콜백 함수는 함수다
