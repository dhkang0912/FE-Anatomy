# Next.js의 렌더링
- [Next.js의 렌더링](#nextjs의-렌더링)
- [1. Next.js 란?](#1-nextjs-란)
  - [1-1. Next.js 등장 배경](#1-1-nextjs-등장-배경)
  - [1-2. Next.js의 장점과 단점](#1-2-nextjs의-장점과-단점)
  - [1-3. Next.js와 React의 차이점](#1-3-nextjs와-react의-차이점)
  - [1-4. 미리 알아둘 용어](#1-4-미리-알아둘-용어)
- [2. Next.js의 렌더링 과정](#2-nextjs의-렌더링-과정)
  - [2-1. 사전 렌더링(Pre-rendering)과 하이드레이션(Hydration)](#2-1-사전-렌더링pre-rendering과-하이드레이션hydration)
  - [2-2. Next.js의 주요 렌더링 방식](#2-2-nextjs의-주요-렌더링-방식)
    - [CSR(Client-Side Rendering) - 클라이언트 사이드 렌더링](#csrclient-side-rendering---클라이언트-사이드-렌더링)
    - [SSR(Server-Side Rendering) - 서버 사이드 렌더링](#ssrserver-side-rendering---서버-사이드-렌더링)
    - [SSG(Static Site Generation) - 정적 사이트 생성](#ssgstatic-site-generation---정적-사이트-생성)
    - [ISR(Incremental Static Regeneration) - 증분 정적 재생성](#isrincremental-static-regeneration---증분-정적-재생성)
  - [2-3. Next.js 라우터에 따른 렌더링 분류](#2-3-nextjs-라우터에-따른-렌더링-분류)
- [3. 전체적인 렌더링 과정](#3-전체적인-렌더링-과정)
- [Q\&A](#qa)
  - [1. CSR, SSR, SSG, ISR 차이점과 각각의 장단점은 무엇인가요?](#1-csr-ssr-ssg-isr-차이점과-각각의-장단점은-무엇인가요)
  - [2. Next.js에서 getStaticProps와 getServerSideProps의 차이점은 무엇인가요?](#2-nextjs에서-getstaticprops와-getserversideprops의-차이점은-무엇인가요)
  - [3. RSC 페이로드에 대해 설명해보세요.](#3-rsc-페이로드에-대해-설명해보세요)
  - [4. 서버 컴포넌트와 클라이언트 컴포넌트의 차이와 어떨 때 사용해야 하는지 설명해보세요.](#4-서버-컴포넌트와-클라이언트-컴포넌트의-차이와-어떨-때-사용해야-하는지-설명해보세요)
  - [5. 하이드레이션이란 무엇인가요?](#5-하이드레이션이란-무엇인가요)

# 1. Next.js 란?
Next.js는 React 기반의 웹 프레임워크로, 서버 사이드 렌더링(SSR)과 정적 사이트 생성(SSG) 등을 지원하여 SEO 최적화 및 빠른 페이지 로딩 속도를 제공하는 것이 특징이다.

## 1-1. Next.js 등장 배경
React는 원래 CSR(Client-Side Rendering, 클라이언트 사이드 렌더링) 방식을 사용했다. 하지만 이 방식에는 다음과 같은 문제가 있었다.

- SEO(검색 엔진 최적화) 문제
  - CSR은 빈 HTML을 반환하고, 브라우저에서 JavaScript가 실행되어야 화면이 보이므로 검색 엔진이 HTML을 크롤링하기 어려움
- 첫 로딩 속도 저하
  - 데이터를 불러오고 화면을 그릴 때까지 사용자가 기다려야 함
- 사용자 경험(UX) 저하
  - 초기 로딩이 느리고, 로딩 스피너만 보이면 사용자 이탈률 증가

➡ 이를 해결하기 위해 Next.js가 등장했다!  
Next.js는 SSR, SSG, ISR 등의 렌더링 방식을 제공하여 SEO 최적화 및 초기 로딩 속도를 개선할 수 있도록 만들어졌다.

## 1-2. Next.js의 장점과 단점
**장점**  
① SEO 최적화
- SSR과 SSG를 통해 완성된 HTML을 제공하여 검색 엔진 크롤링이 가능  

② 빠른 페이지 로딩 속도
- 사전 렌더링을 통해 첫 페이지 로딩 시간을 단축  

③ 자동 코드 스플리팅(Code Splitting)
- 필요한 페이지별로 JavaScript를 나누어 로딩 속도를 최적화  

④ 파일 기반 라우팅
- pages/ 또는 app/ 디렉토리에 파일을 생성하는 것만으로도 라우팅이 자동 설정됨  

⑤ 서버 기능 지원
- API Routes 기능을 통해 간단한 서버 기능도 구현 가능

**단점**  
① SSR 사용 시 서버 부하 증가
- 요청마다 서버에서 HTML을 생성하면 트래픽이 많을 때 부담이 커질 수 있음 
   
② 초기 설정이 복잡할 수 있음
- 프로젝트에 맞는 렌더링 방식을 선택하고 최적화해야 함

## 1-3. Next.js와 React의 차이점

|구분|	Next.js |	React |
|---|---|---|
|종류|	프레임워크(Framework)|	라이브러리(Library)|
|기본 렌더링 방식|	SSR, SSG, ISR, CSR 지원|	CSR|
|라우팅 방식	|파일 기반 라우팅 (pages/, app/)|	직접 react-router-dom 설정 필요|
|SEO 최적화|	SSR 및 SSG 지원으로 SEO에 유리	|CSR 기본 방식은 SEO에 불리|
|서버 기능|	API Routes 지원|	자체적으로 백엔드 기능 없음|
|상태 관리|	필요 시 별도 라이브러리 사용|	Redux, React Context 사용|

➡ 즉, React는 UI 라이브러리이고 Next.js는 이를 확장한 풀스택 프레임워크!

<details>
<summary>참고 - 라이브러리와 프레임워크의 차이</summary>
1. 라이브러리(Library) <br/>
  특정 기능을 모아둔 코드 집합으로, 개발자가 필요한 기능을 가져와서 사용하는 방식 => 개발자가 제어<br/>
2. 프레임워크(Framework)<br/>
   애플리케이션의 전체 구조와 흐름을 제어하는 틀로, 개발자는 필요한 부분을 채워넣는 방식 => 프레임워크가 제어
</details>

## 1-4. 미리 알아둘 용어
- `getServerSideProps` (SSR)
  - 요청이 들어올 때마다 서버에서 데이터를 가져와 HTML을 생성
  - 최신 데이터를 제공할 때 유용하지만 서버 부하가 큼
  ```JavaScript
  export async function getServerSideProps() {
  const res = await fetch("https://api.example.com/data");
  const data = await res.json();
  return { props: { data } };
  ```

- `getStaticProps` (SSG)
  - 빌드 시점에 데이터를 미리 가져와 정적 HTML을 생성하여 제공
  - SEO 최적화 및 속도가 빠름
  ```JavaScript
  export async function getStaticProps() {
  const res = await fetch("https://api.example.com/data");
  const data = await res.json();
  return { props: { data } };
  ```

- `getStaticPaths` (SSG)
  - 빌드 시점에 동적 경로의 정적 페이지 생성을 위해 사용
  - pages 디렉터리 안에 [param].js 같은 동적 라우트 파일을 만들었을 때 사용
    - `getStaticPaths` 함수에서 경로 목록을 반환
    - Next.js는 빌드 시 그 경로들에 대해 getStaticProps를 실행해 정적 HTML 파일로 생성
    - 반환할 땐 paths 배열과 fallback 값 지정
      ```JavaScript
      export async function getStaticPaths() {
        return {
          paths: [
            { params: { id: '1' } },
            { params: { id: '2' } },
          ],
          fallback: false,
        };
      }
      ```
      <details>
      <summary>fallback 옵션 정리</summary>
      - false : paths에 정의된 경로만 생성, 나머지는 404 (경로가 적고 고정된 경우 적합) <br/>
      - true : paths에 없는 경로는 요청 시 서버에서 HTML 생성 후 캐시에 저장 (경로가 많거나 동적 추가되는 경우 적합, 로딩 UI 필요) <br/>
      - 'blocking' : true와 유사, 생성이 완료될 때까지 기다린 후 완성된 페이지 반환 (로딩 화면 없이 처음부터 완성된 페이지 제공)

- `fetch()` API  
  - Next.js에서는 서버와 클라이언트에서 모두 fetch()를 사용할 수 있음
  ```JavaScript
  const res = await fetch("/api/data");
  const data = await res.json();
  ```

# 2. Next.js의 렌더링 과정
## 2-1. 사전 렌더링(Pre-rendering)과 하이드레이션(Hydration)

**사전 렌더링(Pre-rendering)**
- Next.js는 기본적으로 모든 페이지를 사전 렌더링
- 초기 HTML을 미리 생성해 브라우저에 전달
- SSR(빌드 시 생성), SSG(요청 시 생성)

<br/>

**하이드레이션(Hydration)**
- 브라우저에서 JavaScript가 실행되면서 React가 이벤트 핸들러를 연결하고 동적 기능 활성화하는 과정
  - 서버에서 렌더링된 정적인 HTML을 동적으로 변경할 수 있도록 함
  - JavaScript가 실행되기 전까지는 상호작용 불가능   
- 단계
  - HTML과 CSS가 브라우저에 로드
  - JavaScript가 실행되며 React가 HTML을 활성화(takeover)
  - 인터랙션(클릭, 입력 등) 가능
- 하이드레이션 속도가 느리면 사용자 경험 나빠질 수 있음 => React Server Components로 최적화 가능


## 2-2. Next.js의 주요 렌더링 방식
### CSR(Client-Side Rendering) - 클라이언트 사이드 렌더링
**동작 과정**
- 사용자가 웹사이트에 접속하면 서버는 빈 HTML 반환, JavaScript 다운로드
- 브라우저가 JavaScript를 실행해 화면을 만들고, 데이터를 API에서 불러와 채움

**특징**  
- 화면을 구성하는 작업이 전적으로 브라우저에서 이루어짐
- 검색 엔진이 가져가는 HTML이 비어있어 SEO 불리
- 첫 화면을 그리기까지 시간이 오래 걸릴 수 있음

**사용하는 경우**
- 로그인 후에만 접속 가능한 관리자 페이지
- 검색 엔진 최적화가 필요 없는 대시보드, 실시간 데이터 페이지

### SSR(Server-Side Rendering) - 서버 사이드 렌더링
**동작 과정**
- 사용자가 페이지를 요청하면 서버에서 HTML을 완성해 반환
  - getServerSideProps 사용해 데이터를 서버에서 미리 가져옴
- 브라우저는 HTML을 받은 즉시 렌더링
- 이후 React가 실행되며 브라우저에서 상호작용할 수 있게 만듦

**특징**  
- HTML이 미리 서버에서 완성되어 SEO 최적화에 유리
- 화면이 더 빨리 보여 UX 향상
- 매 요청마다 서버에서 HTML을 다시 만들어야 해 서버 부하 증가
- 데이터 요청 시간이 길어질 경우 화면이 늦게 뜰 수 있음

**사용하는 경우**
- SEO가 중요한 블로그, 뉴스 사이트
- 사용자가 요청할 때마다 새로운 데이터가 필요한 페이지(검색 결과 페이지)

### SSG(Static Site Generation) - 정적 사이트 생성
**동작 과정**
- 빌드할 때 HTML 미리 만듦
  - getStaticProps, getStaticPaths 사용해 데이터를 미리 가져와 HTML에 포함
- 사용자가 요청하면 CDN에서 이미 만들어진 HTML을 받아 그대로 보여줌

**특징**  
- 정적 HTML을 제공해 SEO 최적화 및 빠른 속도
- 서버에서 처리할 필요가 없어 트래픽이 많아도 부담 적음
- 빌드 이후 데이터가 변경되면 반영 안 됨
  - 배포 후 데이터가 바뀌면 다시 빌드해야 반영됨

**사용하는 경우**
- 콘텐츠가 자주 바뀌지 않는 페이지(블로그, 제품 상세 페이지)
- SEO가 중요한 마케팅 랜딩 페이지

### ISR(Incremental Static Regeneration) - 증분 정적 재생성
**동작 과정**
- SSG처럼 미리 정적 HTML 만들어 제공
- `revalidate` 설정해 새로운 데이터를 받을 시간 설정
- 해당 주기가 지나면 다음 요청 때 백그라운드에서 HTML 재생성 및 CDN 업데이트
  ```JavaScript
  export async function getStaticProps() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  return { props: { data }, revalidate: 10 };} // 10초마다 새로운 데이터 갱신
  ```

**특징**  
- 속도가 빠르며 데이터 변경 반영 가능
- 데이터가 바뀌는 주기가 일정하다면 효율적

**사용하는 경우**
- 블로그 페이지(새 글이 올라오면 일정 시간마다 갱신)
- 자주 업데이트되지만 실시간은 필요 없는 페이지

<details>
<summary>CDN(Content Delivery Network)</summary>
전 세계 여러 위치에 정적 파일(HTML, CSS, JS, 이미지 등)을 캐싱해 두는 분산 서버 <br/>
- 가까운 서버에서 빠르게 파일을 가져오도록 해 속도를 높임
</details>


| 구분 | CSR (Client Side Rendering) | SSR (Server Side Rendering) | SSG (Static Site Generation) | ISR (Incremental Static Regeneration) |
|------|-----------------------------|-----------------------------|-----------------------------|--------------------------------------|
| **사용 함수** | `useEffect`, `useState` | `getServerSideProps` | `getStaticProps` | `getStaticProps` + `revalidate` 옵션 |
| **렌더링 시점** | 클라이언트에서 실행 시 렌더링 | 요청 시 서버에서 렌더링 | 빌드 시 정적 파일 생성 | 빌드 시 정적 파일 생성 + 요청 시 일정 주기로 재생성 |
| **초기 로딩 속도** | 느림 (JS 다운로드 후 렌더링) | 빠름 (서버에서 미리 렌더링) | 매우 빠름 (정적 파일 제공) | 빠름 (정적 파일 제공, 주기적 업데이트) |
| **SEO** | 나쁨 (JS 의존도 높음) | 좋음 (HTML 완성 형태 제공) |  좋음 (HTML 완성 형태 제공) | 좋음 (정적 페이지 제공 + 업데이트 가능) |
| **데이터 최신성** | 실시간 (최신 데이터 가능) | 요청 시 최신 데이터 |  정적 (빌드 시점 데이터 고정) |  일정 주기로 최신 데이터 업데이트 |
| **SPA/MPA** | SPA | SPA / MPA | SPA | SPA |
| **사용 예** | SPA, 대시보드 | 블로그, 마케팅 페이지 | 블로그, 문서 사이트 | 이커머스, 블로그 (자주 변경되는 페이지) |




## 2-3. Next.js 라우터에 따른 렌더링 분류
|구분|	Page Router (pages/)|	App Router (app/)|
|---|---|---|
|렌더링 방식|	파일 기반 라우팅 (pages/index.tsx)|	디렉토리 기반 라우팅 (app/page.tsx)|
|데이터 패칭|	getServerSideProps, getStaticProps, getInitialProps 사용|	React Server Components(RSC), fetch() 사용|
|SSR/SSG 방식|	getServerSideProps, getStaticProps로 컨트롤|	서버 컴포넌트 기반으로 자동 최적화|
|ISR 지원 | revalidate 옵션 사용 | 기본적으로 지원|
|Streaming 지원|없음	| 지원 (점진적 로딩 가능)|
|CSR 사용 | 기본적으로 사용 | "use Client" 사용해야함|

**Page Router (pages/)**
- 기존 Next.js 라우팅 방식
- getServerSideProps, getStaticProps를 사용하여 렌더링 방식을 제어

**App Router (app/)**
- Next.js 13에서 도입된 새로운 방식
- React Server Components(RSC)를 활용해 서버에서 렌더링
  - RSC는 React에서 서버에서만 렌더링되는 컴포넌트
  - 서버에서 컴포넌트를 렌더링한 결과를 HTML처럼 바로 보내는 게 아닌, JSON 형태의 데이터로 전달
    - 서버 컴포넌트는 서버에서 실행되기 때문에 fetch나 DB 조회 등 서버 자원 접근 가능 -> 데이터를 가져와 컴포넌트 트리의 일부로 렌더링 결과에 포함
      - 서버는 여러 API를 비동기 병렬(fetch나 Promise.all)로 호출 가능
  - **RSC 페이로드** : 어떤 컴포넌트를 어떤 트리 구조로 보여줘야 하는지에 대한 정보
    - 어떤 컴포넌트 트리가 그려져야 하는지 정보
    - 각 컴포넌트의 props 값
    - 서버 컴포넌트 내부에서 가져온 최종 데이터 결과
    - 즉, 서버에서 미리 데이터 패칭까지 끝내고, 그 데이터를 포함한 컴포넌트 구조 정보
  - 클라이언트는 이 정보를 받아 실제 필요한 부분만 클라이언트 컴포넌트로 동작할 수 있게 하이드레이션(API 재호출 안 함)
  - 기존에는 모든 컴포넌트가 클라이언트로 번들링되어 내려갔기 때문에 JS 용량이 커졌으나, RSC는 서버에서 무거운 계산 등을 처리하고 최소한의 정보만 클라이언트로 보내 JS 번들이 가벼워지고 결과적으로 빠른 초기 렌더링과 가벼운 페이지 제공 가능
- `Suspense`와 함께 사용하면 좋음
  - Suspense : 비동기 작업이 끝날 때까지 대기 상태를 관리하고, 그동안 보여줄 대체 UI(fallback)를 지정해주는 기능
  - pages router에서도 `React.lazy()`를 통해 JS 모듈 로딩시 import하여 제공 가능
  ```tsx
  import { Suspense } from 'react';

  function Loading() {
    return <p>로딩 중...</p>;
  }

  function SomeComponent() {
    // 내부에서 데이터 가져오거나 무거운 작업 수행
    return <div>완성된 콘텐츠</div>;
  }

  export default function Page() {
    return (
      <Suspense fallback={<Loading />}>
        <SomeComponent />
      </Suspense>
    );
  }
  ```

<details>
<summary>서버 컴포넌트와 클라이언트 컴포넌트</summary>
<div markdown="1">
<span style="font-weight:bold">서버 컴포넌트(Server Component)</span> <br/>
서버에서 실행되는 React 컴포넌트로, 클라이언트에 불필요한 JavaScript를 최소화하여 성능 최적화 <br/><br/>
<span style="font-weight:bold">특징</span> <br/>

- 기본적으로 모든 컴포넌트는 서버 컴포넌트 
- 클라이언트에 JavaScript 코드가 전달되지 않음(불필요한 실행 방지, 보안 강화)
- fetch()를 직접 사용할 수 있음(서버에서 데이터를 가져와 바로 HTML 생성)
- 상태 관리, 이벤트 핸들링 등의 기능은 사용 불가 -> 클라이언트 컴포넌트를 따로 만들어야 함
  
<br/>
<span style="font-weight:bold">동작 과정</span> <br/>

- fetch()가 서버에서 실행되어 API에서 데이터를 가져옴
- 서버에서 HTML을 생성하여 클라이언트에 전송
- 클라이언트는 완성된 HTML만 받아 렌더링 <br/>
<hr/>
<br/>
<span style="font-weight:bold">클라이언트 컴포넌트(Client Component)</span> <br/>
클라이언트에서 실행되는 컴포넌트로, "use Client" 선언 필요 <br/>
<br/>
<span style="font-weight:bold">특징</span> <br/>

  - "use client" 선언 해야함
  - useState, useEffect 등 훅 사용 가능
  - 이벤트 핸들링 가능
  - 서버에서 실행되지 않으며, 클라이언트에서 JavaScript를 통해 동작

<br/>
<span style="font-weight:bold">동작 과정</span> <br/>
  
  - Next.js가 서버에서 초기 HTML을 렌더링해 클라이언트에 전달
  - 클라이언트가 HTML을 받은 후, React가 Hydration을 수행하여 이벤트 핸들러를 연결
  - 이후 인터랙션을 통해 상태가 변경되면 Virtual DOM을 통해 UI 업데이트됨
<br/>
</div>
</details>

<details>
<summary>Streaming이란?</summary>
<div markdown="1">
데이터를 한 번에 모두 전송하는 것이 아닌 조각(chunk) 단위로 조금씩 보내면서 점진적으로 처리하는 방식 <br/>
- 서버가 HTML을 완성될 때까지 기다리지 않고 부분적으로 렌더링 가능한 부분부터 먼저 전송 가능 <br/>
  
- 브라우저가 요청 보냄 -> 서버가 렌더링 가능한 부분부터 즉시 브라우저로 전송 -> 이후 데이터가 준비되는 대로 추가적인 HTML을 조각 단위로 전송 -> 브라우저는 받아온 HTML을 즉시 렌더링하면서 점진적으로 페이지 완성
</div>
</details>

<br />

  
# 3. 전체적인 렌더링 과정
① Next.js 렌더링  
- SSR/SSG/ISR/CSR 중 하나를 선택하여 HTML 생성
- getServerSideProps, getStaticProps 등이 실행해 데이터를 가져옴
- HTML과 JavaScript 번들을 브라우저에 전달

② React 렌더링 (Virtual DOM & Hydration)
- Next.js가 전달한 HTML을 브라우저가 먼저 렌더링
- React가 실행되고 Virtual DOM 생성
- React가 Hydration을 수행하면서 이벤트 핸들러를 추가
- 이후 상태 변화에 따라 변경 사항을 감지하고 업데이트 수행

③ 브라우저 렌더링
- HTML & CSS 파싱하여 DOM, CSSOM 생성
- Render Tree 형성 (DOM + CSSOM 결합)
- Layout → Paint → Composite & Display 과정을 거쳐 최종적으로 화면 표시

# Q&A
## 1. CSR, SSR, SSG, ISR 차이점과 각각의 장단점은 무엇인가요?
CSR은 요청 시 서버가 최소한의 HTML과 JavaScript를 반환하고, 브라우저에서 JS를 실행해 화면을 렌더링하는 방식입니다. 동적 상호작용에 강하지만 초기 로딩 시 콘텐츠가 보이지 않고 SEO에 취약할 수 있습니다.  
SSR은 요청 시 서버에서 HTML을 완성해 반환하는 방식으로, 초기 로딩 속도와 SEO가 뛰어나지만 요청마다 서버의 연산 부하가 발생합니다.  
SSG는 빌드 시 HTML을 미리 생성해 배포하는 방식으로, 성능과 비용 효율이 좋고 빠르게 응답할 수 있지만 실시간 데이터 반영이 어렵습니다.  
ISR은 SSG의 한계를 보완하는 방식으로, revalidate 속성을 사용해 일정 주기마다 정적 페이지를 자동으로 재생성합니다. 빠른 성능과 일정 주기의 최신성을 동시에 가져갈 수 있지만 구현과 관리가 상대적으로 복잡합니다.

## 2. Next.js에서 getStaticProps와 getServerSideProps의 차이점은 무엇인가요?
getStaticProps는 빌드 시 실행되어 정적 페이지 생성을 위한 데이터를 미리 가져오는 함수입니다. 변동이 거의 없거나 주기적인 콘텐츠에 적합하며 성능이 뛰어납니다.  
getServerSideProps는 페이지 요청 시마다 서버에서 실행되어 데이터를 가져오고 HTML을 동적으로 생성하는 함수입니다. 실시간 데이터나 사용자 맞춤형 페이지처럼 매 요청마다 다른 정보를 보여줘야 할 때 적합합니다.

## 3. RSC 페이로드에 대해 설명해보세요.
RSC는 React Server Components의 약자로, 클라이언트가 서버로부터 JSON 형태의 구성 정보(페이로드)를 받아 필요한 컴포넌트를 클라이언트에서 조립하는 방식입니다.  
이 페이로드에는 HTML이 아닌 렌더링 정보와 컴포넌트 트리 정보가 담겨 있으며, 클라이언트는 이 데이터를 기반으로 UI를 렌더링합니다.  
이를 통해 불필요한 JavaScript 번들을 줄이고, 서버 자원을 활용해 초기 렌더링 속도와 성능을 개선할 수 있습니다.  

## 4. 서버 컴포넌트와 클라이언트 컴포넌트의 차이와 어떨 때 사용해야 하는지 설명해보세요.
서버 컴포넌트는 서버에서 렌더링되고 브라우저로 전달되며, 클라이언트 측에서는 별도의 JS 번들로 다운로드되지 않습니다. 주로 데이터 페칭, 보안 처리, 렌더링 최적화에 적합합니다.  
클라이언트 컴포넌트는 브라우저에서 실행되는 컴포넌트로, 상호작용이 필요한 부분(버튼 클릭, 입력 폼 등)이나 상태 관리를 필요로 하는 UI에 사용됩니다.  
일반적으로 데이터 중심적이고 상호작용이 필요 없는 컴포넌트는 서버 컴포넌트로 작성하고, 사용자 이벤트나 동적 상태가 필요한 부분은 클라이언트 컴포넌트로 나누어 설계합니다.

## 5. 하이드레이션이란 무엇인가요?
하이드레이션은 요청 또는 빌드 시점에 생성된 정적 HTML에 클라이언트 측 JavaScript를 연결해 상호작용 기능을 활성화하는 과정입니다.  
초기에는 정적인 HTML로 빠르게 화면을 렌더링하고, 이후 React의 가상 DOM과 연결하여 이벤트 처리 및 상태 변경과 같은 동적 기능이 가능하도록 만드는 단계입니다.  
이 과정을 통해 SSR이나 SSG로 미리 렌더링된 페이지가 사용자와 상호작용할 수 있는 완전한 리액트 앱으로 동작하게 됩니다.