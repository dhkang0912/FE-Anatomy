# React 렌더링 과정
- [React 렌더링 과정](#react-렌더링-과정)
  - [React 렌더링 전 알아야 할 개념](#react-렌더링-전-알아야-할-개념)
    - [1. JSX](#1-jsx)
      - [1-1. JSX란?](#1-1-jsx란)
      - [1-2. JSX의 구성](#1-2-jsx의-구성)
      - [1-3. JSX가 자바스크립트로 변환되는 과정](#1-3-jsx가-자바스크립트로-변환되는-과정)
    - [2. 가상 DOM과 리액트 파이버](#2-가상-dom과-리액트-파이버)
      - [2-1. 브라우저 렌더링 과정](#2-1-브라우저-렌더링-과정)
      - [2-2. 가상 DOM 탄생 배경](#2-2-가상-dom-탄생-배경)
      - [2-3. 가상 DOM을 위한 아키텍처, 리액트 파이버(React Fiber)](#2-3-가상-dom을-위한-아키텍처-리액트-파이버react-fiber)
    - [3. 클래식 컴포넌트와 함수 컴포넌트](#3-클래식-컴포넌트와-함수-컴포넌트)
      - [3-1. 클래식 컴포넌트](#3-1-클래식-컴포넌트)
      - [3-2. 함수 컴포넌트](#3-2-함수-컴포넌트)
      - [3-3. 클래스 컴포넌트와 함수 컴포넌트의 차이](#3-3-클래스-컴포넌트와-함수-컴포넌트의-차이)
  - [React 렌더링 과정](#react-렌더링-과정-1)
    - [1. React 렌더링이란?](#1-react-렌더링이란)
    - [2. React 렌더링의 시점](#2-react-렌더링의-시점)
    - [3. React 렌더링 프로세스](#3-react-렌더링-프로세스)
      - [3-1. 렌더 단계와 커밋 단계](#3-1-렌더-단계와-커밋-단계)
- [Q\&A](#qa)
  - [1. 리액트에서 index를 key값으로 사용하면 안 되는 이유는 무엇인가요?](#1-리액트에서-index를-key값으로-사용하면-안-되는-이유는-무엇인가요)
  - [2. 브라우저 렌더링과 리액트 렌더링의 차이점을 설명해주세요.](#2-브라우저-렌더링과-리액트-렌더링의-차이점을-설명해주세요)
  - [3. props와 state란 무엇인가요?](#3-props와-state란-무엇인가요)
  - [4. 리액트의 재조정 과정을 리액트 18버전 이전과 이후로 비교하여 설명해주세요.](#4-리액트의-재조정-과정을-리액트-18버전-이전과-이후로-비교하여-설명해주세요)
  - [5. 클래스 컴포넌트와 함수 컴포넌트의 차이점을 설명해주세요.](#5-클래스-컴포넌트와-함수-컴포넌트의-차이점을-설명해주세요)

## React 렌더링 전 알아야 할 개념
### 1. JSX
#### 1-1. JSX란?
- 페이스북(현 메타)에서 개발한 구문
- XML과 유사한 내장형 구문
- 자바스크립트 표준의 일부 아님, 리액트에 종속적이지 않음
- 목표 : 다양한 트랜스파일러에서 다양한 속성을 가진 트리 구조를 토큰화해 자바스크립트(ECMAScript)로 변환
  - HTML, XML 등 다른 구문으로도 확장 가능
  - 최대한 간결하고 친숙하게 작성할 수 있도록 설계

#### 1-2. JSX의 구성
**JSXElement**
- JSX를 구성하는 가장 기본 요소
- HTML의 요소와 비슷한 역할
- JSXElement가 되기 위한 조건 (3개 중 하나의 형태)
  - <span style="background-color:lightyellow">JSXOpeningElement - JSXClosingElement</span> : `<JSXElement></JSXElement>`, 시작요소와 종료요소가 같은 단계에 선언
  - <span style="background-color:lightyellow">JSXSelfClosingElement</span> : `<JSXElement/>`, 요소가 시작되고 스스로 종료(내부적으로 자식 포함 못 함)
  - <span style="background-color:lightyellow">JSXFragment</span> : `<></>`, 아무런 요소가 없는 형태(</>는 불가능)
- 리액트에서 HTML 구문 이외에 사용자가 컴포넌트를 만들어 사용할 때는 **반드시 대문자**로 시작해야함 -> HTML-사용자 태그명을 구분 짓기 위해 
- JSXElementName : JSXElement의 요소 이름으로 쓸 수 있는 것
  - <span style="background-color:lightyellow">JSXIdentifier</span> : JSX 내부에서 사용할 수 있는 식별자 ($, _, 나머지 특수 문자와 숫자는 시작 불가)
  - <span style="background-color:lightyellow">JSXNamespacedName</span> : 콜론(:)을 이용해 서로 다른 식별자 이어줄 수 있음 (두 개 이상 불가)
  - <span style="background-color:lightyellow">JSXMemberExpression</span> : 점(.)을 이용해 서로 다른 식별자 이어줄 수 있음 (:와 함께 사용 불가)

**JSXAttributes**
- JSXElement에 부여할 수 있는 속성
- 필수 아님
- <span style="background-color:lightyellow">JSXSpreadAttributes</span> : `{...AssignmentExpression}`, 자바스크립트의 전개 연산자와 동일한 역할
- <span style="background-color:lightyellow">JSXAttribute</span> : `<foo.bar foo:bar="O"></foo.bar>`, 속성을 나타내는 키-값 ("", '', {}, JSXElement, JSXFragment)

**JSXChildren**
- JSXElement의 자식 값
- <span style="background-color:lightyellow">JSXChild</span> : JSXChildren을 이루는 기본 단위, 0개 이상 가질 수 있음
  - JSXText({,<,>,} 제외한 문자열), JSXElement, JSXFragment, {JSXChildExpressin (optional)}
  
**JSXStrings**
- " ", ' ', JSXText
- HTML과 JSX 사이에 복사/붙여넣기를 쉽게 하기 위함
  - \는 자바스크립트에서 특수문자를 처리할 때 사용되어 `\`를 표현하려면 `\\`로 이스케이프해야 함

#### 1-3. JSX가 자바스크립트로 변환되는 과정
@babel/plugin-trasform-react-jsx 플러그인
- 리액트 17, 바벨 7.9.0 이후 버전은 자동 런타임으로 트랜스파일 가능
- JSXElement를 첫 번째 인수로 선언해 요소를 정의하고, 나머지 구성요소는 이후 인수로 넘겨주어 처리
- JSX 반환값은 React.createElement로 귀결됨

### 2. 가상 DOM과 리액트 파이버
#### 2-1. 브라우저 렌더링 과정
① 브라우저가 사용자가 요청한 주소를 방문해 HTML 파일 다운로드  
② 브라우저 렌더링 엔진은 HTML을 파싱해 DOM 노드로 구성된 트리(DOM) 생성  
    ②-1. 위 과정에서 CSS 파일 만나면 해당 CSS 파일 다운로드  
    ②-2. 브라우저 렌더링 엔진은 CSS 파싱해 CSS 노드로 구성된 트리(CSSOM) 생성  
③ DOM과 CSSOM을 결합해 Render Tree 생성(눈에 보이는 노드만)  
④ 각 노드에 대한 CSS 스타일 정보 적용   
    ④-1. 레이아웃 : 각 노드가 브라우저 화면의 어느 좌표에 나타나야 하는 지 계산  
    ④-2. 페인팅 : 레이아웃 단계를 거친 노드에 색상과 같은 스타일 적용. 노드가 레이어로 분리되었다면 이후 합성 과정 거침  
![alt text](브라우저렌더링과정.png)

#### 2-2. 가상 DOM 탄생 배경
웹페이지를 렌더링하는 과정은 매우 복잡하고 많은 비용이 드는데, 리플로우와 리페인팅 외에도 사용자와의 상호작용을 늘리거나 라우팅이 변경될 경우 DOM의 모든 변경 사항을 추적하는데에 많은 시간과 비용이 듦

가상 DOM : 값을 가지고 있는 UI(Value UI)
- 웹페이지가 표시해야 할 DOM을 일단 메모리에 저장하고 리액트가 실제 변경에 대한 준비가 완료되었을 때 실제 DOM에 반영

#### 2-3. 가상 DOM을 위한 아키텍처, 리액트 파이버(React Fiber)
**리액트 파이버(React Fiber)**
- 리액트에서 관리하는 자바스크립트 객체
    - 하나의 element에 하나 생성(1:1 관계)
        <details>
            <summary>tag : 1:1로 매칭된 정보 가지고 있음</summary>
            <div markdown="1">
            <p>- FunctionComponent<br/>    
            - ClassComponent<br/>
            - IndeterminateComponent<br/>
            - HostRoot / HostPotal / HostComponent (웹 div와 같은 요소) / HostText<br/>
            - Fragment<br/>
            - Mode<br/>
            - ContextConsumer / ContextProvider<br/>
            - ForwardRef<br/>
            - Profiler<br/>
            - SuspenseComponent / SuspenseListComponent<br/>
            - MemoComponent /SimpleMemoComponent<br/>
            - LazyComponent<br/>
            - IncompleteClassComponent<br/>
            - DehydratedFragment<br/>
            - ScopeComponent<br/>
            - OffscreenComponent<br/>
            - LegacyHiddenComponent<br/>
            - CacheComponent<br/>
            - TracingMarkerComponent</p>
            </div>
        </details>
- 가상 DOM을을 관리하는 라이브러리
- 파이버 재조정자(fiber reconciler)가 관리
  - 재조정(reconciliation) : 가상 DOM과 실제 DOM을 비교하는 작업(알고리즘)
      - 차이가 있다면 변경에 관련된 정보를 가진 파이버를 기준으로 화면에 렌더링 요청
      - 바로 처리하거나 스케줄링 => 유연한 처리
- 실행 시점 : state 변경, 생명주기 메서드 실행, DOM의 변경 필요 등

<details>
<summary>참고 - 과거 리액트의 조정 알고리즘 (스택) </summary>
<div markdown="1">
<span style="font-weight:bold">• 리액트 렌더링은 항상 동기식으로 작동</span> <br/>
  - 하나의 스택에 렌더링에 필요한 작업들이 쌓이면 빌 때까지 동기적으로 작업 <br/>
  - 자바스크립트는 싱글 스레드이기 때문에 작업 중단 불가 <br/>
  - 렌더링 과정이 길어지면 애플리케이션 성능 저하와 브라우저의 다른 작업을 지연시킬 가능성 있음 <br/>
  - 만약 A 컴포넌트 렌더링 작업이 무거워 상대적으로 빠른 B라도 렌더링해 보여줄 수 있다면? <br/>
<br/>
<span style="font-weight:bold">• 리액트 18에서 동시성 렌더링 도입</span> <br/>
  - 의도된 우선순위로 컴포넌트를 렌더링해 최적화할 수 있는 비동기 렌더링 <br/>
  - 렌더 단계가 비동기로 작동해 특정 렌더링의 우선순위를 낮추거나, 필요하다면 중단하거나 재시작, 포기 등 가능 <br/>
  - 브라우저의 동기 작업을 차단하지 않고 백그라운드에서 새로운 리액트 트리 준비 가능
</div>
</details>

<br/>

**파이버의 구현**  
- 파이버는 하나의 작업 단위로 구성
- 렌더 단계에서 작업을 여러 단위로 분할하고 쪼갠 다음 우선순위 매김(비동기)
  - 이러한 작업은 일시 중지하고 나중에 다시 시작하거나 전에 했던 작업을 재사용, 필요하지 않은 경우 폐기 가능
  - 작업 단위를 하나씩 처리하고 finishedWork()로 마무리
- 커밋 단계에서 DOM에 실제 변경 사항을 반영하기 위해 commitWort() 실행(동기)
- 컴포넌트가 최초로 마운트되는 시점에 생성되어 이후에는 가급적 재사용
    <details>
    <summary>리액트 내부 코드에 작성되어 있는 파이버 객체(리액트 18.2.0 기준)</summary>
    <div markdown="1">
    <pre><code>
    function FiberNode(tag, pendingProps, key, mode) {
        // Instance
        this.tag = tag
        this.key = key
        this.elementType = null
        this.type = null
        this.stateNode = null  // 파이버 자체에 대한 참조 정보(이 참조를 바탕으로 리액트는 파이버와 관련된 상태에 접근)

        // Fiber
        this.return = nul1
        this.child = null     // 하나의 child만 존재
        this.sibling = null   // 나머지는 형제로 구성
        this.index = 0        // 여러 형제들 중 자신의 위치
        this.ref = null
        this.refCleanup = null
        this.pendingProps = pendingProps  // 아직 작업을 처리하지 못한 props
        this.memoizedProps = null  // 렌더링 완료 후 pendingProps를 저장 및 관리
        this.updateQueue = null    // 상태 업데이트, 콜백 함수, DOM 업데이트 등 필요한 작업 담아두는 큐
        this.memoizedState = null  // 함수 컴포넌트의 훅 목록
        this.dependencies = null
        this.mode = mode

        // Effects
        this.flags = NoFlags
        this.subtreeFlags = NoFlags
        this.deletions = null
        this.lanes = NoLanes
        this.childLanes = NoLanes
        this.alternate = null      // 반대편 트리 파이버

        // 이하프로파일러,__DEV__ 코드 생략
    }
    </code></pre>
    </div>
    </details>

**파이버 트리**  
① 현재 모습을 담은 파이버 트리  
② 작업 중인 상태를 나타내는 workInProgress 트리  

**더블 버퍼링** : 리액트가 파이버의 작업이 끝나면 단순히 포인터만 변경해 workInProgress 트리를 현재 트리로 바꿔버리는 기술
  - 불완전한 트리를 보여주지 않기 위해 사용
  - 커밋 단계에서 수행
    - 현재 UI 렌더링을 위해 존재하는 트리인 current를 기준으로 모든 작업 시작 
    - 만약 업데이트 발생하면 새로 받은 데이터로 새로운 workInProgress 트리 빌드
    - 빌드 작업이 끝나면 다음 렌더링에 이 트리 사용
    - 최종적으로 UI에 렌더링되어 반영이 완료되면 current로 변경됨
![alt text](파이버트리.png)

**파이버의 작업 순서**
- 루트 노드부터 beginWork() 함수를 실행해 파이버 작업 수행(트리 형식으로 더 이상 자식이 없는 파이버를 만날 때까지)
- 자식이 없다면 해당 노드에서 completeWork() 함수 실행해 파이버 작업 완료
- 형제가 있다면 형제로 넘어감
- 형제 노드도 작업이 완료되었다면 return으로 돌아가 자신의 작업이 완료됐음을 알림
  
### 3. 클래식 컴포넌트와 함수 컴포넌트
#### 3-1. 클래식 컴포넌트
- 구조 : class 컴포넌트 extends 만들고 싶은 컴포넌트(React.Component, React.PureComponent)
- 컴포넌트의 일부 구성 요소
  - <span style="background-color:yellowgreen">constructor()</span> : 생성자 함수
    - 컴포넌트 내부에 있다면 컴포넌트가 초기화되는 시점에 호출
        <details>
        <div markdown="1">
        constructor 없이 state 초기화 가능<br/>  
        - ES2022에 클래스 필드(class fields) 추가 : 별도의 초기화 과정 없이 클래스 내부에 필드를 선언할 수 있게 도와줌<br/>
        - ES2022 환경을 지원하는 브라우저에서만 코드 제공 또는 @babel/plugin-proposal-class-properties 사용해 트랜스파일 거쳐야 함
        </div>
        </details>
    - super() : 컴포넌트를 만들면서 상속받은 상위 컴포넌트, 즉 extends 다음 컴포넌트의 생성자 함수를 먼저 호출해 상위 컴포넌트에 접근할 수 있게 도와줌
  - <span style="background-color:yellowgreen">props</span> : 컴포넌트에 특정 속성 전달하는 용도
  - <span style="background-color:yellowgreen">state</span> : 클래스 컴포넌트 내부에서 관리하는 값
    - 항상 객체여야 함
    - 값에 변화가 있을 때마다 리렌더링 발생
  - <span style="background-color:yellowgreen">메서드</span> : 렌더링 함수 내부에서 사용되는 함수, 보통 DOM에서 발생하는 이벤트와 함께 사용됨
    - constructor에서 this 바인드 : 일반적인 함수로 메서드를 만들면 this는 전역 객체가 바인딩되어 undefined가 나옴. 따라서 생성된 함수에 bind를 활용해 강제로 this 바인딩 (예시 this.handleClick = this.handleClick.bind(this))
    - 화살표 함수 사용 : 작성 시점에 this가 상위 스코프로 결정되는 화살표 함수 사용
    - 렌더링 함수 내부에서 함수를 새롭게 만들어 전달 : 매번 렌더링 일어날 때마다 새로운 함수를 생성해 할당하므로 최적화 수행 어려움(지양)

**클래스 컴포넌트의 생명주기 메서드**
- 생명주기 메서드가 실행되는 시점
  - **마운트(mount)** : 컴포넌트가 생성되는 시점
  - **업데이트(update)** : 이미 생성된 컴포넌트의 내용이 변경되는 시점
  - **언마운트(unmount)** : 컴포넌트가 더 이상 존재하지 않는 시점
  
- 생명주기 메서드
  - <span style="background-color:skyblue">render()</span>
    - 컴포넌트가 UI를 렌더링하기 위한 함수
    - 클래스 컴포넌트의 유일한 필수 값
    - 마운트와 업데이트 과정에서 일어남
    - 항상 순수해야 하며 부수 효과가 없어야 함 (같은 입력값이 들어가면 같은 결과물 반환해야 함)
      - render() 내부에서 state 직접 업데이트하면 안 됨 -> this.state X
      - 최대한 간결 & 깔끔한 코드 작성 유지
  
  - <span style="background-color:skyblue">componentDidMount()</span>
    - 컴포넌트가 마운트되고 준비되는 즉시 실행
    - 내부에서 this.setState()로 state 값 변경 가능 -> state 변경 즉시 다시 렌더링
      - 일반적으로 state는 생성자에서 다루는 게 좋음
      - state 값 변경을 허용하는 것은 생성자 함수에서 할 수 없거나 API 호출 후 업데이트, DOM에 의존적인 작업(이벤트 리스너 추가 등) 등을 위함
    - 성능 문제 일으킬 수 있어 꼭 해당 메서드에서 할 수 밖에 없는 작업인지 확인해보고 사용해야 함
  
  - <span style="background-color:skyblue">componentDidUpdate()</span>
    - 컴포넌트가 업데이트된 이후 바로 실행
    - state나 props의 변화에 따라 DOM을 업데이트하는 등에 쓰임
    - this.setState를 사용할 수 있으나 적절한 조건문으로 감싸지 않으면 계속 호출되어 성능상 안 좋음
  
  - <span style="background-color:skyblue">componentWillUnmount()</span>
    - 컴포넌트가 언마운트되거나 더 이상 사용되지 않기 직전에 호출
    - 메모리 누수나 불필요한 작동을 막기 위한 클린업 함수를 호출하기 위한 최적의 위치
      - 이벤트를 지우거나 API 호출 취소, setInterval, setTimeout으로 생성된 타이머 지우는 등의 작업에 유용
    - this.setState 호출 불가
  
  - <span style="background-color:skyblue">shouldComponentUpdate()</span>
    - state나 props의 변경으로 컴포넌트가 리렌더링되는 것을 막고 싶을 때
      - 컴포넌트에 영향을 받지 않는 변화
    - 특정한 성능 최적화 상황에서만 고려
    - **Component와 PureComponent와의 차이**
      - Component는 해당 값이 변경될 때마다 렌더링 발생
      - PureComponent는 해당 값에 대해 얕은 비교를 수행해 결과가 다를 때만 렌더링 수행
        - 객체와 같이 복잡한 구조의 데이터 변경은 감지 못 함
        - 얕은 비교를 했을 때 일치하지 않은 경우가 많다면 비교 무의미

  - <span style="background-color:skyblue">static getDerivedStateFromProps()</span>
    - 다음에 올 props를 바탕으로 현재의 state를 변경하고 싶을 때 
    - render() 호출하기 직전에 호출
    - 사라진 componentWillReceiveProps를 대체하는 메서드
    - static으로 선언되어 this에 접근 불가
    - 반환하는 객체가 있다면 해당 객체의 내용이 모두 state로 들어가고, 없다면(null이면) 아무 일도 안 일어남
  
  - <span style="background-color:skyblue">getSnapShotBeforeUpdate()</span>
    - DOM에 렌더링되기 전에 윈도우 크기 조절, 스크롤 위치 조절 등 작업 
    - DOM 업데이트되기 직전에 호출
    - componentWillUpdate()를 대체하는 메서드
    - 반환되는 값은 componentDidUpdate로 전달
      - componentDidUpdate에서 제네릭의 3번째 인수로 snapshot을 넣을 수 있으며, 전달받은 값은 snapshot에서 접근 가능
      - snapshot 값이 있다면 스크롤 위치를 재조정해 기존 아이템이 스크롤에서 밀리지 않도록 도와줌
    - 리액트 훅으로 구현되어 있지 않아 필요하다면 반드시 클래스 컴포넌트 사용해야 함
  
    ![alt text](리액트생명주기다이어그램.png)

    <details>
    <summary>에러 상황에서 실행되는 메서드</summary>
    <div markdown="1"></div>
    <p>- ErrorBoundary(에러 경계 컴포넌트)를 만들기 위한 목적으로 사용됨 <br/>
    - 리액트 애플리케이션 전역에서 처리되지 않은 에러를 처리하기 위한 용도 <br/>
    - ErrorBoundary의 경계 외부에 있는 에러는 잡을 수 없으며, 여러 개 선언하여 컴포넌트 별 에러 처리를 다르게 적용할 수 있지만 만약 발견하지 못하면 일반적인 자바스크립트 코드처럼 에러는 throw됨 <br/>
    - 아직 리액트 훅으로 구현되어 있지 않아 필요하다면 반드시 클래스 컴포넌트 사용해야 함 <br/>
    <br/>
    <span style="background-color:skyblue">getDerivedStateFromError(error: Error)</span> <br/>
    • 자식 컴포넌트에서 에러가 발생했을 때 호출되는 에러 메서드 <br/>
    • static 메서드, error(하위 컴포넌트에서 발생한 에러)를 인수로 받음 <br/>
    • 반드시 state 값 반환 -> 하위 컴포넌트에서 에러가 발생했을 경우 어떻게 자식 컴포넌트를 렌더링할지 결정하는 용도로 제공되는 메서드이기 때문 <br/>
    • 부수 효과(에러 상태에 따른 state를 반환하는 것 이외에 console.error와 같은 모든 작업) 발생 안 됨 -> 렌더링 과정에 호출되는 메서드이기 때문, 부수 효과를 추가해도 에러는 발생하지 않지만 렌더링 과정을 불필요하게 방해함 <br/><br/>
    <span style="background-color:skyblue">componentDidCatch(error: Error, info: ErrorInfo)</span> <br/>
    • getDerivedStateFromError에서 에러를 잡고 state를 결정한 이후에 실행되는 메서드 <br/>
    • 인수(2) : 동일한 error, 정확히 어떤 컴포넌트가 에러를 발생시켰는지 정보를 가지고 있는 info (info의 componentStack은 Function.name 또는 컴포넌트의 displayName을 따르기 때문에 추적을 용이하게 하기 위해선 기명 함수나 displayName을 쓰는 것이 좋음) <br/>
    <img src="componentDidCatch(info).png" alt="componentDidCatch info.conponentStack"/>
    • 부수 효과 수행 가능 -> 커밋 단계에서 실행되기 때문 <br/>
    • 개발 모드에서는 에러가 window까지 전파되어 window.eonerror나 window.addEventListener('error', callback) 메서드가 오류를 잡을 수 있지만, 프로덕션 모드에서는 componentDidCatch에서 잡히지 않은 에러만 window까지 전파됨
    </p>
    </div>
    </details>

**클래스 컴포넌트의 한계**
- 데이터의 흐름을 추적하기 어려움
  - 서로 다른 여러 메서드에서 state의 업데이트가 일어날 수 있고, 코드 작성 시 메서드의 순서가 강제되어 있지 않아 state가 어떤 흐름으로 변경되어 렌더링 여부가 달라지는지 판단하기 어려움
- 애플리케이션 내부 로직의 재사용 어려움
  - 컴포넌트 간 중복되는 로직을 재사용하고 싶을 때 고차 컴포넌트나 props가 많아지는 래퍼 지옥(wrapper hell)에 빠져들 수 있음
- 기능이 많아질수록 컴포넌트의 크기 커짐
- 함수에 비해 상대적으로 어려움
- 코드 크기를 최적화하기 어려움(번들링 최적화하기에 불리한 조건)
- 핫 리로딩 하는데 상대적으로 불리함
  - **핫 리로딩(hot reloading)** : 코드에 변경 사항 발생했을 때 앱을 다시 시작하지 않고서도 해당 코드만 업데이트해 변경 사항을 빠르게 적용하는 기법, 개발 단계에서 많이 사용


#### 3-2. 함수 컴포넌트
리액트 16.8 버전 이전에는 단순히 무상태 컴포넌트를 구현하는 데에 사용  
이후 버전에서는 훅이 등장하며 많은 개발자들이 사용

#### 3-3. 클래스 컴포넌트와 함수 컴포넌트의 차이
| | 클래스 컴포넌트 | 함수 컴포넌트 |  
|---|---|---|
| 생명주기 메서드 | O | X |  
| 렌더링된 값 고정 | X | O |

- 생명주기 메서드의 부재
  - 클래스 컴포넌트 (O)
    - render 메서드가 있는 React.Component를 상속받아 구현하는 자바스크립트 클래스
  - 함수 컴포넌트 (X)
    - props를 받아 단순히 리액트 요소만 반환하는 함수
    - `useEffect` : useEffect 컴포넌트의 state를 활용해 동기적으로 부수 효과를 만드는 메커니즘, 생명주기 메서드(componentDidMount, componentDidUpdate, componentWillUnmount) 비슷하게 구현 가능  

- 렌더링된 값의 고정 여부
  - 클래스 컴포넌트 (X)
    - 시간의 흐름에 따라 변화하는 this를 기준으로 렌더링
    - props의 값을 항상 this로부터 가져옴
    - props는 외부에서 변경되지 않는 이상 불변 값이나 this가 가리키는 객체(컴포넌트의 인스턴스의 멤버)는 변경 가능한 값이기 때문에 부모 컴포넌트가 props를 변경해 리렌더링 되었다면 this.props의 값은 바뀌게 됨
  - 함수 컴포넌트 (O)
    - 렌더링이 일어날 때마다 그 순간의 값인 props와 state를 기준으로 렌더링
    - props를 인수로 받아 컴포넌트는 값을 변경할 수 없고 그대로 사용

## React 렌더링 과정
### 1. React 렌더링이란?
- 브라우저 렌더링 : HTML과 CSS 리소스를 기반으로 웹페이지에 필요한 UI를 그리는 과정  
- 리액트 렌더링 : 브라우저가 렌더링에 필요한 DOM 트리를 만드는 과정
  - 리액트 애플리케이션 트리 안에 있는 모든 컴포넌트들이 현재 자신이 가지고 있는 props와 state의 값을 기반으로 어떻게 UI를 구성하고, 어떤 DOM 결과를 브라우저에게 제공할 것인지 계산하는 일련의 과정
  - 시간과 리소스를 소비해 수행되는 과정

### 2. React 렌더링의 시점
- 최초 렌더링 : 사용자가 처음 애플리케이션에 진입했을 때
- 리렌더링 : 최초 렌더링 이후의 모든 렌더링
  - 클래스 컴포넌트의 setState가 실행될 때
  - 클래스 컴포넌트의 forceUpdate가 실행될 때
    - render가 state나 props가 아닌 다른 값에 의존하고 있어 리렌더링을 자동으로 실행할 수 없으면 forceUpdate 실행 가능
    - 개발자가 강제로 렌더링이 필요하다고 선언한 것이기 때문에 shouldComponentUpdate는 무시
    - render 내에 사용되면 무한 루프에 빠짐
  - 함수 컴포넌트의 useState()의 setter가 실행될 때
    - setter는 setState(클래스 컴포넌트)처럼 state 업데이트하는 함수
  - 함수 컴포넌트의 useReducer()의 dispatch가 실행될 때
    - dispatch는 상태를 업데이트하는 함수
  - 컴포넌트의 key props가 변경될 때
    - key는 형제 요소들 사이에서 동일한 요소를 식별하는 값
    - 리렌더링이 필요한 컴포넌트 최소화
      - 두 개의 파이버 트리 사이에서 같은 컴포넌트인지 구별
  - props가 변경될 때
  - 부모 컴포넌트가 렌더링될 때

### 3. React 렌더링 프로세스
① 렌더링 프로세스 시작  
② 컴포넌트의 루트부터 내려가며 업데이트가 필요하다고 지정되어 있는 모든 컴포넌트 찾음  
③ 업데이트가 필요한 컴포넌트를 발견하면 클래스 컴포넌트는 render(), 함수 컴포넌트는 FunctionComponent() 호출한 후 그 결과물 저장  
③-1. 결과물은 JSX문법으로 구성되어 있어 자바스크립트로 컴파일되며 React.createElement()를 호출하는 구문으로 변환됨 -> 자바스크립트 객체 반환  
④ 가상 DOM과 비교하여 재조정 과정  
⑤ 모든 변경 사항이 하나의 동기 시퀀스로 실제 DOM에 적용됨  
=> 이후 브라우저 렌더링 발생!

#### 3-1. 렌더 단계와 커밋 단계
**렌더 단계(Render Phase)**  
- 컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 작업
- type, props, key 중 하나라도 변경된 것이 있으면 변경이 필요한 컴포넌트로 체크

**커밋 단계(Commit Phase)**  
- 렌더 단계의 변경 사항을 실제 DOM에 적용해 사용자에게 보여주는 과정
  - DOM을 업데이트한다면 변경된 모든 DOM 노드 및 인스턴스를 가리키도록 리액트 내부의 참조를 업데이트
  - componentDidMount, componentDidUpdate(클래스 컴포넌트) 또는 useLayoutEffect(함수 컴포넌트) 호출

**리액트 렌더링이 일어난다고 해서 무조건 DOM 업데이트가 발생하진 않음**
- 렌더링을 수행했으나 변경 사항 감지 안 되면 커밋 단계 생략
- 렌더링은 가시적인 변경이 일어나지 않아도 발생 가능  
![alt text](렌더커밋단계.png)  

# Q&A
## 1. 리액트에서 index를 key값으로 사용하면 안 되는 이유는 무엇인가요?
리액트에서 index를 key 값으로 쓰게 되면 배열의 요소가 추가되거나 삭제될 때 배열의 순서가 바뀌면서 문제가 발생할 수 있기 때문입니다.  
리액트는 key를 통해 동일한 요소를 식별합니다. 예를 들어 하나의 요소가 추가되었을 때 index를 사용하고 있었다면 그 뒤의 요소들은 순서가 밀리게 되어 불필요한 렌더링이 발생하거나 요소의 상태 처리가 잘못될 수 있습니다.  
따라서 key는 배열의 순서나 요소 변경에 영향을 받지 않는 id와 같이 고유한 값을 사용하는 것이 좋습니다.

## 2. 브라우저 렌더링과 리액트 렌더링의 차이점을 설명해주세요.
브라우저 렌더링은 HTML과 CSS를 파싱하여 각각 객체 모델을 만들고 이를 결합한 후 레이아웃과 페인트 과정을 거쳐 웹페이지의 UI를 그리는 과정을 의미합니다.  
반면, 리액트 렌더링은 브라우저 렌더링에 필요한 DOM 트리를 만드는 과정으로, 각 컴포넌트들이 현재의 props와 state 값을 기반으로 UI 구성을 계산하는 일련의 과정을 의미합니다.

## 3. props와 state란 무엇인가요?
props는 부모 컴포넌트가 자식 컴포넌트에게 전달하는 데이터입니다.  
props는 읽기 전용으로 자식 컴포넌트는 수정할 수 없습니다. 또 컴포넌트 간의 데이터 흐름을 예측할 수 있게 하며, 컴포넌트의 재사용성을 높입니다.  
state는 컴포넌트 내부에서 관리되는 데이터입니다.  
state는 동적으로 변경될 수 있으며, 컴포넌트의 렌더링에 영향을 미칩니다. 이러한 state는 주로 사용자 입력이나 네트워크 요청의 응답에 따라 변하는 데이터를 관리할 때 사용됩니다.

## 4. 리액트의 재조정 과정을 리액트 18버전 이전과 이후로 비교하여 설명해주세요.
재조정은 가상 DOM과 실제 DOM을 비교하는 작업입니다.  
리액트 18버전 이전의 재조정은 동기적으로 진행되었습니다.  
하나의 스택에 렌더링이 필요한 작업들이 쌓이면 스택이 빌 때까지 동기적으로 작업되었고, 중간에 중단할 수 없었습니다. 이로 인해 렌더링 과정이 길어지면 애플리케이션 성능 저하 또는 브라우저의 다른 작업들이 지연될 가능성이 있었습니다.  
반면 리액트 18버전 이후의 재조정은 비동기적으로 진행됩니다.  
파이버를 사용하여 특정 렌더링의 우선순위를 낮추거나 필요하다면 중단 및 재시작 또는 포기 등이 가능합니다. 따라서 브라우저의 동기 작업을 차단하지 않고 백그라운드에서 새로운 리액트(파이버) 트리를 준비할 수 있게 되었습니다.

## 5. 클래스 컴포넌트와 함수 컴포넌트의 차이점을 설명해주세요.
클래스 컴포넌트와 함수 컴포넌트의 차이점은 크게 두 가지가 있습니다.  
첫 번째는 생명주기 메서드의 부재입니다.  
클래스 컴포넌트는 render 메서드가 있는 React.Component를 상속받아 구현하는 자바스크립트 클래스로 생명주기 메서드가 존재합니다.  
반면, 함수 컴포넌트는 props를 받아 단순히 리액트 요소만 반환하는 함수로 생명주기 메서드가 없습니다. 하지만 useEffect를 사용해 비슷하게 구현할 수 있습니다.  
두 번째는 렌더링된 값의 고정 여부입니다.  
클래스 컴포넌트는 시간의 흐름에 따라 변화하는 this를 기준으로 렌더링하고, 이 this로부터 props의 값을 가져오기 때문에 렌더링 값이 고정되지 않습니다.  
반면, 함수 컴포넌트는 렌더링이 일어날 때마다 그 순간의 값인 props와 state를 기준으로 렌더링합니다. 컴포넌트는 props를 인수로 받기 때문에 값을 변경할 수 없습니다.
