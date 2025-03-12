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


- 🌟 **초기 렌더링 vs 하이드레이션 차이**  

  | 구분                   | 초기 렌더링 (Initial Rendering)                  | 하이드레이션 (Hydration)                  |
  | ---------------------- | ------------------------------------------------ | ----------------------------------------- |
  | **정의**               | DOM이 처음으로 생성되고 사용자에게 표시되는 과정 | 기존 DOM에 상태 및 이벤트를 연결하는 과정 |
  | **발생 위치**          | 서버 또는 클라이언트                             | 클라이언트에서만 발생                     |
  | **SSR에서 발생 여부**  | ✅ 발생 (서버에서 수행)                           | ✅ 발생 (클라이언트에서 수행)              |
  | **CSR에서 발생 여부**  | ✅ 발생 (클라이언트에서 수행)                     | ❌ 발생하지 않음                           |
  | **주체**               | 서버 (SSR), 클라이언트 (CSR)                     | 클라이언트 (React 상태 연결)              |
  | **이벤트 핸들러 연결** | ❌ 이벤트 핸들러 없음                             | ✅ 이벤트 핸들러 연결                      |
  | **DOM 생성 여부**      | ✅ CSR에서는 React가 DOM 생성                     | ❌ 기존 DOM 재활용                         |
  | **Hydration Mismatch** | ❌ 발생하지 않음                                  | ✅ 발생 시 DOM 삭제 후 재생성 가능         |

<br>


- 🌟 **CSR vs SSR에서의 초기 렌더링과 하이드레이션 비교**
  | 구분                         | CSR                                    | SSR                                             |
  | ---------------------------- | -------------------------------------- | ----------------------------------------------- |
  | **HTML 반환 시점**           | 빈 HTML 반환 (`<div id="root"></div>`) | 완성된 HTML 반환                                |
  | **초기 렌더링 발생 시점**    | JS 파일이 로딩된 후 React가 DOM 생성   | 서버에서 생성된 HTML이 브라우저에서 렌더링될 때 |
  | **상태 및 이벤트 연결 시점** | 초기 렌더링 시점에서 바로 연결         | 하이드레이션 단계에서 연결                      |
  | **DOM 생성 주체**            | React가 직접 생성                      | 서버에서 생성 후 브라우저에서 렌더링            |
  | **하이드레이션 발생 여부**   | ❌ 발생하지 않음                        | ✅ 하이드레이션 발생 (기존 DOM 재활용)           |

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