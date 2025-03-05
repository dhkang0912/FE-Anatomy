- [리액트 렌더링 과정](#리액트-렌더링-과정)
  - [1. 개요 및 미리보기 (이정환 - React.js의 렌더링 방식 살펴보기)](#1-개요-및-미리보기-이정환---reactjs의-렌더링-방식-살펴보기)
    - [1-1. 웹 브라우저 동작 방식](#1-1-웹-브라우저-동작-방식)
    - [1-2. 리액트의 렌더링 프로세스](#1-2-리액트의-렌더링-프로세스)
  - [2. 상세 (\[번역\] 리액트 렌더링 동작의 (거의) 완벽한 가이드 \[A (Mostly) Complete Guide to React Rendering Behavior\])](#2-상세-번역-리액트-렌더링-동작의-거의-완벽한-가이드-a-mostly-complete-guide-to-react-rendering-behavior)
    - [2-1. 리액트의 렌더링이란 무엇인가?](#2-1-리액트의-렌더링이란-무엇인가)
      - [(1) 전체적인 렌더링 과정](#1-전체적인-렌더링-과정)
    - [2-2. 렌더와 커밋 단계](#2-2-렌더와-커밋-단계)
    - [2-3. 리액트 전체 렌더링 과정 요약](#2-3-리액트-전체-렌더링-과정-요약)
      - [(1) React Element (리액트 엘리먼트)](#1-react-element-리액트-엘리먼트)
      - [(2) 컴포넌트 트리 (Component Tree)](#2-컴포넌트-트리-component-tree)
      - [(3) Virtual DOM (가상 DOM)](#3-virtual-dom-가상-dom)
      - [(4) Fiber 트리 (Fiber Tree, React 16버전 이후)](#4-fiber-트리-fiber-tree-react-16버전-이후)
      - [(5) 실제 DOM 업데이터 (Commit 단계)](#5-실제-dom-업데이터-commit-단계)
      - [(6) 한 줄 요약](#6-한-줄-요약)
    - [2-4. React는 렌더를 어떻게 다룰까?](#2-4-react는-렌더를-어떻게-다룰까)
      - [1. 렌더링 순서 만들기](#1-렌더링-순서-만들기)
      - [2. 표준적인 렌더 동작](#2-표준적인-렌더-동작)
      - [3. React의 렌더링 규칙](#3-react의-렌더링-규칙)
      - [4. 컴포넌트 메타데이터와 Fibers](#4-컴포넌트-메타데이터와-fibers)
      - [5. 컴포넌트 타입과 재조정 (Reconciliation)](#5-컴포넌트-타입과-재조정-reconciliation)
      - [6. keys와 재조정 (Reconcilation)](#6-keys와-재조정-reconcilation)
      - [7. 렌더링 배치(Batching)와 타이밍](#7-렌더링-배치batching와-타이밍)
      - [8. 렌더 동작의 엣지 케이스](#8-렌더-동작의-엣지-케이스)
  - [Q\&A](#qa)
    - [Q1. React의 렌더링 과정에 대해 설명해주세요.](#q1-react의-렌더링-과정에-대해-설명해주세요)
    - [Q2. React는 상태 업데이트(setState())를 어떻게 최적화하나요?](#q2-react는-상태-업데이트setstate를-어떻게-최적화하나요)
    - [Q3. React에서 key의 역할은 무엇이며, 왜 중요할까요?](#q3-react에서-key의-역할은-무엇이며-왜-중요할까요)
    - [Q4. 컴포넌트에서 useState 값이 유지되는 조건은 무엇인가요?](#q4-컴포넌트에서-usestate-값이-유지되는-조건은-무엇인가요)
    - [Q5. 불필요한 리렌더링을 최적화하는 방법은 무엇인가요?](#q5-불필요한-리렌더링을-최적화하는-방법은-무엇인가요)
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

- 전체 컴포넌트 트리에서 렌더 결과물을 모두 수집하면 **재조정(Reconciliation)** 을 진행
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

<details>
<summary>
&nbsp <mark>리액트 전체 렌더링 과정 요약</mark>
</summary>

### 2-3. 리액트 전체 렌더링 과정 요약
#### (1) React Element (리액트 엘리먼트)
- 리액트에서 `가장 기본적인 단위`
- 컴포넌트가 반환하는 UI의 형태를 표현하는 단순한 불변 **객체**
  - 단순한 객체로 부모-자식 관계 같은 구조적 정보가 없음
  - JSX를 JavaScript 객체로 변환한 것
- 어떤 요소를 렌더링할 것인지 확인한 것
  - UI를 어떻게 그릴지 설명하는 **설계도**와 같은 역할
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
  > `

#### (2) 컴포넌트 트리 (Component Tree)
- 리액트의 컴포넌트 계층 구조 `(부모-자식 관계 표현)`
  - React Element들을 부모-자식 관계로 정리한 구조
- 리액트는 이를 기반으로 Virtual DOM을 생성 
- 컴포넌트 트리는 컴포넌트 간의 관계만을 표현하며 UI의 구체적인 정보는 없음
  - State나 DOM 업데이트 정보는 없음

  > 예시
  > ```JS
  > function App() {
  > return (
  >   <div>
  >     <Header />
  >     <Main />
  >     <Footer />
  >   </div>
  > );
  > }
  > 
  > ```
  > 컴포넌트 트리 구조
  > ```JS
  >   App
  > ├── Header
  > ├── Main
  > └── Footer
  > 
  > ```

#### (3) Virtual DOM (가상 DOM)
- Virtual DOM : React의 UI 상태를 나타내는 가상의 DOM 
  - UI를 그릴 요소 + Props 정보
- React Element를 트리 구조로 합성하여 `UI 구조`를 표현한 것
- 각 노드는 React Element를 포함하여 부모-자식 관계가 정의됨 (단순 UI)
  - 컴포넌트 트리의 부모-자식 관계가 반영됨 

#### (4) Fiber 트리 (Fiber Tree, React 16버전 이후)
- 리액트가 `렌더링을 최적화`하기 위해 사용
- 각 컴포넌트의 상태(State), 업데이트 정보, 부모-자식 관계 등을 포함
- 리액트는 `Fiber를 이용해 변경 사항을 감지`하고 최적화된 렌더링을 수행
- Fiber 트리에서 추가된 정보
  - 각 컴포넌트의 상태(State), Props, 부모-자식 관계
  - 이전 렌더링과 비교하기 위한 "alternate" (이전 Fiber 트리 노드)
  - 최적화된 렌더링을 위한 내부 포인터 구조 (부모, 형제, 자식 요소 연결)

> **Virtual DOM → Fiber 트리 변환** 예시
> ```JS
> {
>  type: "div",
>  props: { children: [...] },
>  stateNode: instance, // 클래스 컴포넌트의 경우 인스턴스 저장
>  memoizedState: { count: 0 }, // useState의 값 저장
>  child: firstChildFiber, // 첫 번째 자식 요소
>  sibling: nextSiblingFiber, // 형제 요소
>  return: parentFiber, // 부모 요소
>  alternate: previousFiber, // 이전 렌더링의 Fiber (비교용)
> }
>```

#### (5) 실제 DOM 업데이터 (Commit 단계)
- Fiber 트리를 기반으로 변경 사항을 감지하고 실제 DOM에 반영
- 최소한의 변경만 적용하여 성능 최적화
> Fiber 트리에서 **변경사항 감지**
> ```JS
>{
>  type: "p",
>  memoizedState: { count: 1 }, // 새로운 상태
>  alternate: { count: 0 }, // 이전 상태
>  effectTag: "UPDATE", // 변경 발생
>}
>```
- `effectTag`가 있는 Fiber만 실제 DOM에 반영

#### (6) 한 줄 요약
- React Element → 컴포넌트 트리 → Virtual DOM → Fiber 트리 → 실제 DOM 업데이트 순서로 렌더링한다.
  - 렌더 과정 : React Element → 컴포넌트 트리 → Virtual DOM → Fiber 트리
  - 커밋 과정 : 실제 DOM 업데이트

</details>

### 2-4. React는 렌더를 어떻게 다룰까?
#### 1. 렌더링 순서 만들기
- 최초의 렌더가 끝난 이후 React가 리렌더링을 Queue에 넣도록 하는 방법 (리렌더링을 발생시키는 방법)
  1. 클래스형 컴포넌트 
     - `this.setState()`
     - `this.forceUpdate()`
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
> 즉, 외부 상태(현재 컴포넌트 기준)를 변경하지 않고, 같은 입력이 주어지면 **항상 같은 결과**를 반환해야 함.
- 렌더링 로직은 다음과 같은 행위를 해서는 안됨
  - 현재 존재하는 변수와 객체를 변경하는 행위 => 외부 변수 변경 금지
  - `Math.random()` 또는 `Date.now()` 등의 랜덤 값을 만들어내는 행위 => 매번 다른 결과가 나와 불필요한 리렌더링이 발생할 수 있음
  - 네트워크 요청을 만들어내는 행위 => 렌더링할 때마다 요청이 갈 수 있음
  - 상태 업데이트를 Queue에 넣은 행위 => 렌더링 과정 중 상태 업데이트가 있으면 무한 루프에 빠질 수 있음
- 다음과 같은 행위는 괜찮을 수 있음
  - 렌더링 도중 새롭게 만들어진 객체를 변경하는 행위 => 외부 상태를 건드리지 않음
  - 에러를 발생시키는 행위
  - 캐싱된 값처럼 아직 만들어지지 않은 데이터를 "Lazy 초기화"하는 행위

#### 4. 컴포넌트 메타데이터와 Fibers
- Fiber (피버) : 각 컴포넌트를 관리하는 내부 객체로 컴포넌트의 상태, 업데이트 정보, 부모-자식 관계를 연결 리스트로 관리함 (리액트 16부터 사용됨)
  - Fiber는 리액트의 `핵심 렌더링 엔진`
    - 어떻게 화면을 그릴지 결정하고 실제 화면에 반영하는 역할
    - 렌더 단계 : 변경 사항을 계산하는 과정 (Virtual DOM과 비교)
    - 커밋 단계 : 실제 DOM을 업데이트하는 과정

> **Fiber 트리 구조 예시**
> ```JS
>{
>  type: "div",
>  props: { children: [...] },
>   stateNode: instance, // 클래스 컴포넌트의 경우 인스턴스 저장
>   memoizedState: { count: 0 }, // useState의 값 저장
>   child: firstChildFiber, // 첫 번째 자식 요소
>   sibling: nextSiblingFiber, // 형제 요소
>   return: parentFiber, // 부모 요소
>   alternate: previousFiber, // 이전 렌더링의 Fiber (비교용)
> }
> 
> ```

<br>

- Fiber 객체(Fiber Node) 정보
  - 각 Fiber 객체는 `컴포넌트의 메타데이터 (정보)`를 저장함
  - 컴포넌트의 상태와 업데이트 방식을 관리하는 데이터 구조

    | 정보                       | 설명                                                          |
    | :------------------------- | :------------------------------------------------------------ |
    | 컴포넌트 유형              | 이 Fiber가 어떤 컴포넌트인지 (함수형, 클래스형, HTML 태그 등) |
    | Props & State              | 현재 이 컴포넌트의 Props와 State 값                           |
    | 부모-형제-자식 포인터	트리 | 구조를 유지하기 위해 부모, 형제, 자식 Fiber를 가리키는 포인터 |
    | 업데이트 정보              | 이 컴포넌트에서 발생한 변경 사항 (리렌더링 여부)              |
    | 이전 Fiber 정보            | 이전 렌더링 시 사용했던 Fiber 객체 (변경 사항 비교용)         |

<br>

- Fiber의 동작 방식
  - React는 컴포넌트를 렌더링할 때 Fiber 트리를 탐색하면서 업데이트를 수행
  1. 렌더링 단계 (Render Phase)
     - React가 새로운 Virtual DOM을 생성하고 `Fiber 트리를 순회하며 변경 사항을 계산`
     - 새로운 Fiber 트리를 만들고 기존 Fiber 트리와 비교하여 업데이트가 필요한 부분을 찾음
  2. 커밋 단계 (Commit Phase)
     - 변경된 Fiber만 실제 DOM에 반영
     - React는 이전 Fiber와 새로운 Fiber를 비교하여 `변경된 부분만 업데이트`

<br>

- Fiber와 클래스형 & 함수형 컴포넌트 차이
  - 클래스형 컴포넌트에서 Fiber 동작 방식
    - 클래스형 컴포넌트는 인스턴스(this)를 직접 생성해서 관리함
    - new Component(props)를 실행하여 컴포넌트의 인스턴스를 만들고 Fiber 객체에 저장
    > 예시
    ```JS
    const instance = new MyComponent(props); // 클래스 인스턴스 생성
    fiberNode.stateNode = instance; // Fiber가 컴포넌트 인스턴스를 저장
    ```
  - 함수형 컴포넌트에서 Fiber 동작 방식
    - 함수형 컴포넌트는 별도의 인스턴스를 만들지 않음!
    - 대신, 그냥 `함수 실행(YourComponent(props))을 통해 결과를 반환`
    - React는 Fiber 트리에 이 함수의 렌더링 결과를 저장
    > 예시
    ```JS
    fiberNode.stateNode = null; // 함수형 컴포넌트는 인스턴스를 가지지 않음

    ```
    - 대신, Fiber가 훅(useState, useReducer 등)의 상태를 저장함
    - 훅은 Fiber 내부의 "연결 리스트"로 관리됨

<br>

- Fiber와 React Hooks (훅 관리)
  - React의 훅(useState, useEffect 등)은 Fiber 내부에서 `연결 리스트`로 관리됨
  - React가 `함수형 컴포넌트를 렌더링할 때, 이전 Fiber에서 훅 리스트를 가져와 상태를 유지함`

#### 5. 컴포넌트 타입과 재조정 (Reconciliation)
- 리액트는 기존 존재하는 컴포넌트 트리와 DOM 구조를 최대한 재활용해서 효율적으로 리렌더링하려고 함
  - 같은 위치에서 같은 유형(Component Type)의 컴포넌트가 유지되는 이유는 Fiber의 비교 방식 덕분

<br>

1. `컴포넌트의 타입(Type)`을 기준으로 비교 (Diffing)
  - 컴포넌트를 재사용할지, 삭제를 할지 결정할 때 컴포넌트 타입을 우선시 여김
  - 같은 위치에서 같은 유형의 컴포넌트가 있다면 유지 
    - 같은 위치에 있고 같은 유형이지만 Props만 업데이트한 경우 -> Props만 업데이트 
    > 같은 컴포넌트 유지되는 예시
      ```JS
      // 같은 컴포넌트 유지
      function MyComponent({ text }) {
        return <p>{text}</p>;
      }

      function App({ isVisible }) {
        return (
          <div>
            {isVisible ? <MyComponent text="Hello" /> : <MyComponent text="World" />}
          </div>
        );
      }

      // 이전 렌더링
      <MyComponent text="Hello" />

      // 이후 렌더링
      <MyComponent text="World" />

      ```
      - `MyComponent`가 같은 위치에 있고 같은 유형이므로 Props만 업데이트
    
2. 컴포넌트 타입이 바뀌면 기존 트리를 파괴하고 새로 만듦
  - 만약 기존 위치에 완전히 다른 컴포넌트가 렌더링되면 React는 트리를 재사용하지 않음
   - 대신 기존 컴포넌트를 `제거`하고 새로운 컴포넌트를 `생성`함.
    
  > 다른 컴포넌트로 변경 → 트리 삭제 & 새로 생성되는 예시

  ```JS
  function ComponentA() {
    return <p>A Component</p>;
  }

  function ComponentB() {
    return <p>B Component</p>;
  }

  function App({ isToggled }) {
    return (
      <div>
        {isToggled ? <ComponentA /> : <ComponentB />}
      </div>
    );
  }

  // 이전 렌더링
  <ComponentA />

  // 이후 렌더링 (isToggled 변경)
  <ComponentB />

  ```
  - ComponentA와 ComponentB는 다른 타입이므로 React는 ComponentA를 삭제하고, ComponentB를 새로 만듦
  - React가 렌더링 최적화를 위해 같은 위치에서 같은 컴포넌트 유형(Type)이 유지됨

3. 클래스형 컴포넌트의 경우 실제 인스턴스를 유지함
  - 클래스형 컴포넌트에서는 인스턴스(객체)가 생성되므로, 같은 위치에서 같은 컴포넌트가 있으면 인스턴스를 유지함
  > 클래스형 컴포넌트 인스턴스 유지 예시
  ```JS
  class MyComponent extends React.Component {
    componentDidMount() {
      console.log("Mounted!");
    }

    componentDidUpdate() {
      console.log("Updated!");
    }

    render() {
      return <p>{this.props.text}</p>;
    }
  }

  function App({ isVisible }) {
    return (
      <div>
        {isVisible ? <MyComponent text="Hello" /> : <MyComponent text="World" />}
      </div>
    );
  }

  // 처음 마운트 시 콘솔 출력
  Mounted!

  // 이후 Props 변경 시 (text 변경)
  Updated!
  ```
  - 같은 위치에서 같은 타입이면 `this 인스턴스`가 유지되므로 `componentDidUpdate`가 실행됨
  - 클래스형 컴포넌트의 경우 같은 위치에서 같은 유형이면 인스턴스를 유지하며, Props만 변경

4. 함수형 컴포넌트는 인스턴스가 없지만, 같은 위치면 상태를 유지함
   - 함수형 컴포넌트는 클래스형처럼 this 인스턴스가 없지만, 같은 위치라면 내부 상태`(useState)`를 유지
  > 같은 위치라면 `useState` 값 유지 예시
  ```JS
  function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
  }

  function App({ showCounter }) {
    return (
      <div>
        {showCounter ? <Counter /> : <p>Hidden</p>}
      </div>
    );
  }

  // showCounter가 true일 때
  <Counter /> (count = 0)

  // 사용자가 버튼을 클릭하여 count 증가 → (count = 1)
  // 이후 showCounter를 false로 변경했다가 다시 true로 변경
  <Counter /> (count = 1)  // 같은 위치라서 count 상태 유지됨!

  ```
  - 같은 위치에 같은 유형(Type)이라면 변경된 useState 값도 유지됨 
  - 즉 함수형 컴포넌트는 인스턴스가 없지만 같은 위치에서 렌더링되면 변경된 useState 값도 유지됨!

#### 6. keys와 재조정 (Reconcilation)
- 리액트가 **컴포넌트 인스턴스를 식별**하는 방법
  - **컴포넌트 타입** : 리액트에서 해당 요소가 어떤 컴포넌트인지 나타내는 값
    - React Element, Fiber에 `type`으로 저장됨
  - **key** : 의사-Prop(pseudo-prop)으로 특정 인스턴스를 식별하기 위한 고유 식별자
    - 컴포넌트 타입이 같더라도 key 값이 다르면 완전히 다른 컴포넌트로 간주하고 새로운 인스턴스를 생성
    - 의사-Prop (pseduo-prop) : 컴포넌트에서 일반적인 props처럼 사용할 수 없지만 리액트 내부에서 특별한 용도로 관리되는 Prop

<br>

- **Key의 주요 활용 : 배열 렌더링**
  - 배열을 `재정렬`하거나 `요소를 추가, 삭제`하는 등의 방식 `(인덱스가 변경됨)`으로 변경될 수 있는 배열을 렌더링할 경우 key가 중요
  - `key`에 따라 `인스턴스를 식별`하기 때문에 key에 따라 리렌더링 여부가 결정됨
    - 따라서 불필요한 리렌더링을 막기 위해 key는 가능하다면 `고유한 값`으로 사용하여 인스턴스를 구분하여 `특정한 부분만 리렌더링` 될 수 있게 해야함
    -  배열의 인덱스는 정말 최후의 수단으로 활용해야함


<br>

- 예시
  - **인덱스**를 **key**로 설정
    ```jsx
    function List({ items }) {
      return (
        <ul>
          {items.map((item, index) => (
            <li key={index}>{item.text}</li> // ❌ 인덱스를 key로 사용하면 안 됨!
          ))}
        </ul>
      );
    }
    ```
    - 배열의 `첫번째 요소가 삭제`되는 경우
      1. 기존 요소들의 `index가 한 칸씩 당겨짐`
      2. key=index를 사용하여 요소를 식별하기 때문에 `기존 요소를 제대로 찾지 못하고` 새로운 인스턴스로 인식하거나 잘못 매칭하여 재사용할 수 있음
      3. 잘못된 리렌더링이 발생하여 기존 요소가 새로운 내용으로 덮어씌워질 수 있음
      4. 동일한 내용으로 적혀있는 다른 배열의 인스턴스들도 새로운 인스턴스로 인식하여 `불필요한 리렌더링`이 됨
  - **고유한 id**를 **key**로 설정
    ```jsx
    function List({ items }) {
      return (
        <ul>
          {items.map((item) => (
            <li key={item.id}>{item.text}</li> // ✅ id를 key로 사용!
          ))}
        </ul>
      );
    }

    ```
    - 배열의 요소가 추가, 삭제, 재정렬되더라도 id를 기준으로 요소를 정확하게 식별할 수 있음
  
#### 7. 렌더링 배치(Batching)와 타이밍
> - **렌더링 배치 (Rendering Batching)** : 리액트의 `상태 업데이트`가 주로 `비동기적으로(Batch processing)`으로 발생하게 되고 `여러 개의 setState() 호출을 하나의 렌더링 패스`로 묶어서 처리하는 `최적화`를 수행하는 것

<br>

- **리액트의 상태 업데이트 흐름**
  1. setState()가 호출되면 새로운 렌더링 패스가 시작됨
  2. 여러 개의 setState() 호출을 하나의 렌더링 패스로 묶어(Batching) 최적화함
  3. 렌더링이 완료되면 최종적으로 변경된 상태를 DOM에 반영하는 커밋 단계 (Commit Phase)를 실행

  - 리액트의 `상태 업데이트`는 `비동기적으로 발생`할 가능성이 있으며 상태 업데이트를 즉시 반영하지 않고 일정한 규칙에 따라 배치 처리함
  - `렌더링 배치`를 통해서 `불필요한 렌더링을 방지`하고 성능을 최적화

<br>

- 렌더링 배치 예시 : 여러 개의 setState() 호출이 배치 처리되는 경우
  ```jsx
  function Counter() {
    const [count, setCount] = useState(0);

    const handleClick = () => {
      setCount(count + 1);
      setCount(count + 2);
      setCount(count + 3);
    };

    console.log("렌더링 발생!");

    return <button onClick={handleClick}>Count: {count}</button>;
  }

  ```
  - setState()를 세번 호출했지만 리액트는 한번의 렌더링 패스로 처리함
    - 상태는` count + 3`으로 업데이트되며, `console.log("렌더링 발생!")`은 `한번`만 출력됨

<br>

- **배치 처리가 되지 않는 경우**
  - `setTimeout()`, `Promise`, `async/await` 같은 비동기 코드에서는 배치 처리가 자동으로 적용되지 않음
  - `await` 이후의 `setState()` 호출은 새로운 이벤트 루프에서 실행되기 때문에 배치되지 않음
  - 예시 : 비동기 코드에서 배치 처리가 되지 않는 경우
    ```jsx
    function Counter() {
      const [count, setCount] = useState(0);

      const handleClick = async () => {
        setCount(0);
        setCount(1); // ✅ 같은 이벤트 루프 내에서 실행 → 배치 처리됨

        await fetchSomeData(); // ✅ 비동기 코드 실행 (이벤트 루프가 넘어감)

        setCount(2);
        setCount(3); // 🚨 새로운 이벤트 루프에서 실행 → 배치되지 않음 (두 번 렌더링 발생)
      };

      console.log("렌더링 발생!");

      return <button onClick={handleClick}>Count: {count}</button>;
    }

    ```
    - `setState(0)과 setState(1)은 배치 처리`되지만,` await fetchSomeData() 이후` 실행된 `setState(2)와 setState(3)는 각각 별도`의 렌더링을 발생

<br>

- **리액트의 배치 처리 방식**
  - `unstable_batchedUpdates()`라는 내부 함수를 사용하여 배치 업데이트 수행
    - 리액트 내부에서 실행되며 이벤트 핸들러 내의 상태 업데이트를 자동으로 배치 처리함
  - 예시 :  React가 내부적으로 실행하는 코드 (의사 코드)
    ```jsx
    function internalHandleEvent(e) {
      const userProvidedEventHandler = findEventHandler(e);
      
      let batchedUpdates = [];
      
      unstable_batchedUpdates(() => {
        userProvidedEventHandler(e); // 🚀 모든 setState() 호출을 배치로 처리
      });
      
      renderWithQueuedStateUpdates(batchedUpdates); // 🚀 단일 렌더링 패스로 실행, 설명을 위한 의사코드
    }

    ```
    - `unstable_batchedUpdates()` 함수를 사용하여 배치 업데이트를 수행하며 모든 상태 업데이트를 추적하고 한번에 처리 => `자동으로 상태 업데이트를 일괄 처리`
    - `renderWithQueuedStateUpdates()` 를 통해 `unstable_batchedUpdates()`로 모아놓은 setState() 호출들을 실행하여 한번에 렌더링 패스로 처리

<br>

- **Concurrent Mode와의 관계**
  - 리액트는 이벤트 핸들러에서만 배치 처리를 자동으로 수행하지만 Concurrent Mode에서는 모든 setState() 호출을 항상 배치 처리함
  - 비동기 코드 (`await`, `Promise`) 이후의 상태 업데이트도 자동으로 배치 처리함

#### 8. 렌더 동작의 엣지 케이스
> **엣지 케이스** : 리액트의 렌더링 과정에서 일반적인 흐름과 다른 특별한 상황이 발생하는 것
>
> 이해 못하고 있으면 예상치 못한 버그가 발생할 수 있음

**(1) `<StrictMode>`에서 컴포넌트가 두번 렌더링됨**
- **StrictMode** : 리액트의 `개발 중`에만 활성화되는 기능으로 컴포넌트가 예상대로 동작하는지 검사하는 역할 
  - 실제 DOM 업데이트는 한번만 이루어짐 (커밋)
  - React DevTools, useEffect()를 활용해서 렌더링 횟수를 정확히 확인할 수 있음

**(2) 렌더링 도중 상태 업데이트 시 에러 발생 가능**
- 렌더링 과정에서 상태 업데이트 (`setState()`)를 직접 호출하면 외부 상태를 변경하게 되면서 무한 루프 발생 가능
  - 리액트는 일정 횟수 (기본 50회) 이상 리렌더링이 발생하면 오류를 발생시켜 실행을 중지함
- 예시 : 렌더링 도중 상태 업데이트 (잘못된 코드)
  ```jsx
  function MyComponent() {
    const [count, setCount] = useState(0);

    console.log("렌더링됨!");

    setCount(count + 1); // ❌ 렌더링 도중 상태 업데이트 (무한 루프 발생 가능)

    return <div>Count: {count}</div>;
  }

  ```
<br>

- **예외**: 함수형 컴포넌트에서는 특정 상황에서 상태 업데이트가 허용됨
  - 함수형 컴포넌트의 경우 특정 조건에서 허용됨
  - 클래스형 컴포넌트의 `getDerivedStateFromProps()`와 유사한 개념
  - 예시 : 조건부로 상태 업데이트가 허용되는 경우
    ```jsx
    function MyComponent({ value }) {
      const [state, setState] = useState(value);

      if (state !== value) {
        setState(value); // ✅ 조건이 맞으면 상태 업데이트 허용
      }

      return <div>{state}</div>;
    }

    ```
    - 매번 렌더링될 때마다가 실행되는 것이 아니라 허용
  
**(3) 무한 루프 방지: React는 50회 이상 리렌더링되면 강제로 중단**
- 렌더링 도중 상태를 변경하면 `무한 루프`에 빠질 수 있음.
- React는 `50회 이상 연속`으로 렌더링되면 `오류`를 발생시키고 실행을 멈춤.

</details>

## Q&A
### Q1. React의 렌더링 과정에 대해 설명해주세요.

A1. React의 렌더링 과정은 크게 **렌더 단계(Render Phase)** 와 **커밋 단계(Commit Phase)** 로 나뉩니다.

**렌더 단계**에서는 React가 컴포넌트를 실행하여 Virtual DOM을 생성하고, 이전 Virtual DOM과 비교하여 변경 사항을 계산합니다. 이 과정에서는 실제 DOM에 아무런 영향을 주지 않고, 단순히 "이전 상태와 비교하여 변경된 부분이 있는지"를 판별하는 단계입니다.

이후 **커밋 단계**에서는 렌더 단계에서 계산된 변경 사항을 실제 DOM에 반영하는 작업이 이루어집니다. 이때, ref 업데이트, componentDidMount, componentDidUpdate, useLayoutEffect 같은 생명주기 메서드가 실행되고 브라우저 화면에 최종적인 UI가 반영됩니다.

이렇게 두 단계로 나누는 이유는 성능 최적화를 위해서입니다. 즉, 불필요한 DOM 업데이트를 최소화하고, 변경된 부분만 효율적으로 적용하여 렌더링 성능을 높일 수 있습니다.

### Q2. React는 상태 업데이트(setState())를 어떻게 최적화하나요?
A2. React는 여러 개의 setState() 호출을 하나의 렌더링 패스로 묶어서 실행하는 렌더링 배치(Batching) 기법을 사용하여 성능을 최적화합니다.

예를 들어, 하나의 이벤트 핸들러에서 여러 번 setState()를 호출한다고 해도, React는 이를 한 번의 렌더링으로 묶어 처리합니다. 이를 통해 불필요한 렌더링을 방지하고, 렌더링 성능을 최적화할 수 있습니다.

하지만 모든 경우에 자동으로 배치 처리가 적용되는 것은 아닙니다. 예를 들어, setTimeout(), Promise, async/await 같은 비동기 코드 내부에서는 setState() 호출이 각각 별도의 렌더링을 발생시킬 수 있습니다.

이를 해결하기 위해 React 내부에서는 unstable_batchedUpdates()라는 API를 활용하여 배치 업데이트를 수행하며, Concurrent Mode가 활성화된 경우에는 모든 상태 업데이트가 자동으로 배치 처리됩니다.

### Q3. React에서 key의 역할은 무엇이며, 왜 중요할까요?
A3. React에서 key는 리스트 렌더링 시 각 요소를 식별하는 고유한 값입니다.

React는 리스트에서 요소를 비교할 때, 기본적으로 컴포넌트의 타입과 key 값을 비교하여 기존 요소를 재사용할지, 새로 생성할지를 결정합니다.

예를 들어, 배열의 요소를 추가하거나 삭제하는 경우, key가 없거나 인덱스를 key로 사용하면 React는 예상과 다르게 요소를 재사용할 수도 있고, 불필요한 리렌더링이 발생할 수도 있습니다. 따라서 리스트의 요소를 렌더링할 때는 고유한 ID 값을 key로 사용하는 것이 가장 좋습니다.

### Q4. 컴포넌트에서 useState 값이 유지되는 조건은 무엇인가요?
A4. React에서 useState 값이 유지되려면, 같은 위치에서 같은 타입(Type)의 컴포넌트가 렌더링되어야 합니다. React는 렌더링할 때 컴포넌트의 위치와 타입을 기준으로 비교하고, 같은 타입이라면 기존 상태를 유지합니다. 그러나 key 값이 변경되거나 컴포넌트 타입이 바뀌면 React는 새로운 인스턴스로 인식하여 상태(useState 또는 this.state)가 초기화됩니다.

클래스형 컴포넌트의 경우 this.state는 같은 인스턴스를 유지하는 한 계속 유지됩니다. 하지만 함수형 컴포넌트는 인스턴스가 없고, 대신 React의 Fiber 구조에서 useState의 상태를 저장하며 관리합니다. 따라서 같은 위치에서 같은 함수형 컴포넌트가 렌더링되면 React가 이전 상태를 유지하지만, key가 변경되거나 타입이 바뀌면 새로운 상태로 초기화됩니다.

### Q5. 불필요한 리렌더링을 최적화하는 방법은 무엇인가요?
A5. React에서는 상태가 변경되면 해당 컴포넌트와 그 자식 컴포넌트가 리렌더링됩니다. 하지만 불필요한 리렌더링이 계속 발생하면 성능이 저하될 수 있기 때문에 이를 방지하는 여러 최적화 방법이 존재합니다.

첫 번째 방법은 React.memo를 사용하는 것입니다. React.memo는 함수형 컴포넌트에서 사용되며, props가 변경되지 않는 한 이전 렌더링 결과를 재사용하여 불필요한 렌더링을 방지합니다. 기본적으로 부모 컴포넌트가 리렌더링되면 자식 컴포넌트도 함께 리렌더링되지만, React.memo를 적용하면 자식 컴포넌트는 동일한 props가 전달될 경우 다시 렌더링되지 않습니다. 예를 들어, 다음과 같이 React.memo를 사용하면, 부모가 리렌더링되더라도 MyComponent는 props가 변경되지 않는 한 다시 렌더링되지 않습니다.

두 번째 방법은 useMemo를 활용하는 것입니다. useMemo는 복잡한 연산이 포함된 값을 메모이제이션하여 불필요한 연산을 줄일 수 있습니다. 예를 들어, 수천 개의 데이터를 필터링하는 연산이 있다고 가정했을 때, 매번 렌더링될 때마다 이 연산이 수행되면 성능이 저하될 수 있습니다. useMemo를 사용하면 특정 의존성 값이 변경되지 않는 한 이전 결과를 캐싱하여 불필요한 연산을 방지할 수 있습니다.

세 번째 방법은 useCallback을 활용하는 것입니다. useCallback은 함수의 참조를 메모이제이션하여 불필요한 재생성을 방지합니다. React에서는 함수도 객체처럼 참조형 데이터이기 때문에, 부모 컴포넌트가 리렌더링될 때마다 함수가 다시 생성되면 자식 컴포넌트가 불필요하게 리렌더링될 수 있습니다. 이를 방지하기 위해 useCallback을 사용하면, 특정 의존성이 변경되지 않는 한 동일한 함수 참조를 유지할 수 있습니다.

네 번째 방법은 useReducer를 사용하는 것입니다. useState를 사용하면 상태가 변경될 때마다 컴포넌트가 리렌더링되지만, useReducer를 활용하면 상태 변경 로직을 분리하여 불필요한 렌더링을 줄일 수 있습니다. 특히, 복잡한 상태 관리가 필요한 경우 useReducer를 사용하면 더 나은 성능 최적화가 가능합니다.

마지막으로, useRef를 활용하는 방법이 있습니다. useRef는 상태를 저장하지만 값이 변경되더라도 리렌더링을 발생시키지 않습니다. 예를 들어, 어떤 값이 렌더링과는 무관하게 유지되어야 하는 경우 useRef를 사용할 수 있습니다.


## 참고 자료
1. [React.js의 렌더링 방식 살펴보기 - 이정환 | 2023 NE(O)RDINARY CONFERENCE
](https://www.youtube.com/watch?v=N7qlk_GQRJU)
2. [[번역] 리액트 렌더링 동작의 (거의) 완벽한 가이드 [A (Mostly) Complete Guide to React Rendering Behavior]
](https://velog.io/@arthur/%EB%B2%88%EC%97%AD-%EB%A6%AC%EC%95%A1%ED%8A%B8-%EB%A0%8C%EB%8D%94%EB%A7%81-%EB%8F%99%EC%9E%91%EC%9D%98-%EA%B1%B0%EC%9D%98-%EC%99%84%EB%B2%BD%ED%95%9C-%EA%B0%80%EC%9D%B4%EB%93%9C-A-Mostly-Complete-Guide-to-React-Rendering-Behavior)