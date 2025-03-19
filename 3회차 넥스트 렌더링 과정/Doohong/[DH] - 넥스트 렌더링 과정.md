- [넥스트 렌더링 과정](#넥스트-렌더링-과정)
  - [1. 예제로 알아보는 Next js 렌더링 기법 (CSR, SSR, SSG, ISR)](#1-예제로-알아보는-next-js-렌더링-기법-csr-ssr-ssg-isr)
    - [1-1. Next js를 사용하는 이유](#1-1-next-js를-사용하는-이유)
    - [1-2. 왜 Next가 SEO에 유리할까?](#1-2-왜-next가-seo에-유리할까)
      - [(1) Rendering 기법](#1-rendering-기법)
      - [(2) CSR](#2-csr)
      - [(3) SSR](#3-ssr)
      - [(4) 초기 렌더링과 하이드레이션](#4-초기-렌더링과-하이드레이션)
      - [(5) SSG](#5-ssg)
      - [(6) ISR](#6-isr)
    - [1-3. 렌더링 패턴 구현](#1-3-렌더링-패턴-구현)
      - [1. CSR](#1-csr)
      - [2. SSR](#2-ssr)
      - [3. SSG](#3-ssg)
      - [4. ISR](#4-isr)
  - [2. NextJS에서 다양한 렌더링 방식들은 서로 어떤 연관이 있을까? (SSG, SSR, ISR, CSR, Static(Pre), Dynamic) ](#2-nextjs에서-다양한-렌더링-방식들은-서로-어떤-연관이-있을까-ssg-ssr-isr-csr-staticpre-dynamic-)
    - [(1) 개요](#1-개요)
    - [(2) SSG 구현 방식](#2-ssg-구현-방식)
      - [1. Pages 라우터에서 SSG 구현 방식](#1-pages-라우터에서-ssg-구현-방식)
      - [2. App 라우터에서 SSG 구현 방식](#2-app-라우터에서-ssg-구현-방식)
    - [(3) SSR 구현 방식](#3-ssr-구현-방식)
      - [1. Pages 라우터에서 SSR 구현 방식](#1-pages-라우터에서-ssr-구현-방식)
      - [2. App 라우터에서 SSR 구현 방식](#2-app-라우터에서-ssr-구현-방식)
    - [(4) CSR \<-\> SSR / SSG, SSR, ISG 맥락에서 SSR 의미](#4-csr---ssr--ssg-ssr-isg-맥락에서-ssr-의미)
      - [1. CSR \<-\> SSR](#1-csr---ssr)
      - [2. SSG, SSR, ISG 맥락](#2-ssg-ssr-isg-맥락)
    - [(5) ISR (Incremental Static Regeneration)](#5-isr-incremental-static-regeneration)
      - [1. Pages 라우터에서 ISR 구현 방식](#1-pages-라우터에서-isr-구현-방식)
      - [2. App 라우터에서 ISR 구현 방식](#2-app-라우터에서-isr-구현-방식)
  - [3. Next 렌더링 과정 요약](#3-next-렌더링-과정-요약)
  - [Q\&A](#qa)
    - [Q1. 브라우저, 리액트, 넥스트 렌더링 과정을 연결해서 설명해주세요.](#q1-브라우저-리액트-넥스트-렌더링-과정을-연결해서-설명해주세요)
    - [Q2. Next의 렌더링 방식의 종류와 각 렌더링 방식의 차이점을 설명해주세요.](#q2-next의-렌더링-방식의-종류와-각-렌더링-방식의-차이점을-설명해주세요)
    - [Q3. ISR에서 상태 변화는 즉시 반영되지만 HTML은 변경되지 않는 이유를 설명해주세요.](#q3-isr에서-상태-변화는-즉시-반영되지만-html은-변경되지-않는-이유를-설명해주세요)
    - [Q4. 사용자는 항상 넥스트를 사용할 때 리액트보다 빠르게 느끼나요? 관련해서 설명해주세요.](#q4-사용자는-항상-넥스트를-사용할-때-리액트보다-빠르게-느끼나요-관련해서-설명해주세요)
    - [Q5. SSR에서 상태 변화가 발생하면 어떻게 처리되나요? SSR에서 상태 기반 렌더링이 가능한가요?](#q5-ssr에서-상태-변화가-발생하면-어떻게-처리되나요-ssr에서-상태-기반-렌더링이-가능한가요)

# 넥스트 렌더링 과정
## 1. [예제로 알아보는 Next js 렌더링 기법 (CSR, SSR, SSG, ISR)](https://www.youtube.com/watch?v=GswzHF5UpHA)

### 1-1. Next js를 사용하는 이유
- 대다수의 기업은 Next.Js를 사용함
  - **why? : Next.Js는 React와 달리 SEO에 유리함**
    - SEO : 검색엔진최적화
      - 구글, 네이버에 특정한 키워드를 입력하여 검색하면 내가 만든 웹사이트가 상위에 노출될 수 있도록 하는 기술

### 1-2. 왜 Next가 SEO에 유리할까?
#### (1) Rendering 기법
- No Pre-rendering  
  ➀ CSR 
- Pre-rendering  
  ➁ SSR   
  ➂ SSG   
  ➃ ISG

#### (2) CSR  
- **CSR (Client Side Rendering) : 브라우저에서 JavaScript를 이용해 `동적으로 페이지를 렌더링`하는 방식**  
  - 리액트의 기본적인 렌더링 방식 
  - 렌더링의 주체 : 클라이언트 
  - SEO에 불리
  - **No Pre-rendering**  
    - JavaScript가 로딩되고 실행될 때까지 페이지가 비어 있음  
      - JS가 로딩된 후에 React가 DOM을 생성해야 화면이 보임  
    - 페이지가 로딩되기 전에는 사용자는 페이지를 볼 수 없고 JavaScript가 로딩되어야만 페이지를 볼 수 있음  
    - ![alt text](<images/No Pre-rendering.png>)  

<br>

- **CSR 과정**  
  1. 클라이언트가 페이지 요청  
  2. 서버는 빈 HTML과 JS 파일만 반환  
     - `<div id="root"></div>`  
     - 아무것도 없는 빈 구조만 반환  
  3. 클라이언트는 JS 파일을 다운로드한 후 React가 실행되면서 `Virtual DOM` 생성  
     - 이 과정은 하이드레이션이 아니라 **초기 렌더링**  
     - React가 Virtual DOM을 기반으로 실제 DOM을 생성하고 화면에 렌더링  
     - 상태 및 이벤트 핸들러가 연결되면서 상호작용 가능해짐  

<br>

#### (3) SSR 
- **SSR (Server Side Rendering) : 사용자의 요청이 있을 때마다 `서버`에서 페이지의 `HTML을 생성`**
  - Next의 기본적인 렌더링 방식
  - 렌더링의 주체 : 서버 
  - Pre-Rendering이 이뤄어짐
    - 사용자가 요청이 있을 때 이루어짐
  - **장점**
    - 실시간 데이터 제공
    - SEO에 유리
    - 클라이언트 측에서 별도로 데이터를 패칭할 필요가 없음
      - 서버 컴포넌트는 데이터베이스나 API에서 데이터를 직접 가져와서 사용할 수 있음
  - **단점** 
    - 사용자의 요청 때마다 서버에서 페이지의 HTML을 생성하기 때문에 서버 부하 증가

<br>

- **SSR 과정**  
  1. **클라이언트가 페이지 요청**  
     - 브라우저에서 특정 URL로 요청 발생  

  2. **서버가 클라이언트에 HTML 제공**  
     - 서버에서 `renderToString()`으로 생성된 HTML이 클라이언트에 전송됨  
     - 예시: `<h1>Hello from SSR</h1>`  
     - 서버에서 HTML 생성 시점에서 상태 및 이벤트 핸들러는 연결되지 않음  

  3. **브라우저에서 HTML을 받아서 DOM을 먼저 렌더링 (초기 렌더링 발생)**  
     - 서버에서 받은 HTML이 브라우저에서 DOM으로 렌더링됨  
     - 이 시점에서 UI는 보이지만 상태 및 이벤트는 작동하지 않음  
     - 사용자는 HTML에 렌더링된 UI를 볼 수 있지만 버튼 클릭 같은 상호작용은 불가능함  
     - `개발자 도구 > Network > Preview` 에서 확인 가능

  4. **클라이언트는 요청한 모든 스크립트를 다운로드**  
     - React와 관련된 JS 파일이 브라우저에 의해 로딩됨  

  5. **JS 파일이 로드되면 React가 실행되면서 하이드레이션 발생**  
     - React가 실행되면서 Virtual DOM 생성  
     - 기존 DOM과 Virtual DOM을 비교 → 일치하면 기존 DOM을 유지하고 상태 및 이벤트 핸들러 연결  
     - 차이가 발생하면 **Hydration Mismatch** 발생 → 기존 DOM 삭제 후 새로 생성  

  6. **상태 및 이벤트 핸들러가 연결되면서 상호작용 가능**  
     - 하이드레이션 완료 후 버튼 클릭 같은 이벤트가 정상적으로 작동  
     - 상태 업데이트 및 사용자 상호작용이 활성화됨  

<br>

- **사전 렌더링 (Pre-Rendering) : 사용자가 페이지를 요청하기 전 HTML을 미리 생성하여 전달**  
  - `SSR, SSG, ISR` 에서 이루어짐
  - 서버에서 만들어놓은 HTML을 자바스크립트가 로드되기 전 미리 전달
    - 상호작용이 되기 전 미리 화면이 보임  
    - `개발자 도구 > Network > Preview` 에서 확인 가능  
    - 브라우저는 HTML을 즉시 DOM으로 변환 → 빠르게 화면 표시  
    - JS 파일이 로딩되기 전까지는 상호작용 불가능  

  - **서버에서 보여지기 때문에 SEO에 유리함**  
    - 검색 엔진이 HTML 내용을 직접 읽을 수 있어 색인(indexing)에 유리함  
    - 빈 HTML 구조만 반환되는 CSR보다 검색 엔진 노출이 좋음  
    - ✅ 구글 검색 등에서 잘 노출됨  
    - ✅ Open Graph, Twitter Card 같은 메타데이터 설정 가능  

  - **예 : 뉴스 사이트 (속도, 실시간 업데이트 중요)**  
    - 뉴스는 최신 정보를 빠르게 보여줘야 하므로 SSR 사용이 적합  
    - 실시간으로 반영이 필요하므로 SSR/ISR 사용  
    - 정적 콘텐츠가 많은 블로그, 문서 사이트는 SSG 사용  


<br>

#### (4) 초기 렌더링과 하이드레이션

- **초기 렌더링 (Initial Rendering) : 브라우저에서 최초로 DOM이 생성되고 사용자에게 UI가 표시되는 과정**  
  - 초기 렌더링은 `SSR(Server Side Rendering), SSG(Static Site Generation), CSR(Client Side Rendering)`에서 모두 발생  

  - **CSR에서는 JS 파일이 로딩된 후**  
    - React가 Virtual DOM을 생성하고 DOM에 렌더링  
    - 상태 및 이벤트 핸들러까지 이 단계에서 연결됨  
    - React가 DOM을 직접 생성하고 UI를 표시함  
    - 예: `ReactDOM.createRoot()`에서 컴포넌트를 렌더링하는 과정  

  - **SSR/SSG에서는 서버에서 이미 생성된 HTML이 브라우저에서 DOM으로 렌더링됨**  
    - 서버에서 생성된 HTML이 브라우저에서 DOM으로 렌더링되는 순간이 초기 렌더링  
    - 상태 및 이벤트 핸들러는 연결되지 않은 상태  
    - JS 파일이 로딩된 후 React가 Virtual DOM을 생성하고 기존 DOM과 비교 → 상태 및 이벤트 핸들러 연결 (하이드레이션)  

  - **SSR의 경우 서버에서 전달된 HTML은 브라우저에서 재활용되기 때문에 React가 DOM을 다시 생성하지 않음**  
    - 기존 DOM을 유지하면서 상태 및 이벤트 핸들러만 연결됨  

<br>

- **하이드레이션 (Hydration) : 서버에서 렌더링된 HTML에 React가 상태와 이벤트 핸들러를 연결해 상호작용할 수 있도록 만드는 과정**  
  - 하이드레이션은 `SSR(Server Side Rendering)` 또는 `SSG(Static Site Generation)`에서만 발생  
  - CSR에서는 하이드레이션이 발생하지 않음 (React가 처음부터 DOM을 생성하기 때문)  

  - 하이드레이션 과정에서 React는 다음과 같은 작업을 수행함:  
    - 서버에서 제공한 HTML을 기반으로 **Virtual DOM 생성**  
    - 기존 DOM과 Virtual DOM을 비교 (**Diffing**)  
    - 차이가 없으면 기존 DOM을 유지하고 상태 및 이벤트 핸들러만 연결  
    - 차이가 있으면 React가 기존 DOM을 삭제하고 새로 생성함 (**Hydration Mismatch**)  

  - **Hydration Mismatch 발생 시 문제점**  
    - 서버에서 보낸 HTML과 클라이언트에서 생성한 Virtual DOM이 다르면 발생  
    - 경고 메시지가 출력됨  
    - 기존 DOM이 삭제되고 새로 생성될 수 있음 → 깜빡임 발생 가능  
    - 상태 및 이벤트 핸들러가 연결되지 않아 작동하지 않을 수 있음  

<br>


- 🌟 **초기 렌더링, 사전 렌더링, 하이드레이션 정의 및 특징**  
  | 구분             | 정의                                                                            | 특징                                                                                                         |
  | ---------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
  | **초기 렌더링**  | DOM이 처음으로 생성되고 사용자에게 표시되는 과정                                | - CSR: JS 로드 후 React가 DOM 생성 → 상태 및 이벤트 핸들러 연결<br>- SSR/SSG: 서버에서 HTML 제공 후 DOM 생성 |
  | **사전 렌더링**  | 서버에서 HTML을 미리 생성하고 클라이언트에 제공하는 과정                        | - SSR: 매 요청마다 HTML 생성<br>- SSG: 빌드 시 HTML 생성<br>- ISG: 일정 주기마다 HTML 생성                   |
  | **하이드레이션** | 기존 DOM에 React 상태 및 이벤트 핸들러를 연결하여 상호작용 가능하게 만드는 과정 | - CSR: 발생하지 않음<br>- SSR/SSG/ISG: JS 로드 후 상태 및 이벤트 핸들러 연결 단계에서 발생                   |


<br>

- 🌟 **CSR, SSR, SSG, ISG에서의 초기 렌더링, 사전 렌더링, 하이드레이션 흐름**
  | 구분                         | CSR                                               | SSR                                               | SSG                                              | ISG                                               |
  | ---------------------------- | ------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------- |
  | **초기 렌더링 발생 시점**    | JS 로드 후 React가 DOM 생성                       | 서버에서 HTML 생성 후 브라우저에서 렌더링         | 빌드 시 HTML 생성 후 브라우저에서 렌더링         | 최초 빌드 시 HTML 생성 후 브라우저에서 렌더링     |
  | **초기 렌더링 주체**         | 클라이언트에서 React가 Virtual DOM 생성 후 렌더링 | 서버에서 HTML 생성 후 브라우저에서 DOM 렌더링     | 빌드 시 HTML 생성 후 브라우저에서 DOM 렌더링     | 빌드 시 HTML 생성 후 브라우저에서 DOM 렌더링      |
  | **사전 렌더링 발생 시점**    | ❌ 발생하지 않음                                   | 매 요청마다 서버에서 HTML 생성                    | 빌드 시 HTML 생성                                | 최초 빌드 시 HTML 생성 후 일정 시간마다 HTML 갱신 |
  | **사전 렌더링 주체**         | ❌ 발생하지 않음                                   | 서버에서 `getServerSideProps()` 실행 후 HTML 반환 | 서버에서 `getStaticProps()` 실행 후 HTML 반환    | 최초 빌드 시 `getStaticProps()` 실행 후 HTML 반환 |
  | **하이드레이션 발생 시점**   | ❌ 발생하지 않음                                   | JS 로드 후 상태 및 이벤트 핸들러 연결 시 발생     | JS 로드 후 상태 및 이벤트 핸들러 연결 시 발생    | JS 로드 후 상태 및 이벤트 핸들러 연결 시 발생     |
  | **상태 및 이벤트 연결 시점** | 초기에 상태 및 이벤트 핸들러 연결 완료            | 하이드레이션 단계에서 상태 및 이벤트 핸들러 연결  | 하이드레이션 단계에서 상태 및 이벤트 핸들러 연결 | 하이드레이션 단계에서 상태 및 이벤트 핸들러 연결  |
  | **상태 변화 발생 시 처리**   | Virtual DOM에서 상태 기반으로 CSR처럼 동작        | Virtual DOM에서 상태 기반으로 CSR처럼 동작        | Virtual DOM에서 상태 기반으로 CSR처럼 동작       | Virtual DOM에서 상태 기반으로 CSR처럼 동작        |
  | **새 HTML 생성 여부**        | ❌ 상태 변화 시 새 HTML 생성 안 함                 | ❌ 상태 변화 시 새 HTML 생성 안 함                 | ❌ 상태 변화 시 새 HTML 생성 안 함                | ❌ 상태 변화 시 새 HTML 생성 안 함                 |
  | **SEO 반영 여부**            | ❌ 반영 어려움                                     | ✅ 서버에서 HTML 생성 시 반영 가능                 | ✅ 빌드 시 HTML 생성 시 반영 가능                 | ✅ 일정 주기마다 최신 HTML 제공 가능               |


#### (5) SSG 
- **SSG (Static Site Generation) : 빌드 타임에 모든 페이지를 미리 HTML로 생성**
  - SSR <-> SSG 차이
    - **SSR** : 사용자의 `요청`이 있을 때마다 `서버에서 페이지의 HTML을 생성`
    - **SSG** : `빌드 타임`에 미리 `모든 페이지를 HTML로 생성`
  - Pre-Rendering이 이루어짐
    - SSR과 달리 빌드 타임에 이루어짐
  - 장점 : 이미 만들어져 있는 사이트를 제공하여 빠른 로딩 시간을 보장 
    - 4가지 렌더링 기법 중 가장 빠름
  - 단점 : 미리 모든 HTML을 만들어놓기 때문에 `실시간 데이터가 지원되지 않음`

#### (6) ISR 
- **ISR (Incremental Static Regeneration, 점진적 정적 재생성) : 정적 페이지를 먼저 보여주고 필요에 따라 서버에서 페이지를 재생성하는 방식**
  - ISR = SSR + SSG 
    - SSG : 정적 페이지를 보여주기 
    - SSR : 필요에 따라 서버에서 페이지를 재생성하기
      - `설정한 주기만큼` 페이지를 생성
  - 장점
    - 정적 페이지를 먼저 제공하여 UX 향상
    - 콘텐츠가 변경되었을 때 서버에서 페이지를 재생성하여 최신 상태를 (그나마) 유지할 수 있음
      - 매초마다 실시간 데이터가 필요하진 않지만 그래도 실시간에 가까운 데이터를 보여줘야 할 때 유용 (예 : 블로그)

### 1-3. 렌더링 패턴 구현
- 원활한 테스팅을 위해 `dev 모드`가 아닌 `production mode`로 진행해야 함
  - dev 모드 : SSR처럼 동작
- production server 실행 방법
  - 빌드 : `yarn build`, `npm run build`
  - 실행 : `yarn dev`, `npm run dev`

#### 1. CSR 
1. `fetch`시 요청이 있을 때마다 지속해서 갱신
2. `use client;`가 존재
3. 클라이언트 측에서 `useEffect`, `useState`를 통해 데이터를 가져옴 
4. 순수 리액트를 사용하는 경우

#### 2. SSR 
- CSR과 동작 방식은 똑같지만 SEO 지원
1. `fetch` 시 요청이 있을 때마다 지속해서 갱신 (캐시된 데이터 취급 X)
2. 매번 서버로부터 최신 데이터를 가져옴 (cache:"no-cache" 옵션의 경우)
3. 컴포넌트 자체를 async 함수로 작성하여 서버에서 데이터를 가져옴 
  - `useEffect`, `useState`를 사용 X

#### 3. SSG 
1. `fetch` 시 아무리 새로고침을 하여도 동일한 페이지만 출력됨 (캐시 사용)
2. 한번 빌드 시, 모든 데이터가 정적으로 생성되어 페이지가 동일하게 출력됨
   - 빌드 시마다 데이터가 변경됨
3. 아무 옵션을 주지 않으면 `fetch`에 `force-cache`라는 옵션을 부여하는 것과 같음 (디폴트 값)
4. `force-cache` 옵션은 브라우저가 요청 시 캐시를 사용하도록 강제함 (최신 데이터를 가져오지 못함)

#### 4. ISR
1. `fetch` 시 주어진 시간에 한번씩 갱신
  - 방법 1. `fetch`에 `revalidate` 옵션 추가
  - 방법 2. `fetch`에 옵션을 주지 않고 페이지 컴포넌트에 직접 `revalidate` 설정
    - 컴포넌트 레벨에서 적용 불가능 
    - `page.tsx`, `layout.tsx`에서 가능

<br> <br>

## 2. [NextJS에서 다양한 렌더링 방식들은 서로 어떤 연관이 있을까? (SSG, SSR, ISR, CSR, Static(Pre), Dynamic) ](https://www.youtube.com/watch?v=I2La3ivhX_s)

### (1) 개요
- 아래 두 상황에서 SSR이 차이점이 있음 : 이름은 같지만 주목하는 바가 달라서 `의미가 다름`
  - SSG, `SSR`, ISR 
  - CSR <-> `SSR` 
- Static, Dynamic Rendering 개념까지 추가되면 헷갈리지 않기 위해 개념을 잘 알아야 함
- 구현 방식을 통해 멘탈 모델을 알 수 있음
  - App 라우터는 너무 추상화되어 있어 멘탈 모델을 알기가 어려움
  - Pages 라우터를 먼저 확인해야 멘탈 모델을 알기 쉽고, 구현 방식의 변화를 알 수 있음

### (2) SSG 구현 방식
- Static Rendering : 고정된 내용이 렌더링됨

#### 1. Pages 라우터에서 SSG 구현 방식
- Pages 라우터에서는 모두 페이지 단위로 개발됨
  - 예 : `Index.tsx` - Index 페이지 
- getStaticPaths를 통해 정적인 Path 정보를 넘겨줄 수 있음
  - 예 : Blog 페이지를 접속 => 블로그의 path를 확인 => id를 가져옴
- getStaticProps를 통해 정적인 Props를 넘겨줄 수 있음
  - 예 : Blog 페이지를 접속 => getStaticPaths를 통해 블로그의 path를 확인 => id를 가져옴 => 정적 데이터 id를 Props로 전달
- getStaticProps를 통해 Static Site를 생성할 수 있음 
  - 요청 전에 미리 렌더링 가능
  - 고정된 정적인 데이터를 보여주는 방식

<br>

- 예시 코드 : 블로그 게시글을 정적으로 렌더링 -  `pages/blog/[id].tsx`
```tsx
import { GetStaticPaths, GetStaticProps } from 'next';

interface Props {
  id: string;
  title: string;
}

export default function BlogPost({ id, title }: Props) {
  return (
    <div>
      <h1>{title}</h1>
      <p>Post ID: {id}</p>
    </div>
  );
}

// ✅ 어떤 경로를 생성할지 정의
export const getStaticPaths: GetStaticPaths = async () => {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map((post: any) => ({
    params: { id: post.id.toString() },
  }));

  return {
    paths, // 미리 생성할 경로 정의
    fallback: false, // 정의되지 않은 경로는 404 반환
  };
};

// ✅ 각 경로에 해당하는 데이터를 미리 가져와서 HTML 생성
export const getStaticProps: GetStaticProps = async ({ params }) => {
  const res = await fetch(`https://api.example.com/posts/${params?.id}`);
  const post = await res.json();

  return {
    props: {
      id: post.id,
      title: post.title,
    },
  };
};
```

#### 2. App 라우터에서 SSG 구현 방식
- React Server Component에서 SSG 구현 조건
  1. Dynamic APIs (cookies, headers ...)이 호출되지 않음
  2. fetch해온 Data가 캐시된 상태
     - 요청 전 미리 데이터를 알고 있어야 빌드 타임에 HTML을 미리 생성 가능

<br>

- 예시 : 블로그 게시글을 정적으로 렌더링 - `app/blog/[id]/page.tsx`
```tsx
import { notFound } from 'next/navigation';

interface Props {
  params: {
    id: string;
  };
}

export async function generateStaticParams() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return posts.map((post: any) => ({
    id: post.id.toString(),
  }));
}

async function getPost(id: string) {
  const res = await fetch(`https://api.example.com/posts/${id}`);
  if (!res.ok) {
    notFound();
  }
  return res.json();
}

export default async function BlogPost({ params }: Props) {
  const post = await getPost(params.id);

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

```

### (3) SSR 구현 방식
- Dynamic Rendering : 고정되지 않고 클라이언트마다 변경됨
  - Dynamic APIs (cookies, headers ...)가 호출됨
  - 예 : 추천 아이템

#### 1. Pages 라우터에서 SSR 구현 방식
- getServersideProps를 통해 Props를 가져옴 
  - 예 : `Index.tsx`에 Props를 전달
- getStaticProps <-> getServersideProps 차이점 
  - getStaticProps : 빌드 타임에 호출됨 (요청 전)
  - getServersideProps : 런타임에 호출됨 (매 요청 시)
    - 런타임 : 실제로 실행되는 시점, 브라우저에서 해당 코드가 실행될 때

<br>

- 예시 : 사용자 상태나 요청별로 데이터가 달라지는 경우 - `pages/blog/[id].tsx`
```tsx
import { GetServerSideProps } from 'next';

interface Props {
  id: string;
  title: string;
}

export default function BlogPost({ id, title }: Props) {
  return (
    <div>
      <h1>{title}</h1>
      <p>Post ID: {id}</p>
    </div>
  );
}

// ✅ 요청이 발생할 때마다 동적으로 HTML 생성
export const getServerSideProps: GetServerSideProps = async ({ params }) => {
  const res = await fetch(`https://api.example.com/posts/${params?.id}`);
  const post = await res.json();

  return {
    props: {
      id: post.id,
      title: post.title,
    },
  };
};

```

#### 2. App 라우터에서 SSR 구현 방식
- App 라우터에서 SSR 구현 조건 
  - 둘 중 하나만 충족되더라도 SSR 방식으로 구현됨
  1. Dynamic APIs (cookies, headers ...)를 호출할 때
     - Dynamic Rendering 이자 SSR을 결정하는 요소
  2. 캐시된 데이터가 없어 새롭게 바뀐 데이터를 가져올 때

- 예시 : 사용자 상태나 요청별로 데이터가 달라지는 경우 - `app/blog/[id]/page.tsx`
```tsx
interface Props {
  params: {
    id: string;
  };
}

async function getPost(id: string) {
  const res = await fetch(`https://api.example.com/posts/${id}`, {
    cache: 'no-store', // SSR 모드 설정
  });
  if (!res.ok) {
    throw new Error('Failed to fetch post');
  }
  return res.json();
}

export default async function BlogPost({ params }: Props) {
  const post = await getPost(params.id);

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

```

### (4) CSR <-> SSR / SSG, SSR, ISG 맥락에서 SSR 의미
#### 1. CSR <-> SSR
- 렌더링 위치에 따른 분류
- Client <-> Server 각각에서 렌더링된다는 차이를 의미 

#### 2. SSG, SSR, ISG 맥락
- 렌더링 시점의 관점
- 동일하게 Server Side Rendering이지만 요청 시마다 Dynamic하게 렌더링함

### (5) ISR (Incremental Static Regeneration)
- Static
  - 기본적으로 정적인 렌더링
  - 정적인 페이지(SSG)의 단점 : 바뀔 수 없음
- `Re`generation 
  - 캐시된 데이터를 `갱신`함으로 정적인 페이지의 `단점을 극복`
  - cache => revalidate
    - 특정한 주기마다 서버로 검증을 다시 요청하여 SSG를 다시 만들게 함

#### 1. Pages 라우터에서 ISR 구현 방식
- SSG 구현 방식 + 옵션 cache 값 설정

- 예시 : 일정 시간마다 정적 페이지 갱신 - `pages/blog/[id].tsx`
```tsx
import { GetStaticProps, GetStaticPaths } from 'next';

export default function BlogPost({ post }: any) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map((post: any) => ({
    params: { id: post.id.toString() },
  }));

  return { paths, fallback: 'blocking' };
}

export async function getStaticProps({ params }: any) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();

  return {
    props: { post },
    revalidate: 10, // 10초마다 새로운 HTML 생성
  };
}

```

#### 2. App 라우터에서 ISR 구현 방식
- `export const revalidate = 60`와 같이 작성되어있으면 revalidate이 발생한다고 판단함
  - Pages 라우터에서 옵션 cache 값 설정한 것과 동일하게 인식

- 예시 : 일정 시간마다 정적 페이지 갱신 - `app/blog/[id]/page.tsx`
```tsx
export const revalidate = 10; // 10초마다 갱신

async function getPost(id: string) {
  const res = await fetch(`https://api.example.com/posts/${id}`);
  return res.json();
}

export default async function BlogPost({ params }: any) {
  const post = await getPost(params.id);

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

```

## 3. Next 렌더링 과정 요약
1. 브라우저 렌더링 단계
- HTML/CSS 파싱 → DOM/CSSOM 생성 → 렌더 트리 생성 → 레이아웃 → 페인트

2. Next.js에서 SSR/SSG/ISR 발생 시
- 서버에서 HTML 생성 → 브라우저로 전달 → DOM 생성 (초기 렌더링)

3. React 렌더링 단계
- Virtual DOM 생성 → Diffing → Commit 단계에서 실제 DOM 반영
- 상태 변화 발생 시 Render → Commit 반복

4. 상태 변화 발생 시
- CSR과 동일하게 상태 기반 렌더링
- React의 Virtual DOM 기반 렌더링 구조로 동작

## Q&A 
### Q1. 브라우저, 리액트, 넥스트 렌더링 과정을 연결해서 설명해주세요.
A1. Next.js는 SSR, SSG, ISR, CSR 방식을 통해 HTML 제공 시점을 결정합니다.
SSR은 매 요청 시, SSG는 빌드 시, ISR은 일정 주기마다 새로운 HTML을 생성하고 브라우저에 반환합니다.
CSR은 빈 HTML을 반환하고 브라우저에서 상태 및 이벤트 핸들러를 연결합니다.
React는 Render 단계에서 Virtual DOM을 생성하고, Diffing 후 Commit 단계에서 실제 DOM에 반영합니다.
이후 상태 변화 발생 시 Render → Commit 단계가 반복되면서 CSR처럼 상태 기반으로 업데이트됩니다.

### Q2. Next의 렌더링 방식의 종류와 각 렌더링 방식의 차이점을 설명해주세요.
A2. Next.js는 SSR, SSG, ISR, CSR 방식에 따라 HTML 제공 시점과 렌더링 방식이 다릅니다.

**SSR(Server Side Rendering)**은 매 요청 시 서버에서 getServerSideProps()를 통해 HTML이 생성됩니다. 따라서 서버에서 렌더링이 발생하고 브라우저에서 초기 렌더링 후 하이드레이션 단계에서 상태 및 이벤트 핸들러가 연결됩니다. 매 요청 시 새로운 HTML이 생성되므로 최신 데이터가 반영됩니다.

**SSG(Static Site Generation)**은 빌드 시 서버에서 getStaticProps()를 통해 HTML이 생성됩니다. 브라우저는 생성된 HTML을 받아서 렌더링하고 이후 하이드레이션 단계에서 상태 및 이벤트 핸들러가 연결됩니다. 빌드 시점에서 HTML이 고정되기 때문에 실시간 데이터 반영이 어렵지만 성능이 우수하고 SEO가 강화됩니다.

**ISR(Incremental Static Regeneration)**은 SSG와 비슷하지만 revalidate 옵션을 통해 일정 시간마다 새로운 HTML이 생성됩니다. 브라우저는 새로운 HTML을 받아서 렌더링한 후 하이드레이션에서 상태 및 이벤트 핸들러가 연결됩니다. 따라서 성능이 우수하면서 최신 데이터 반영도 가능합니다.

**CSR(Client Side Rendering)**은 서버에서 빈 HTML이 반환됩니다. 브라우저에서 JS 파일이 로드된 후 React가 Virtual DOM을 생성하고 상태 및 이벤트 핸들러를 연결하면서 초기 렌더링이 발생합니다. 이후 상태 변화 발생 시 Render → Commit 단계에서 Virtual DOM에서 diffing 후 DOM이 업데이트됩니다. CSR은 서버 부하가 적고 유연한 상태 관리가 가능하지만 초기 로딩 속도가 느리고 SEO 성능이 떨어질 수 있습니다.

SSR은 매 요청마다 HTML이 생성되어 최신 데이터가 반영되며, SSG는 빌드 시 HTML이 생성되어 성능이 우수하지만 실시간 데이터 반영이 어렵습니다. ISR은 성능과 최신 데이터 반영의 장점을 모두 가집니다. CSR은 상태 기반 렌더링에 유리하고 서버 부하가 적지만 SEO 성능이 떨어집니다. 따라서 최신 데이터가 필요한 경우 SSR을, 성능이 중요한 경우 SSG를, 성능과 최신 데이터 반영 모두 필요한 경우 ISR을, 상태 기반 렌더링이 중요한 경우 CSR을 선택합니다.

### Q3. ISR에서 상태 변화는 즉시 반영되지만 HTML은 변경되지 않는 이유를 설명해주세요.
A3. ISR에서 상태 변화는 하이드레이션 이후 React에서 상태 기반으로 즉시 반영됩니다. 하이드레이션이 발생하면 서버에서 제공한 HTML을 React가 Virtual DOM으로 변환하고, 상태 및 이벤트 핸들러를 연결합니다. 이후 상태가 변하면 React는 Render 단계에서 새로운 Virtual DOM을 생성하고, Diffing 과정을 통해 변화된 부분만 Commit 단계에서 실제 DOM에 반영합니다. 즉, 상태 변화가 발생하면 CSR처럼 React에서 상태 기반으로 화면이 즉시 바뀌게 됩니다. 하지만 ISR에서 HTML 자체는 빌드 시점에서 고정되기 때문에 상태 변화가 발생해도 HTML은 변경되지 않습니다.
HTML 자체가 변경되려면 ISR에서 revalidate 값이 설정되어 있어야 하며, revalidate 주기가 도래했을 때 서버에서 새로운 HTML을 생성합니다. 따라서 상태 변화는 클라이언트에서 즉시 반영되지만, HTML 자체는 revalidate 주기 이후에만 새로 생성됩니다. 이렇게 하면 정적 사이트의 빠른 성능과 상태 변화에 따른 즉각적인 반응이 동시에 가능해집니다.

### Q4. 사용자는 항상 넥스트를 사용할 때 리액트보다 빠르게 느끼나요? 관련해서 설명해주세요. 
A4. Next.js는 기본적으로 SSR, SSG, ISR 같은 서버 렌더링 기법을 통해 성능을 강화하기 때문에 React의 CSR보다 일반적으로 빠르게 느껴질 수 있습니다. 그러나 SSR의 경우 서버 성능에 따라 응답 속도가 느려질 수 있고, 초기 렌더링은 더 빠르게 느껴지나 하이드레이션 이후에는 리액트와 동일하게 동작하기 때문에 별로 차이가 없게 느껴질 수 있습니다.

### Q5. SSR에서 상태 변화가 발생하면 어떻게 처리되나요? SSR에서 상태 기반 렌더링이 가능한가요?
A5. SSR에서는 상태 변화가 발생하면 React에서 CSR처럼 상태 기반으로 즉시 업데이트됩니다.
SSR은 매 요청 시 서버에서 getServerSideProps()에서 HTML을 생성하고 브라우저에 반환합니다. 브라우저에서 HTML이 렌더링되면 React가 Virtual DOM을 생성하고 상태 및 이벤트 핸들러를 연결하기 위해 하이드레이션이 발생합니다. 하이드레이션 이후에는 상태 변화 발생 시 React에서 CSR처럼 상태 기반으로 즉시 반영됩니다.
상태 변화가 발생하면 React에서 Render 단계에서 새로운 Virtual DOM을 생성하고, Diffing 과정을 통해 변화된 부분을 찾아 Commit 단계에서 실제 DOM에 반영합니다. 따라서 상태 변화는 CSR처럼 즉시 반영되지만, SSR은 HTML이 매 요청 시 새로 생성되기 때문에 새로운 HTML이 필요하면 서버에서 새로운 HTML이 다시 생성됩니다.
결국 상태 변화는 CSR처럼 즉시 반영되지만, HTML 자체는 상태 변화가 아니라 매 요청 시 서버에서 갱신됩니다. 따라서 상태 변화와 HTML 생성은 독립적으로 작동합니다.