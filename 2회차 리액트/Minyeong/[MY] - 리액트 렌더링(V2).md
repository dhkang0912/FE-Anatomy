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
    - [3. 클래식 컴포넌트와 함수형 컴포넌트](#3-클래식-컴포넌트와-함수형-컴포넌트)
  - [React 렌더링 과정](#react-렌더링-과정-1)

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
  - 가상 DOM과 실제 DOM을 비교해 변경 사항 수집
      - 차이가 있다면 변경에 관련된 정보를 가진 파이버를 기준으로 화면에 렌더링 요청
      - 바로 처리하거나 스케줄링 => 유연한 처리
- 파이버 재조정자(fiber reconciler)가 관리
- 실행 시점 : state 변경, 생명주기 메서드 실행, DOM의 변경 필요 등



**파이버의 구현**  
- 파이버는 하나의 작업 단위로 구성
- 렌더 단계에서 작업을 여러 단위로 분할하고 쪼갠 다음 우선순위 매김(비동기)
  - 이러한 작업은 일시 중지하고 나중에 다시 시작하거나 전에 했던 작업을 재사용, 필요하지 않은 경우 폐기 가능
  - 작업 단위를 하나씩 처리하고 finishedWork()로 마무리
- 커밋 단계에서 commitWork()가 실행되어 DOM에 실제 변경 사항을 반영(동기)
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

더블 버퍼링 : 리액트가 파이버의 작업이 끝나면 단순히 포인터만 변경해 workInProgress 트리를 현재 트리로 바꿔버리는 기술
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
  
### 3. 클래식 컴포넌트와 함수형 컴포넌트

## React 렌더링 과정