- [리액트 렌더링 과정](#리액트-렌더링-과정)
  - [1. 개요 및 미리보기 (이정환 - React.js의 렌더링 방식 살펴보기)](#1-개요-및-미리보기-이정환---reactjs의-렌더링-방식-살펴보기)
    - [1-1. 웹 브라우저 동작 방식](#1-1-웹-브라우저-동작-방식)
    - [1-2. 리액트의 렌더링 프로세스](#1-2-리액트의-렌더링-프로세스)
  - [2. 상세 (\[번역\] 리액트 렌더링 동작의 (거의) 완벽한 가이드 \[A (Mostly) Complete Guide to React Rendering Behavior\])](#2-상세-번역-리액트-렌더링-동작의-거의-완벽한-가이드-a-mostly-complete-guide-to-react-rendering-behavior)
    - [2-1. 리액트의 렌더링이란 무엇인가?](#2-1-리액트의-렌더링이란-무엇인가)
      - [(1) 전체적인 렌더링 과정](#1-전체적인-렌더링-과정)
    - [2-2. 렌더와 커밋 단계](#2-2-렌더와-커밋-단계)
    - [2-3. React는 렌더를 어떻게 다룰까?](#2-3-react는-렌더를-어떻게-다룰까)
      - [1. 렌더링 순서 만들기](#1-렌더링-순서-만들기)
      - [2. 표준적인 렌더 동작](#2-표준적인-렌더-동작)
      - [3. React의 렌더링 규칙](#3-react의-렌더링-규칙)
  - [참고 자료](#참고-자료)

# 리액트 렌더링 과정
## 1. 개요 및 미리보기 (이정환 - React.js의 렌더링 방식 살펴보기)
<details>
<summary>
&nbsp 펼쳐서 확인하기
</summary>

### 1-1. 웹 브라우저 동작 방식
- Critical Rendering Path
![alt text](<critical rendering path.png>)

- 사용자의 이벤트에 대한 업데이트
  - JavaScript가 DOM을 수정하면 업데이트가 발생
  - DOM이 수정되면 Critical Rendering Path가 다시 실행됨

  ![alt text](<브라우저 업데이트.png>)
  => 이렇게 직접적으로 DOM을 계속 수정하게 되면 Reflow, Repaint를 계속 발생시키게 됨  -> `성능 저하`
  - Reflow (Layout 재계산), Repaint (화면에 실제 픽셀 단위로 표현)은 매우 비싼 연산 과정

<br>

- 그럼 어떻게 자바스크립트를 조작하는게 좋을까?
  - 동시에 발생한 `업데이트를 모아서 한번에 수정`하는게 DOM을 최대한 변경하지 않아 성능에 좋음  
  => 개발이 복잡해질수록 이렇게 `바닐라 자바스크립트를 조작하는게 어려울` 수 있다!  
  - `React JS`에서는 `자동`으로 업데이트된 내용들을 `한번에 모아서` DOM에 `반영`하도록 추상화되어있음
    - ![alt text](<리액트 추상화.png>)

### 1-2. 리액트의 렌더링 프로세스
- React의 화면 UI 렌더링 과정
  1. Render Phase
     - 컴포넌트를 계산하고 업데이트 사항을 파악하는 단계
       - React 컴포넌트가 렌더링 해야하는 UI를 Virtual DOM이라는 객체 값으로 변환하는 과정
     1. 계산한 업데이트 사항을 React Element 객체로 파악함
          - React Element : 컴포넌트가 렌더링하고자 하는 모든 정보를 담고 있는 객체
          - ![alt text](<Render Phase.png>)
     2. React Element들을 모아 Virtual DOM 생성
          - ![alt text](<Virtual Dom.png>)
          - Virtual DOM : React Element라고 부르는 객체 값의 모임
            - 실제 DOM은 아니고 `DOM의 복제판`이며 `값으로 표현된 UI(Value UI)`라고 이해하는 게 더 정확함

     - Render Phase 과정 정리
       - ![alt text](<Render Phase 정리.png>)
  2. Commit (확정, 결정) Phase
    - 변경사항을 Virtual DOM을 Actual DOM에 반영함
      - ![alt text](<Commit Phase.png>)

<br>

- 리액트 렌더링 프로세스 정리
![alt text](<리액트 렌더링 프로세스.png>)
  - 굳이 이렇게 복잡한 과정을 거쳐서 리액트가 렌더링 되는 이유 : DOM 수정을 최소화해서 대부분의 상황에 충분히 빠른 업데이트를 보장하기 위해서

<br>

- 업데이트 발생 시 재조정(Reconciliation) 발생
  ![alt text](재조정.png)
  1. Render Phase를 처음부터 다시 실행 -> 새로운 Virtual DOM 생성 
     - ![alt text](<새로운 Virtual DOM 생성.png>)
  2. Next Virtual DOM <-> Prev Virtual DOM의 차이점 비교
     - ![alt text](Diffing.png)
  3. 개선된 차이점을 Actual DOM에 한번에 업데이트
     - ![alt text](<Diffing Update.png>)

  - Virtual DOM의 단점
    - Virtual DOM의 재조정으로 대부분의 상황에 `충분히 빠른 속도로 업데이트`를 할 수 있지만 `항상 최고의 속도를 보장하는 것은 아님 `
    - Virtual DOM을 생성하고 비교하는데도 `연산이 소요`됨

</details>

## 2. 상세 ([번역] 리액트 렌더링 동작의 (거의) 완벽한 가이드 [A (Mostly) Complete Guide to React Rendering Behavior])
<details open>
<summary>
&nbsp 펼쳐서 확인하기
</summary>

### 2-1. 리액트의 렌더링이란 무엇인가?
- 리액트 렌더링 : React가 컴포넌트에게 현재 Props와 State에 기반하여 UI에서 어떻게 보여지고 싶은지 알려달라고 요청하는 과정

#### (1) 전체적인 렌더링 과정
- 렌더링 과정 동안 React는 컴포넌트 트리의 루트에서부터 시작하여 아래쪽으로 순환하며 업데이트가 필요하다고 표시된 컴포넌트를 전부 찾음
- 업데이트가 필요한 컴포넌트를 찾은 경우 클래스형 컴포넌트인 경우 `classComponentInstance.render()`를 함수형 컴포넌트인 경우 `FunctionComponent()`를 호출하고 렌더 결과물을 저장
- 렌더 결과물은 JSX 구문으로 보통 작성되며 JS가 컴파일 되고 배포 준비가 되는 시점에서 `React.createElement()` 호출로 변환됨
  - `createElement` : 일반적인 JS 객체 형식의 React 엘리먼트를 반환하며 이 엘리먼트는 생성하고자 하는 UI 구조를 설명함

  > 예시 
  > ```JS
  > // 다음과 같은 JSX 문법이:
  > return <SomeComponent a={42} b="testing">Text here</SomeComponent>
  >
  > // 이런 식의 호출로 변환됩니다:
  > return React.createElement(SomeComponent, {a: 42, b: "testing"}, "Text Here")
  > 
  > // 그렇게해서 이런 리액트 엘리먼트 객체가 됩니다:
  > {type: SomeComponent, props: {a: 42, b: "testing"}, children: ["Text Here"]}
  > ```

<br>

- 전체 컴포넌트 트리에서 렌더 결과물을 모두 수집하면 **재조정(Reconciliation)**을 진행
  - 재조정 : 새로운 객체 트리(Virtual DOM, Value UI)와 비교하여 의도한대로 보여지기 위한 실제 DOM에 적용시켜야 할 모든 변경사항을 수집하며 비교 및 계산 과정을 거침
- 이후 계산된 모든 변경사항을 하나의 동기적 시퀀스(synchoronous sequence)로 실제 DOM에 적용시킴

### 2-2. 렌더와 커밋 단계
- 위의 React 동작 과정을 개념적으로 2단계로 나눔
  1. 렌더 단계 : 컴포넌트를 렌더링하고 **변경 사항을 계산하는 모든 과정**이 이루어지는 단계
  2. 커밋 단계 : 변경 사항을 **실제 DOM에 적용**하는 단계

<br>

- 커밋 이후
  1. 커밋 단계를 거쳐서 DOM을 업데이트하고 나면 React는 요청된 DOM 노드(ref.current)와 컴포넌트 인스턴스(클래스 컴포넌트의 this)를 가르키도록 모든 참조사항을 업데이트함 
  2. 이후 클래스 생명주기 메서드 혹은 useLayoutEffect 훅이 동기적으로 실행됨 
     - 클래스 컴포넌트인 경우 클래스 생명주기 메서드 `componentDidMount`(최초 마운트)와 `componentDidUpdate`(업데이트) 실행됨
       - [클래스 컴포넌트 생명주기 메소드 <image src="클래스 컴포넌트 리액트 생명주기 메소드 다이어그램.png"/>](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)
     - 또는 함수형 컴포넌트인 경우 `useLayoutEffect` 훅을 동기적으로 실행
  3. 이후 React는 짧은 타임 아웃을 세팅하고 타임아웃이 끝나면 `useEffect` 훅을 실행 (= 수동적 효과, Passive Effect)

<br>

- **✨ 알아둘 것**
  1. 렌더링 != DOM 업데이트 
     - 컴포넌트는 가시적인 변화가 없어도 렌더링될 수 있음
     - 직전과 동일한 렌더링 결과물을 반환하는 경우 변경 사항 X
       - Prev Virtual DOM <-> Next Virtual DOM을 비교하는 Diffing 과정을 거쳤을 때 변화가 없는 경우
     - `Concurrent Mode`에서 React는 컴포넌트 렌더링을 여러번 할 수 있지만 다른 업데이트가 현재 작업을 무효화한다면 매번 렌더링 결과물을 폐기함
       - `Concurrent Mode` : 렌더링을 백그라운드에서 여러번 실행할 수 있음 (createRoot()를 통해 설정 가능)  
        => 화면을 그리면서도 더 나은 렌더링을 미리 계산하는 기능이 존재
       - 위 기능을 통해 화면을 그리는 중이더라도 현재 렌더링을 취소하고 새로운 업데이트 요소를 반영한 렌더링을 시작할 수 있음

### 2-3. React는 렌더를 어떻게 다룰까?
#### 1. 렌더링 순서 만들기
- 최초의 렌더가 끝난 이후 React가 리렌더링을 Queue에 넣도록 하는 방법 (리렌더링을 발생시키는 방법)
  1. 클래스형 컴포넌트 
     - `this.setState()`
     - `this.forceUdate()`
  2. 함수형 컴포넌트
     - `useState` setters
     - `useReducer` dispathces
  3. 그 외
     - `ReactDOM.render(<App>)`을 다시 호출
       - 루트 컴포넌트에서 `forceUpdate()`를 호출하는 것과 동일 

#### 2. 표준적인 렌더 동작
> React는 기본적으로 부모 컴포넌트가 렌더링되면, 그 안에 있는 **모든 자식 컴포넌트를 재귀적으로 렌더링**함

- 예시
  - 상황 : `A > B > C > D` 컴포넌트 트리 => `B 컴포넌트` 수정
    - 컴포넌트 트리 : React 앱의 컴포넌트 계층 구조를 나타내는 트리 (Virtual DOM 기반)
  - 동작
    - B 컴포넌트에서 setState()가 호출되어 B의 리렌더링이 큐에 들어감
    - React는 트리의 최상단부터 렌더 패스 (Render Pass)를 시작합니다.
      - 렌더 패스 : React가 컴포넌트를 평가하고 Virtual DOM을 생성하는 한 번의 렌더링 과정
    - React는 A에는 업데이트가 필요하다는 마크가 없는 것을 보고 그냥 지나침
    - React는 B에 업데이트가 필요하다는 마크가 있는 것을 보고 렌더링합니다. B는 `<C/>` 를 리턴합니다.
    - C는 업데이트가 필요하다는 마크는 없지만, B가 렌더링되었기 때문에 React는 한 단계 밑으로 내려가서 C까지 렌더링합니다. C는 `<D/>`를 리턴합니다.
    - D 역시도 업데이트가 필요하다는 마크는 없지만, 부모 컴포넌트인 C가 렌더링되었기 때문에 React는 한 단계 밑으로 내려가서 D도 렌더링합니다.

<br>

> 즉, 일반적으로 컴포넌트가 렌더링되면 **그 안에 있는 모든 컴포넌트 역시 렌더링** 됨

> 일반적인 렌더링 과정에서 React는 **Props가 변경되었는지 여부는 신경쓰지 않음**, 부모 컴포넌트가 렌더링되면 무조건 자식 컴포넌트도 렌더링되는 것
- `<App>`컴포넌트에서 `setState()`를 호출하면 컴포넌트 트리 안에 모든 컴포넌트가 렌더링됨  
  => 매번 업데이트할 때마다 애플리케이션 전체를 다시 그리는 것처럼 동작
  - 이렇게 전체를 다시 그린 후 과거 가상돔과 바뀐 가상돔을 비교해서 변경된 부분만 DOM에 반영 
  - React는 계속해서 컴포넌트에게 렌더링을 요청하고 그 결과물을 비교하여 시간과 연산 성능이 필요함

> **렌더링은 React가 DOM에 변화를 줘야 할지 여부를 파악하는 방법**일 뿐 나쁜 것은 아님

#### 3. React의 렌더링 규칙
> React 렌더링의 핵심적인 규칙 : 렌더링은 **순수**해야하며 사이드 이펙트를 만들어서는 안됨
>
> 렌더링 과정은 "**순수 함수(pure function)**"처럼 동작함
> 
> 즉, 외부 상태를 변경하지 않고, 같은 입력이 주어지면 **항상 같은 결과**를 반환해야 함.
- 렌더링 로직은 다음과 같은 행위를 해서는 안됨
  - 현재 존재하는 변수와 객체를 변경하는 행위 => 외부 변수 변경 금지
  - `Math.random()` 또는 `Date.now()` 등의 랜덤 값을 만들어내는 행위 => 매번 다른 결과가 나와 불필요한 리렌더링이 발생할 수 있음
  - 네트워크 요청을 만들어내는 행위 => 렌더링할 때마다 요청이 갈 수 있음
  - 상태 업데이트를 Queue에 넣은 행위 => 렌더링 과정 중 상태 업데이트가 있으면 무한 루프에 빠질 수 있음
- 다음과 같은 행위는 괜찮을 수 있음
  - 렌더링 도중 새롭게 만들어진 객체를 변경하는 행위 => 외부 상태를 건드리지 않음
  - 에러를 발생시키는 행위
  - 캐싱된 값처럼 아직 만들어지지 않은 데이터를 "Lazy 초기화"하는 행위


</details>


## 참고 자료
1. [React.js의 렌더링 방식 살펴보기 - 이정환 | 2023 NE(O)RDINARY CONFERENCE
](https://www.youtube.com/watch?v=N7qlk_GQRJU)
2. [[번역] 리액트 렌더링 동작의 (거의) 완벽한 가이드 [A (Mostly) Complete Guide to React Rendering Behavior]
](https://velog.io/@arthur/%EB%B2%88%EC%97%AD-%EB%A6%AC%EC%95%A1%ED%8A%B8-%EB%A0%8C%EB%8D%94%EB%A7%81-%EB%8F%99%EC%9E%91%EC%9D%98-%EA%B1%B0%EC%9D%98-%EC%99%84%EB%B2%BD%ED%95%9C-%EA%B0%80%EC%9D%B4%EB%93%9C-A-Mostly-Complete-Guide-to-React-Rendering-Behavior)