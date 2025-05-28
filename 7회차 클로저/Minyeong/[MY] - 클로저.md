# Chapter 5. 클로저

- [Chapter 5. 클로저](#chapter-5-클로저)
  - [5-1. 클로저의 의미 및 원리 이해](#5-1-클로저의-의미-및-원리-이해)
  - [5-2. 클로저와 메모리 관리](#5-2-클로저와-메모리-관리)
  - [5-3. 클로저 활용 사례](#5-3-클로저-활용-사례)
    - [5-3-1. 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때](#5-3-1-콜백-함수-내부에서-외부-데이터를-사용하고자-할-때)
    - [5-3-2. 접근 권한 제어(정보 은닉)](#5-3-2-접근-권한-제어정보-은닉)
    - [5-3-3. 부분 적용 함수](#5-3-3-부분-적용-함수)
    - [5-3-4. 커링 함수](#5-3-4-커링-함수)
  - [5-4. 정리](#5-4-정리)

## 5-1. 클로저의 의미 및 원리 이해
**클로저(Closure)**
- 함수가 선언될 당시의 **스코프(Lexical Environment)를 기억**해 외부 함수가 종료된 이후에도 해당 변수에 접근할 수 있는 현상
  - 자바스크립트는 함수가 생성될 때 자신이 선언된 위치의 스코프 기억함
  - 그 스코프 안의 지역 변수를 참조하는 내부 함수가 외부로 전달되면, 외부 함수의 실행 컨텍스트가 종료되더라도 해당 변수는 GC(가비지 컬렉터)에 의해 제거되지 않고 유지됨
    <details>
    <summary>📌 Lexical Environment 되짚기</summary>

    **Lexical Environment란?**
    - 자바스크립트 엔진이 **식별자(변수, 함수 등)** 를 **어디서, 어떻게 찾을지** 결정하는 구조
    - 지금 이 시점에서 어떤 변수들을 접근할 수 있는지 정의
    
    **구성 요소**  
    ① Environment Record
     - 현재 스코프의 변수와 함수 저장소
     - `{ a: 1, b: function() {} }` 같은 형태의 키-값 쌍

    ② Outer Environment Reference
     - **외부 렉시컬 환경을 가리키는 참조**
     - 한 단계 바깥의 스코프(예: 외부 함수나 전역 컨텍스트)를 가리킴
     - 이것이 **스코프 체인**을 형성
    </details>
        <details>
    <summary>📌 가비지 컬렉션(Garbage Collection, GC)</summary>

    **가비지 컬렉션이란?**
    - 더 이상 사용되지 않는 메모리를 **자동으로 해제**해주는 메커니즘
    - 자바스크립트는 명시적으로 메모리를 해제하지 않아도 되는 자동 메모리 관리 언어
      - 사용하지 않는 객체를 자동으로 탐지하여 메모리에서 제거
    - 제거 대상 : 아무도 참조되지 않는 값
      - 어떤 값을 참조하는 변수가 하
나라도 있다면 그 값은 수집 대상에 포함시키지 않음
    - 실행 시기 : 주기적/특정 조건 하에 자동 실행
      - 자바스크립트 엔진(V8 등)의 내부 결정
      - 개발자가 직접 호출할 수 없음
    
    *예시*
    ```js
    function outer() {
        let a = 10;
        function inner() {
            console.log(a);
        }
        inner(); // 호출하고 끝
    }
    outer(); // outer와 inner 모두 종료
    ```  
    - outer 함수의 실행 컨텍스트가 종료되면 LexicalEnvironment에 저장된 식
별자들(a, inner)에 대한 참조 지움
    - inner가 외부로 전달되지 않음
    - a를 참조하는 함수가 더 이상 존재하지 않음 → GC 가능
    </details>

**기본 구조**
```js
function outer() {
  let count = 0; // 외부 함수의 지역 변수
  return function inner() {
    return ++count; // 지역 변수를 참조
  };
}

const counter = outer();
console.log(counter()); // 1
console.log(counter()); // 2
```
- inner는 outer의 지역 변수 count를 계속 기억하고 사용
- outer()가 실행 마쳐도 inner()가 count를 참조하고 있어 수집되지 않음

<br/>

> 🤔 클로저가 아닌 예시
>
> ```js
> function outer() {
>   let a = 1;
>   function inner() {
>     console.log(++a);
>   }
>   inner(); // outer 내에서만 inner 실행
> }
> outer(); // a는 외부로 전달되지 않아 GC 대상
> ```
> - inner가 외부로 반환되지 않아 a는 클로저가 되지 않고 메모리에서 제거됨
>

<br/>

**클로저의 핵심 조건**  
① 내부 함수가 외부 함수의 지역 변수 참조  
② 내부 함수가 외부 함수 실행 종료 이후에도 접근 가능  
③ 단순 return 뿐 아니라 이벤트 핸들러, 타이머, 콜백 등에 전달된 경우도 포함 → 외부로 전달
<details>
<summary>다양한 클로저 예시</summary>

*1. setInterval*
```js
(function () {
  let a = 0;
  let intervalId = setInterval(function () {
    if (++a >= 5) clearInterval(intervalId);
    console.log(a);
  }, 1000);
})();
```
- 지역 변수 a를 참조하는 내부 함수가 setInterval로 외부에 전달됨 → 클로저
  
*2. 이벤트 리스너*
```js
(function () {
  let count = 0;
  const btn = document.createElement('button');
  btn.innerText = 'Click me';
  btn.addEventListener('click', function () {
    console.log(++count, 'clicked');
  });
  document.body.appendChild(btn);
})();
```
- click 이벤트가 발생할 때마다 count에 접근 → 클로저 발생
</details>

## 5-2. 클로저와 메모리 관리
클로저는 **의도적으로** 외부 함수의 **지역 변수**를 **함수 외부에서 계속 참조**하도록 만듦  
- 클로저는 메모리 누수를 일으킨다?! ❌
  - 메모리를 더 오래 유지하도록 함
  - GC가 수거하지 않는 이유는 여전히 참조 중이기 때문
  - 누수인지 의도된 유지인지는 개발자의 설계에 따라 다름

**메모리 누수란?**  
- 참조되지 않아야 할 값이 계속 참조되고 있어 GC 대상이 되지 못하는 상태
  - 대표적인 사례: 순환 참조, 제거하지 않은 이벤트 리스너, 타이머 등
  - 최근의 브라우저(특히 Chrome의 V8 엔진)에서는 대부분의 자동 GC 문제 개선됨
- ✨ 참조를 끊어주어 해당 메모리가 수거되도록 해야 함
  - 식별자에 null 또는 undefined 할당
    <details>
    <summary>클로저 해제 예시</summary>

    *1. return에 의한 클로저 메모리 해제*
    ```js
    var outer = (function () {
      var a = 1;
      var inner = function () {
        return ++a;
      };
      return inner;
    })();
    console.log(outer()); // 2
    console.log(outer()); // 3
    outer = null; // ❗참조 해제 → 메모리 회수 가능
    ```
    *2. setInterval에 의한 클로저 해제*
    ```js
    (function () {
      var a = 0;
      var intervalId = null;
      var inner = function () {
        if (++a >= 10) {
          clearInterval(intervalId); // 타이머 해제
          inner = null;              // ❗함수 참조 해제
        }
        console.log(a);
      };
      intervalId = setInterval(inner, 1000);
    })();
    ```
    *3. addEventListener에 의한 클로저 해제*
    ```js
    (function () {
      var count = 0;
      var button = document.createElement('button');
      button.innerText = 'click';

      var clickHandler = function () {
        console.log(++count, 'times clicked');
        if (count >= 10) {
          button.removeEventListener('click', clickHandler); // 이벤트 해제
          clickHandler = null; // ❗참조 해제
        }
      };

      button.addEventListener('click', clickHandler);
      document.body.appendChild(button);
    })();
    ```
    </details>
## 5-3. 클로저 활용 사례

### 5-3-1. 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

### 5-3-2. 접근 권한 제어(정보 은닉)

### 5-3-3. 부분 적용 함수

### 5-3-4. 커링 함수

## 5-4. 정리