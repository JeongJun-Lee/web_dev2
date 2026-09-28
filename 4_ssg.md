# 제4장: Vite-SSG — 빠르고 검색에 잘 걸리는 앱 만들기

> 🎉 이 장의 작은 성공: `npm run build`를 실행하면 `dist/` 폴더에 여러 개의 완성된 HTML 파일이 뚝딱 만들어진다!

3장에서 TypeScript로 코드에 안전망을 씌웠습니다. 이번 장에서는 앱의 _성&#xB2A5;_&#xACFC; _검색 노&#xCD9C;_&#xC774;라는 두 가지 현실적인 문제를 해결합니다.

이 장을 시작하기 전에 잠깐, 웹 페이지가 브라우저 화면에 표시되기까지 어떤 과정을 거치는지 큰 그림을 먼저 이해하면 나머지 내용이 훨씬 쉽게 들어옵니다.

## 0. 웹 렌더링 방식의 역사 — "어디서 HTML을 만드느냐"의 문제

웹 개발의 역사는 "누가, 언제 HTML을 만드는가"에 대한 고민의 역사라고 해도 과언이 아닙니다. 크게 세 가지 시대를 거쳐 왔습니다.

### 0-1. 전통적인 서버 사이드 렌더링(SSR) 시대

![](.gitbook/assets/ssr_traditional.png)

위 그림을 보세요. 클라이언트(브라우저)가 서버에 요청을 보내면, **서버** 쪽의 PHP(또는 JSP, Ruby on Rails 등)가 데이터베이스에서 데이터를 가져와 HTML을 완성한 뒤 클라이언트에게 보내줍니다.

**전통 SSR의 동작 흐름:**

```
①  브라우저 → "index.php 줘!" → 서버 (Apache/Nginx)
②  서버 → PHP 실행 → DB에서 데이터 조회
③  PHP → 데이터를 HTML에 끼워넣어 완성
④  서버 → 완성된 HTML → 브라우저
⑤  사용자가 링크 클릭 → 다시 ①부터 반복 (페이지 전체 새로고침)
```

- **장점**: 서버가 완성된 HTML을 보내주므로, 구글 봇이 내용을 바로 읽을 수 있습니다(SEO — 검색 엔진 최적화, 쉽게 말해 구글 검색 결과 상위에 잘 노출되는 것 — 에 유리). 첫 화면도 빠릅니다.
- **단점**: 버튼 하나 클릭해도 페이지 전체를 다시 불러와야 합니다. 이때 화면이 깜빡이는 현상(새로고침)이 발생하고, 서버에 매번 부하가 걸립니다.

> 💡 **실제 예**: 네이버 블로그의 초창기 버전, 옛날 쇼핑몰 사이트들이 이 방식으로 만들어졌습니다.

---

### 0-2. CSR(Client Side Rendering)과 SPA(Single Page Application) 시대

![](.gitbook/assets/csr_spa.png)

스마트폰이 대중화되고 웹 앱이 복잡해지면서 **"페이지 새로고침 없이 동작하는 앱"** 에 대한 수요가 폭발했습니다. 이를 해결한 것이 Vue.js, React, Angular 같은 JavaScript 프레임워크와 CSR 방식입니다.

**CSR + SPA의 동작 흐름:**

```
①  브라우저 → "index.html 줘!" → 서버
②  서버 → 텅 빈 index.html + main.js → 브라우저
③  브라우저 → main.js 다운로드 & 실행
④  JavaScript가 브라우저 안에서 HTML을 직접 생성(렌더링)
⑤  사용자가 링크 클릭 → JavaScript가 화면 일부만 교체 (새로고침 없음!)
⑥  필요한 데이터 → 서버에 JSON으로 요청 → 받아서 화면 갱신
```

위 그림에서 보듯이, 클라이언트 쪽에 Vue.js / Angular / React / Svelte 같은 프레임워크가, 서버 쪽에는 Node.js, Express, Nuxt, NestJS, Next.js 같은 JavaScript 기반 서버 기술들이 등장했습니다. 데이터베이스도 MongoDB, PostgreSQL, MariaDB, SQLite 등 다양해졌습니다.

- **장점**: 페이지 전환이 부드럽고 앱처럼 동작합니다. 서버와는 HTML이 아닌 JSON 데이터만 주고받아 훨씬 효율적입니다.
- **단점**: 처음 페이지를 열면 JavaScript 파일을 다운받고 실행하는 동안 **빈 화면**이 보입니다. 구글 봇도 이 빈 화면만 보게 됩니다. → 바로 지금 우리 앱의 문제입니다!

> 💡 **실제 예**: 현재 Gmail, Google Docs, 트위터(X) 등 대부분의 현대 웹 앱이 SPA 방식을 사용합니다.

---

이제 위 두 방식의 문제를 정확히 이해했으니, 본론으로 들어가겠습니다.

## 1. 지금 앱의 두 가지 문제

2장에서 만든 Vue 앱은 CSR/SPA 방식입니다. 이를 배포하면 어떤 일이 일어날까요?

### 문제 1: 첫 화면이 느리다 (흰 화면 문제)

브라우저가 서버에서 받은 HTML 파일을 열어보면 이렇습니다:

```html
<!-- samarkand-local-mate의 실제 index.html -->
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <meta content="width=device-width, initial-scale=1.0" name="viewport" />
    <title>Samarkand Local Mate | Find Your Local Guide</title>
    <link
      href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&family=Playfair+Display:wght@600;700&display=swap"
      rel="stylesheet"
    />
  </head>
  <body
    class="bg-background text-on-background font-body-md antialiased overflow-x-hidden"
  >
    <div id="app"></div>
    <!-- ← 텅 비어 있음! 브라우저가 아래 스크립트를 다운받아 실행하기 전까지는 빈 화면입니다 -->
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

`<div id="app">` 이 텅 비어 있습니다. 실제 배포 환경에서는 브라우저가 `main.js`(Vite가 `main.ts`를 번들링하여 생성한 자바스크립트 파일)를 내려받고, 해석하고, 실행해야 비로소 화면이 그려집니다.

이 과정을 시간대별로 표현하면:

```
0ms    : 브라우저가 index.html 받음 → 화면은 여전히 비어 있음 😐
300ms  : main.js 파일 다운로드 완료
800ms  : JavaScript 파싱 & Vue 앱 초기화 완료
1200ms : 화면에 콘텐츠 표시 🎉
```

사용자는 0ms\~1200ms 동안 **하얀 빈 화면**을 봅니다. 모바일이나 느린 인터넷 환경에서는 이 시간이 35초까지 늘어날 수 있습니다. 사용자 연구에 따르면 3초 이상 기다리면 약 40%의 사용자가 페이지를 떠납니다.

### 문제 2: 구글이 빈 화면만 본다 (SEO 문제)

구글 검색 봇이 우리 사이트를 방문합니다. 봇은 HTML을 읽어서 페이지 내용을 파악하는데, `<div id="app"></div>` — 아무것도 없습니다. JavaScript를 실행해서 기다려주지 않거든요.

```
구글 봇이 본 것:
  <div id="app"></div>
        ↓
  "사마르칸트 투어나 가이드 정보가 하나도 없네?"
        ↓
  검색 결과 하위 노출 또는 색인 제외
```

결과적으로 구글은 이 페이지가 비어 있다고 판단하고 검색 결과에서 낮은 순위를 부여하거나 아예 색인하지 않습니다.

우리 앱에는 _"사마르칸트에서 나만의 로컬 가이드를 만나보세요"_, _"역사 · 미식 · 사진 맞춤 가이드"_, _"알리셰르 (Alisher) 역사 전문 가이드"_ 같은 훌륭한 콘텐츠가 가득하지만, 구글 검색 봇은 이를 전혀 볼 수 없습니다. 사마르칸트 투어 앱인데 "사마르칸트 투어", "사마르칸트 로컬 가이드"로 검색해도 안 나온다면 여행자들이 우리 서비스를 찾아올 방법이 없겠죠.

{% hint style="info" %}
**💡 SPA vs SSG — 택배로 비유하기**

**SPA(Single Page Application) 방식**: 빈 택배 상자를 먼저 보내고, 상자 안에 '조립 설명서(JavaScript)'를 넣어둡니다. 받는 사람(브라우저)이 설명서를 읽고 직접 내용물을 조립해야 합니다. → 조립 시간만큼 기다려야 함

**SSG(Static Site Generation) 방식**: 공장(빌드 서버)에서 내용물을 미리 다 조립한 완성품을 보냅니다. 받는 사람은 상자를 열자마자 바로 사용할 수 있습니다. → 기다릴 필요 없음
{% endhint %}

## 2. 해결책: Vite-SSG

**Vite-SSG**는 두 세계의 장점을 합칩니다.

- **개발 중**: 빠르고 유연한 Vue SPA처럼 편하게 작업
- **배포 시**: 각 페이지를 완성된 HTML 파일로 미리 구워서 배포

"SSG(Static Site Generation)"는 "정적 사이트 생성"으로, **빌드 시점에 JavaScript를 미리 실행해서 완성된 HTML을 파일로 저장해두는 기술**입니다. 사용자가 페이지를 요청하면 서버는 이미 만들어진 HTML 파일을 그냥 전달하기만 하면 됩니다.

### 세 가지 방식 한눈에 비교

| 방식      | HTML을 만드는 시점      | 만드는 장소   | 첫 로딩 속도 | SEO      | 서버 필요 여부         |
| --------- | ----------------------- | ------------- | ------------ | -------- | ---------------------- |
| 전통 SSR  | 사용자 요청마다         | 서버          | 빠름         | 강함     | 필요 (항상 실행)       |
| SPA (CSR) | 브라우저에서 JS 실행 후 | 브라우저      | 느림         | 취약     | 불필요 (정적 파일)     |
| **SSG**   | **빌드 시 한 번**       | **빌드 서버** | **빠름**     | **강함** | **불필요 (정적 파일)** |

SSG는 전통 SSR처럼 빠른 첫 로딩과 SEO를 제공하면서도, 서버 없이 정적 파일만으로 배포할 수 있어서 1권에서 배운 **Cloudflare Pages** 같은 서비스에 그대로 올릴 수 있습니다. 가장 좋은 부분만 쏙 뽑은 방식이라고 할 수 있습니다.

{% hint style="warning" %}
**⚠️ SSG의 한계: 실시간 데이터에는 주의**

SSG는 "빌드 시점에 미리 만들어두는" 방식이므로, 실시간으로 자주 바뀌는 데이터(예: 실시간 주식 가격, 채팅 메시지)에는 적합하지 않습니다.

현재 수준의 투어 앱의 경우, 가이드 정보나 투어 상품 설명은 자주 바뀌지 않으므로 현재 수준의 SSG를 적용하면 충분합니다. 새로운 상품을 추가하면 다시 빌드해서 배포하면 됩니다.
{% endhint %}

### 💡 더 나아가기: SSG + 실시간 API 하이브리드

**"나중에 가이드가 직접 자기 소개를 수시로 수정하는 기능이 필요해진다면 어떻게 할까요?"**

걱정하지 마세요 — 이럴 때는 **SSG + 실시간 API 하이브리드** 방식으로 자연스럽게 발전시킬 수 있습니다.

하이브리드의 기본 원리는 **"HTML 껍데기(또는 이전 데이터)를 먼저 보여주고, 실시간 최신 데이터를 뒤이어 끼워넣는 것"**입니다. 실무에서는 보통 두 가지 패턴을 사용합니다:

- **패턴 1: 어제 구워둔 데이터 먼저 보여주고 조용히 교체하기 (SWR: Stale-While-Revalidate 패턴 — 투어 앱 추천)**
  - 사용자가 접속하면 빌드 시점에 구워둔 가이드 소개글이 0.1초 만에 즉시 뜹니다 (사용자는 기다림 없이 바로 글을 읽기 시작합니다).
  - 화면이 뜨자마자 Vue가 백그라운드에서 조용히 백엔드 API(`/api/guides/alisher`)를 호출해 최신 정보를 확인합니다.
  - 가이드가 방금 소개글을 수정했다면, 해당 텍스트 부분만 0.3초 뒤에 최신 내용으로 부드럽게 바뀝니다. 로딩 스피너도, 화면 깜빡임도 없습니다.
- **패턴 2: 골격 껍데기(스켈레톤 UI) 먼저 보여주기 (토스, 유튜브 방식)**
  - 회색 박스로 된 뼈대(스켈레톤) HTML을 먼저 초고속으로 보여주고, API 응답이 오자마자 실제 데이터로 채웁니다.

```
① SSG가 미리 구워둔 HTML이 즉시 화면에 표시됨 (초고속 첫 화면 + SEO 완벽 유지)
② 화면이 뜬 직후, Vue가 백엔드 API를 호출해 혹시 변경된 최신 정보가 있는지 확인
③ 변경 사항이 있다면 화면의 해당 데이터만 최신으로 부드럽게 교체
```

{% hint style="info" %}
**💡 헷갈리기 쉬운 개념: Hydration vs 하이브리드 실시간 API**

- **Hydration (엔진 레벨의 전선 연결)**:
  새로 입주한 아파트 건물(HTML)에 **전기 배선과 수도관(자바스크립트)**을 연결하는 일입니다. 전기가 통해야 버튼을 눌렀을 때 모달이 열립니다.
- **하이브리드 실시간 API (데이터 최신화 전략)**:
  식탁에 **매일 아침 배달되는 신선한 우유(실시간 데이터)**를 채워 넣는 일입니다. 어제 사둔 우유를 꺼내 마시고 있다가, 문앞에 새 우유가 도착하면 새 우유로 바꿔 마시는 것과 같습니다.
  {% endhint %}

사용자는 하얀 로딩 화면 없이 즉시 콘텐츠를 소비하고, 0.2~0.3초 뒤 최신 정보로 업데이트됩니다. SSG의 속도와 SEO 장점을 100% 누리면서 실시간 데이터도 함께 다룰 수 있는 현대 웹 개발의 표준 아키텍처입니다.

## 3. 프롬프팅: Vite-SSG 적용하기

> 💬 "이 웹앱의 초기 로딩이 느린데, `vite-ssg`를 도입해서 빌드할 때 정적 HTML을 미리 뽑아내게 설정해줘. 라우터 설정도 같이 수정해서 각 투어 상세 페이지가 별도의 HTML로 생성되게 해."

AI가 수행하는 작업 순서를 하나씩 살펴보겠습니다.

### 1단계: `vite-ssg` 패키지 설치

```bash
npm install -D vite-ssg
```

패키지 설치는 항상 첫 번째 단계입니다. `npm install -D`는 빌드 도구인 `vite-ssg`를 개발 의존성(`devDependencies`)으로 내려받고, `package.json`에 기록합니다.

### 2단계: `main.ts` 수정 — SSG용 export 추가

`samarkand-local-mate`의 기존 SPA 방식 `main.ts`가 어떻게 생겼는지 먼저 보고, 무엇이 달라지는지 비교해봅니다:

```typescript
// 기존 main.ts — SPA 방식 (samarkand-local-mate/src/main.ts)
import { createApp } from "vue";
import App from "./App.vue";
import "./assets/main.css";

createApp(App).mount("#app");
//              ↑ 브라우저 DOM의 #app 요소에 앱을 즉시 마운트
//                브라우저 환경에서만 실행 가능!
```

SSG 방식으로 수정:

```typescript
// 수정된 main.ts — SSG 방식
import { ViteSSG } from "vite-ssg";
import App from "./App.vue";
import "./assets/main.css";
import { routes } from "./router"; // 라우트 정보 가져오기

// createApp 대신 ViteSSG를 export
export const createApp = ViteSSG(App, { routes });
```

**무엇이 달라졌나요?**

| 항목          | 기존 SPA                     | SSG 방식                 |
| ------------- | ---------------------------- | ------------------------ |
| 가져오는 함수 | `createApp` (vue에서)        | `ViteSSG` (vite-ssg에서) |
| 동작          | `.mount('#app')` — 즉시 실행 | `export` — 내보내기만 함 |
| 실행 주체     | 브라우저                     | 빌드 도구 (Vite-SSG)     |

변경의 핵심: `createApp(...).mount()` 대신 `ViteSSG(...)`를 `export`합니다. 빌드 시 Vite-SSG가 이 함수를 가져가서 각 라우트를 미리 방문하고 완성된 HTML을 생성합니다. 마치 "나중에 필요할 때 가져다 쓰세요"라고 완성 레시피를 내놓는 것과 같습니다.

### 3단계: `vite.config.ts` 확인

```typescript
// vite.config.ts
import vue from "@vitejs/plugin-vue";
import path from "path";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  // vite-ssg는 별도 복잡한 플러그인 설정 없이 main.ts의 createApp export만으로 작동합니다
});
```

`vite.config.ts`는 Vite 빌드 도구의 설정 파일입니다. 다행히 Vite-SSG는 이 파일을 크게 건드리지 않아도 됩니다. `main.ts`의 `export const createApp` 패턴만 있으면 자동으로 SSG 모드로 빌드합니다.

### 4단계: `package.json` — 빌드 스크립트 확인

```json
{
  "name": "samarkand-local-mate",
  "scripts": {
    "dev": "vite --port=3000 --host=0.0.0.0",
    "build": "vite-ssg build",
    "preview": "vite preview"
  }
}
```

`"build"` 스크립트가 `vite build`에서 `vite-ssg build`로 변경됩니다. 이것이 핵심입니다. `npm run build`를 실행하면 일반 Vite 빌드가 아니라 Vite-SSG가 빌드를 담당합니다.

**각 스크립트의 역할:**

- `npm run dev`: 개발 서버 실행 (파일 변경 시 자동 새로고침, 포트 3000)
- `npm run build`: 배포용 정적 HTML 파일 생성 (`dist/` 폴더에 결과물)
- `npm run preview`: 빌드 결과물을 로컬에서 미리보기

## 4. 귀납적 이해: 빌드 결과물로 SSG 원리 발견하기

`npm run build`를 실행한 후 생성된 `dist/` 폴더를 IDE에서 열어봅니다.

```
dist/
├── index.html              ← 메인 랜딩 페이지 (HeroSection, 매칭 폼 등 완성된 HTML!)
├── guides/
│   ├── index.html          ← 가이드 목록 페이지
│   ├── alisher/
│   │   └── index.html      ← 알리셰르 가이드 상세 페이지
│   ├── dilshod/
│   │   └── index.html      ← 딜쇼드 가이드 상세 페이지
│   ├── nigora/
│   │   └── index.html      ← 니고라 가이드 상세 페이지
│   └── jamshid/
│       └── index.html      ← 잠시드 가이드 상세 페이지
└── assets/
    ├── main.js             ← 번들된 JavaScript (Hydration용)
    └── main.css            ← 사마르칸트 테마 Tailwind CSS
```

**"Vue SPA인데 왜 여러 개의 `.html` 파일이 나왔지?"**

SSG가 빌드 시점에 각 라우트를 _미리 방&#xBB38;_&#xD574;서 HTML 스냅샷을 찍어두는 방식이기 때문입니다. 브라우저가 `/guides/alisher`에 접속하면, JavaScript가 실행되기도 전에 이미 완성된 HTML이 전달됩니다.

**빌드 과정을 단계별로 상상해보면:**

```
Vite-SSG 빌드 프로세스:
  ① routes 배열에서 '/' 발견 → 가상 브라우저로 접속 → HTML 스냅샷 → index.html 저장
  ② routes 배열에서 '/guides' 발견 → 가상 브라우저로 접속 → HTML 스냅샷 → guides/index.html 저장
  ③ routes 배열에서 '/guides/alisher' 발견 → 가상 브라우저로 접속 → HTML 스냅샷 → guides/alisher/index.html 저장
  ④ routes 배열에서 '/guides/dilshod' 발견 → 가상 브라우저로 접속 → HTML 스냅샷 → guides/dilshod/index.html 저장
  ... (모든 라우트 반복)
  ⑤ JavaScript와 CSS를 최적화해서 assets/ 폴더에 저장
  빌드 완료! 🎉
```

각 HTML 파일을 열어보면 텅 빈 `<div id="app">` 대신 실제 콘텐츠가 들어있는 것을 확인할 수 있습니다:

```html
<!-- dist/guides/alisher/index.html — SSG가 미리 렌더링한 실제 결과 -->
<div id="app">
  <nav class="bg-surface/80 shadow-sm sticky top-0 backdrop-blur-md z-50">
    <a class="font-bold text-[#C8953C]" href="/">Samarkand Local Mate</a>
  </nav>
  <main class="max-w-4xl mx-auto p-8">
    <div class="flex items-center space-x-4 mb-6">
      <div
        class="w-16 h-16 rounded-full bg-amber-100 flex items-center justify-center text-amber-600 font-bold text-2xl"
      >
        알
      </div>
      <div>
        <h1 class="text-3xl font-bold text-slate-800">알리셰르 (Alisher)</h1>
        <p class="text-amber-600 font-medium">
          역사 및 고건축 전문 가이드 · ⭐ 4.9
        </p>
      </div>
    </div>
    <p class="text-slate-600 leading-relaxed">
      사마르칸트 국립대 역사학과 출신으로 레기스탄과 샤히진다의 숨겨진 역사를
      깊이 있게 전달합니다.
    </p>
    <div class="mt-4 text-sm text-slate-500">
      구사 가능 언어: 한국어, 우즈베크어, 러시아어
    </div>
  </main>
</div>
```

구글 봇이 이 파일을 읽으면, 내용이 가득한 HTML을 완전히 파악할 수 있습니다. 더 이상 "빈 페이지" 문제가 없습니다!

### 💡 Hydration — 정적 HTML에 생명을 불어넣는 마법

SSG가 만든 완성된 HTML 파일을 브라우저가 받으면, 사용자는 기다림 없이 화면을 즉시 볼 수 있습니다. 그 후 백그라운드에서 `main.js`가 로드되면, Vue가 이미 그려진 HTML을 분석하여 버튼 클릭이나 폼 입력 같은 **인터랙션(상호작용) 기능을 활성화**합니다. 이 과정을 **Hydration(수분 공급)**이라고 부릅니다.

```
1. 브라우저가 HTML 받음      → 즉시 화면에 콘텐츠 표시 (빠름! 👀)
2. main.js 다운로드 & 실행  → Vue가 기존 HTML에 이벤트 연결 (Hydration)
3. Hydration 완료         → 버튼 클릭, 모달 열기 등 인터랙션 활성화 🖱️
```

**왜 "Hydration(수분 공급)"이라고 부를까요?**
말라있는 건조한 정적 HTML에 JavaScript라는 "생명수"를 부어주면 살아 숨 쉬게 된다는 비유입니다. 정적 HTML은 눈으로 볼 수는 있지만 버튼을 눌러도 반응이 없는 상태이고, Hydration을 거치고 나면 비로소 사용자와 소통하는 살아있는 웹 앱이 됩니다.

#### Q. 개발자가 직접 Hydration 코드를 작성해야 하나요?

**전혀 아닙니다!** `main.ts`에 작성한 `export const createApp = ViteSSG(App, { routes })` 한 줄 덕분에 브라우저에서 자동으로 일어납니다.

일반 SPA와 SSG의 마운트 방식을 비교해 보면 원리가 한눈에 보입니다:

- **일반 SPA (`createApp`)**:

  ```typescript
  createApp(App).mount("#app");
  ```

  브라우저의 `<div id="app">` 내부가 완전히 비어 있다고 가정하고, 자바스크립트로 처음부터 모든 HTML 태그를 백지 상태에서 새로 만듭니다.

- **SSG 방식 (`createSSRApp`)**:

  ```typescript
  // ViteSSG가 브라우저에서 자동으로 실행해 주는 내부 코드
  import { createSSRApp } from "vue";

  const app = createSSRApp(App);
  app.mount("#app"); // 👈 기존 HTML을 버리지 않고 'Hydration' 모드로 마운트!
  ```

  ViteSSG는 브라우저에서 일반 `createApp` 대신 Vue의 내장 함수인 **`createSSRApp`**을 호출합니다. 화면에 이미 렌더링된 HTML을 절대 지우지 않고 그대로 둔 채, `@click`이나 `v-model` 같은 **이벤트 전선만 쏙쏙 연결**합니다.

#### `samarkand-local-mate` 실제 코드로 보는 Hydration 전과 후

`src/components/HeroSection.vue`에 있는 **"Find My Guide"** 버튼을 예로 들어봅시다:

```html
<!-- src/components/HeroSection.vue -->
<button
  @click="emit('open-modal')"
  class="bg-[#C8953C] text-white px-8 py-4 rounded-DEFAULT ..."
>
  Find My Guide
</button>
```

1. **Hydration 전 (0ms ~ 300ms: HTML만 도착한 상태)**
   - 사용자의 화면에는 이미 "사마르칸트에서 나만의 로컬 가이드를 만나보세요"라는 멋진 타이틀과 노란색 **"Find My Guide"** 버튼이 완벽하게 보입니다.
   - 하지만 이 0.3초 동안 버튼을 광클해도 **모달창이 열리지 않습니다.** 아직 JavaScript(`main.js`)가 다운로드·실행되지 않아 HTML만 화면에 박제된 상태이기 때문입니다.

2. **Hydration 순간 (300ms ~ 800ms: `main.js` 실행)**
   - 브라우저가 `main.js` 실행을 완료하면서 `createSSRApp(App).mount('#app')`이 동작합니다.
   - Vue가 화면에 이미 있는 `<button>` 요소를 찾아서, 우리가 작성해 둔 `@click="emit('open-modal')"` 이벤트 리스너를 **버튼에 부착(전선 연결)**합니다.
   - 동시에 `App.vue`의 반응형 상태 변수(`isModalOpen`, `isLoading` 등)를 화면과 연결합니다.

3. **Hydration 완료 후 (800ms 이후)**
   - 이제 "Find My Guide" 버튼을 클릭하면 즉시 `openModal()` 함수가 실행되며 여행자 취향 입력 모달이 스르륵 열립니다!

| 구분                            | 누가 / 어디서 수행하나?                        | 하는 일                                                                                            |
| :------------------------------ | :--------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| **빌드 시점** (`npm run build`) | ViteSSG (Node.js 빌드 환경)                    | Vue 컴포넌트들을 미리 실행해 글자와 버튼이 꽉 찬 `.html` 파일을 구워냄                             |
| **첫 화면 로딩** (0~300ms)      | 브라우저 렌더 엔진                             | 완성된 `.html`을 받아 즉시 화면에 그림 (초고속 첫 화면 👀)                                         |
| **Hydration** (300~800ms)       | **Vue의 `createSSRApp` (ViteSSG가 자동 실행)** | 화면의 정적 HTML 태그들에 `@click`, `v-model` 등의 **이벤트 리스너를 부착**하여 인터랙션 활성화 🖱️ |

## 5. 귀납적 이해: 라우터 설정 살펴보기

AI가 SSG용으로 수정한 라우터 파일을 열어봅니다.

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from "vue-router";
import type { RouteRecordRaw } from "vue-router";

// routes를 named export로 내보내야 vite-ssg가 사용 가능
export const routes: RouteRecordRaw[] = [
  {
    path: "/",
    component: () => import("@/App.vue"),
  },
  {
    path: "/guides",
    component: () => import("@/components/ResultSection.vue"),
  },
  {
    path: "/guides/:id",
    // ↑ :id는 동적 파라미터 (alisher, dilshod, nigora, jamshid)
    component: () => import("@/pages/GuideDetailPage.vue"),
  },
];

export default createRouter({
  history: createWebHistory(),
  routes,
});
```

**주목할 점 — `export const routes`:**

기존에는 `routes` 배열을 따로 내보내지 않고 `createRouter` 안에서만 사용했을 것입니다. SSG 방식에서는 `routes`를 `export const`로 _이름을 붙여 내보내는_ 이유가 있습니다: `main.ts`에서 `ViteSSG(App, { routes })`에 직접 전달해야 하기 때문입니다.

Vite-SSG가 이 배열을 보고 "어떤 페이지들이 있는지" 파악한 뒤 하나씩 방문해서 HTML을 생성합니다.

**`() => import(...)` 문법 이해하기:**

```typescript
component: () => import("@/pages/GuideDetailPage.vue");
```

이것을 **동적 임포트(Lazy Loading)** 라고 합니다. "이 컴포넌트는 실제로 필요할 때 불러오세요"라는 의미입니다. 덕분에 처음 접속할 때 모든 페이지 코드를 한꺼번에 내려받지 않고, 필요한 페이지 코드만 그때그때 불러옵니다. 앱 초기 로딩 속도를 더 빠르게 만드는 기법입니다.

## 6. 동적 라우트와 SSG — 가이드 상세 페이지

`/guides/:id`처럼 동적 라우트(파라미터가 있는 URL)는 SSG에서 약간 특별하게 다룹니다.

**문제 상황을 이해해봅시다:**

SSG는 빌드 시점에 "어떤 URL들이 있는지" 미리 알아야 합니다. `/`나 `/guides`는 고정된 URL이라 괜찮습니다. 하지만 `/guides/:id`는 `:id` 자리에 무엇이 올지 모릅니다. `alisher`? `dilshod`? `nigora`? 새로 추가한 가이드는요?

> 💬 "가이드 상세 페이지(`/guides/:id`)가 SSG로 미리 생성되려면 어떤 id들이 있는지 알아야 할 텐데, 이걸 어떻게 설정해?"

AI가 안내하는 방법 — `vite-ssg`의 `includedRoutes` 옵션:

```typescript
// main.ts
export const createApp = ViteSSG(
  App,
  { routes },
  ({ router, app, isClient }) => {
    // 클라이언트 사이드 초기화 로직 (플러그인 추가 등)
  },
  {
    // 빌드 시 생성할 동적 라우트 목록을 samarkand-local-mate 실제 가이드 ID로 지정
    includedRoutes(paths) {
      return paths.flatMap((path) => {
        if (path === "/guides/:id") {
          // samarkand-local-mate의 실제 가이드 ID 목록으로 URL 생성
          return [
            "/guides/alisher",
            "/guides/dilshod",
            "/guides/nigora",
            "/guides/jamshid",
          ];
        }
        return [path]; // 다른 라우트('/', '/guides')는 그대로 유지
      });
    },
  },
);
```

**`flatMap`이 하는 일:**

```
입력: ['/guides/:id', '/guides', '/']
처리: '/guides/:id' → ['/guides/alisher', '/guides/dilshod', '/guides/nigora', '/guides/jamshid']
      '/guides'     → ['/guides']
      '/'           → ['/']
출력: ['/guides/alisher', '/guides/dilshod', '/guides/nigora', '/guides/jamshid', '/guides', '/']
```

`flatMap`은 배열의 각 항목을 변환하는 동시에, 변환 결과가 배열이면 펼쳐서(flat) 합쳐줍니다. 결과적으로 "생성해야 할 URL의 목록"을 만들어냅니다.

실제 프로덕션에서는 하드코딩 대신 API를 호출해서 모든 가이드 ID를 가져온 뒤 이 목록을 동적으로 생성합니다:

```typescript
includedRoutes(paths) {
  return paths.flatMap(async (path) => {
    if (path === "/guides/:id") {
      // samarkand-local-mate의 실제 Cloudflare Pages API (/api/guides) 호출
      const res = await fetch("https://samarkand-local-mate.pages.dev/api/guides");
      const guides = await res.json();
      return guides.map((guide: any) => `/guides/${guide.id}`);
    }
    return [path];
  });
}
```

새 가이드를 추가하면 다시 `npm run build`를 실행하면 됩니다. 새 가이드의 HTML도 자동으로 생성됩니다.

## 7. Cloudflare Pages에 배포하기

1권에서 이미 Cloudflare Pages 배포를 배웠습니다. SSG 결과물도 똑같이 정적 파일이기 때문에 설정이 거의 동일합니다.

```
Cloudflare Pages 빌드 설정:
- 프레임워크 프리셋: None (직접 설정)
- 빌드 명령어: npm run build
- 빌드 출력 디렉터리: dist
```

**배포 흐름:**

```
개발자가 코드 푸시 → GitHub 저장소 업데이트
    ↓
Cloudflare Pages가 자동 감지
    ↓
npm run build 자동 실행 (Cloudflare 서버에서)
    ↓
dist/ 폴더의 HTML 파일들이 전 세계 CDN에 배포
    ↓
사용자가 어디서 접속하든 가장 가까운 서버에서 HTML 즉시 전달
```

### 동적 라우트를 위한 \_redirects 파일

한 가지 추가 설정이 필요합니다 — 동적 라우트를 위한 리다이렉트 파일:

```
# public/_redirects
/* /index.html 200
```

이 파일이 없으면 `/guides/alisher`를 직접 주소창에 입력했을 때 404 오류가 납니다. 왜 그럴까요?

Cloudflare Pages는 실제 파일만 서빙합니다. SSG가 미처 생성하지 못한 URL(예: 오타가 난 URL)을 요청하면 파일이 없으므로 404가 납니다. `_redirects` 파일은 "어떤 URL이든 일단 `/index.html`로 보내줘(200)"라는 폴백(fallback) 규칙을 설정합니다.

`public/` 폴더에 이 파일을 놓으면 빌드 시 자동으로 `dist/`로 복사됩니다:

```bash
# public/_redirects 파일 생성
echo "/* /index.html 200" > public/_redirects
```

> 💬 "SSG 앱을 Cloudflare Pages에 배포할 때 필요한 설정 파일이나 주의사항을 알려줘."

## 8. `npm run build` 전후 비교

SSG 적용 전후로 실제 차이를 확인하는 방법:

### Lighthouse 점수 확인 (Chrome 개발자 도구)

Chrome 브라우저에서 F12 → Lighthouse 탭 → "Analyze page load" 버튼을 클릭하면 아래와 같은 점수를 확인할 수 있습니다.

```
적용 전 (SPA):
  Performance:   65    ← "보통" 수준
  SEO:           72    ← 구글 노출 불리
  First Paint:   3.2초 ← 흰 화면 3초 이상

적용 후 (SSG):
  Performance:   94    ← "좋음" 수준
  SEO:           98    ← 구글 최적화
  First Paint:   0.8초 ← 거의 즉시 표시
```

**Performance 점수가 65 → 94로 오른 이유:**

- First Contentful Paint(FCP): 첫 콘텐츠가 화면에 나타나는 시간이 대폭 단축됨
- Time to Interactive(TTI): 사용자가 인터랙션할 수 있게 되는 시간 단축
- Cumulative Layout Shift(CLS): SSG HTML이 미리 레이아웃을 잡아줘서 화면 흔들림 최소화

**SEO 점수가 72 → 98로 오른 이유:**

- 구글 봇이 실제 콘텐츠를 HTML에서 바로 읽을 수 있음
- `<title>`, `<meta description>` 태그도 각 페이지별로 올바르게 설정 가능
- 페이지 로딩 속도 자체가 SEO 점수에 영향을 줌

> 💬 "현재 앱을 Lighthouse로 측정했더니 Performance가 65점이야. SSG 적용 후 개선할 수 있는 추가 최적화 방법도 알려줘."

### 실제로 차이를 느껴보는 법

빌드 후 `npm run preview`를 실행하면 로컬에서 빌드 결과물을 미리볼 수 있습니다. Chrome 개발자 도구(F12)의 Network 탭에서 "Slow 3G"로 네트워크 속도를 낮춰보세요. SPA와 SSG의 첫 로딩 차이가 극명하게 느껴집니다.

---

## 9. SEO 최적화 — 구글 검색에 잘 걸리는 앱 만들기

> 🎯 **이 섹션의 목표**: 아무리 잘 만든 앱이라도 구글 검색 결과 3페이지에 묻혀 있으면 존재하지 않는 것과 같습니다. "사마르칸트 여행 가이드"로 검색했을 때 우리 앱이 1페이지에 나와야 비용 없이 사용자를 유입시킬 수 있습니다.

4장에서 도입한 Vite-SSG가 SEO와 강력하게 연결됩니다. SPA는 초기 HTML이 비어 있어 구글 봇이 내용을 읽지 못하지만, **SSG는 미리 만들어진 HTML에 내용이 가득 차 있어 구글 봇이 바로 인덱싱**할 수 있습니다. 이제 `samarkand-local-mate`에 실제 구현된 SEO 아키텍처를 확인해봅니다.

### 9-1. 패키지 설치 및 main.ts 등록: `@unhead/vue`

Vue 3 생태계에서 메타 태그와 `<head>` 관리를 담당하는 공식 표준 라이브러리는 **`@unhead/vue`**입니다(기존 `@vueuse/head`의 공식 후속작).

```bash
npm install @unhead/vue
```

`main.ts`에서 Unhead 인스턴스를 생성해 앱에 플러그인으로 등록합니다:

```typescript
// src/main.ts
import { createApp } from 'vue';
import { createUnhead } from '@unhead/vue';
import App from './App.vue';
import './assets/main.css';

const app = createApp(App);
const head = createUnhead();
app.use(head); // 👈 Unhead 플러그인 등록

app.mount('#app');
```

> 💡 **Vite-SSG와 함께 쓸 때**: `ViteSSG` 세 번째 인자인 초기화 콜백에서 `({ app }) => { app.use(createUnhead()) }`로 등록하면 빌드 타임의 HTML 스냅샷과 브라우저 클라이언트 양쪽 모두에서 `<head>`가 완벽하게 동기화됩니다.

---

### 9-2. 프롬프팅: 메타 태그와 구조화 데이터를 한 번에

> 💬 "samarkand-local-mate에 구글 검색 최적화(SEO)를 적용해줘. `@unhead/vue`를 활용해서 Open Graph, Twitter 카드, Canonical URL, 그리고 schema.org의 `TouristGuide` JSON-LD 구조화 데이터를 동적으로 제어할 수 있는 `useSeo.ts` 컴포저블과 `SeoMeta.vue` 컴포넌트를 만들어줘."

AI가 프로젝트에 다음 두 핵심 파일을 생성합니다:

1. `src/composables/useSeo.ts` — 메타 태그와 JSON-LD 스크립트 생성을 전담하는 재사용 컴포저블
2. `src/components/SeoMeta.vue` — 템플릿 어디서나 선언적으로 태그를 주입할 수 있는 Renderless 컴포넌트

{% hint style="info" %}
**💡 개념 짚고 가기: '컴포저블(Composable)'이란 무엇인가요?**

2장에서 우리는 Vue 3의 **Composition API**(`<script setup>`, `ref`, `computed` 등)를 배웠습니다. 이 기능들을 활용해 **"상태(state)와 로직(logic)을 깔끔하게 묶어 어디서나 재사용할 수 있게 만든 순수 함수"**를 바로 **컴포저블(Composable)**이라고 부릅니다. (React를 경험해 본 독자라면 '커스텀 훅(Custom Hook)'과 완전히 같은 개념이라고 생각하면 쉽습니다.)

**컴포넌트 vs 컴포저블, 무엇이 다른가요?**

| 구분 | 컴포넌트 (`.vue`) | 컴포저블 (`.ts`) |
| :--- | :--- | :--- |
| **주요 역할** | **화면(UI)** + 동작 결합 | **화면 없는 순수한 로직과 상태 관리** |
| **구성 요소** | `<template>` + `<script>` + `<style>` | 순수 TypeScript 함수 (`ref`, `computed` 포함 가능) |
| **이름 규칙** | `HeroSection.vue`, `SeoMeta.vue` (명사/대문자) | `useSeo.ts`, `useHead.ts` (**`use...` 접두사 소문자**) |
| **비유** | **모니터 달린 커피 머신 본체** | **어디든 장착 가능한 고성능 에스프레소 추출 엔진** |

**왜 컴포넌트와 분리해서 컴포저블을 만들까요?**
메타 태그를 조합하고, JSON-LD 규격에 맞춰 사마르칸트 좌표와 평점을 조립하는 일은 **"화면을 그리는 일(HTML)"**이 아니라 **"순수한 데이터 가공 로직(TypeScript)"**입니다.

이 로직을 `src/composables/useSeo.ts`라는 독립된 부품으로 만들어 두면, 나중에 가이드 상세 페이지든, 투어 예약 완료 페이지든 화면 종류에 상관없이 `useSeo(...)` 한 줄만 불러서 언제 어디서나 SEO 기능을 장착할 수 있습니다.
{% endhint %}

---

### 9-3. 귀납적 코드 이해: `src/composables/useSeo.ts` — 검색엔진의 언어로 변환기

실제 `samarkand-local-mate`의 `src/composables/useSeo.ts`를 열어봅니다:

```typescript
// src/composables/useSeo.ts
import { useHead } from '@unhead/vue'

export interface SeoOptions {
  title: string
  description: string
  imageUrl?: string
  path?: string
  jsonLd?: Record<string, any>
}

export interface TouristGuideSchemaOptions {
  name: string
  description: string
  languages?: string[] | string
  ratingValue?: string | number
  reviewCount?: string | number
  imageUrl?: string
  addressLocality?: string
}

export function useSeo(options: SeoOptions) {
  const siteUrl = 'https://samarkand-local-mate.pages.dev'
  const fullUrl = options.path ? `${siteUrl}${options.path}` : siteUrl
  const image = options.imageUrl || `${siteUrl}/og-image.jpg`

  const headObject: any = {
    title: `${options.title} | Samarkand Local Mate`,
    meta: [
      { name: 'description', content: options.description },
      // Open Graph (SNS 카카오톡/페이스북 공유 카드)
      { property: 'og:title', content: options.title },
      { property: 'og:description', content: options.description },
      { property: 'og:image', content: image },
      { property: 'og:url', content: fullUrl },
      { property: 'og:type', content: 'website' },
      { property: 'og:site_name', content: 'Samarkand Local Mate' },
      // Twitter Card
      { name: 'twitter:card', content: 'summary_large_image' },
      { name: 'twitter:title', content: options.title },
      { name: 'twitter:description', content: options.description },
      { name: 'twitter:image', content: image },
    ],
    link: [
      { rel: 'canonical', href: fullUrl } // 👈 구글이 중복 URL을 방지하는 표준 링크
    ]
  }

  // JSON-LD 구조화 데이터가 있으면 <script type="application/ld+json"> 태그로 주입
  if (options.jsonLd) {
    headObject.script = [
      {
        type: 'application/ld+json',
        children: JSON.stringify(options.jsonLd)
      }
    ]
  }

  return useHead(headObject)
}

// schema.org의 TouristGuide 스키마를 생성하는 헬퍼 함수
export function createTouristGuideSchema(guide: TouristGuideSchemaOptions) {
  const languages = Array.isArray(guide.languages)
    ? guide.languages
    : (guide.languages ? guide.languages.split(',').map(s => s.trim()) : ['Korean', 'Uzbek', 'Russian'])

  return {
    '@context': 'https://schema.org',
    '@type': 'TouristGuide',
    'name': guide.name,
    'description': guide.description,
    'knowsLanguage': languages,
    'image': guide.imageUrl,
    'address': {
      '@type': 'PostalAddress',
      'addressLocality': guide.addressLocality || 'Samarkand',
      'addressCountry': 'UZ'
    },
    'geo': {
      '@type': 'GeoCoordinates',
      'latitude': 39.6547,
      'longitude': 66.9597
    },
    ...(guide.ratingValue ? {
      'aggregateRating': {
        '@type': 'AggregateRating',
        'ratingValue': String(guide.ratingValue),
        'reviewCount': String(guide.reviewCount || '10')
      }
    } : {})
  }
}
```

**코드에서 눈여겨볼 핵심 포인트:**

1. **`<link rel="canonical">`**: 같은 사이트가 도메인 주소나 쿼리 파라미터로 여러 개 잡혀 구글 검색 페널티를 받지 않도록 대표 공식 URL을 명시합니다.
2. **`headObject.script`와 JSON-LD**: 구글 봇이 가장 좋아하는 `schema.org` 표준 객체를 `application/ld+json` 스크립트로 `<head>`에 주입합니다.
3. **`TouristGuide` 스키마**: 사마르칸트의 실제 위도/경도(`39.6547, 66.9597`), 구사 언어, 평점(`aggregateRating`)을 담아 구글 검색 결과에 별점(⭐⭐⭐⭐⭐)이 표시되는 **리치 스니펫(Rich Snippet)**을 만들어냅니다.

---

### 9-4. 귀납적 코드 이해: `src/components/SeoMeta.vue` — 무렌더링(Renderless) 컴포넌트

위 컴포저블을 감싸서 템플릿에서 편리하게 태그 형태로 쓸 수 있도록 만든 실제 `SeoMeta.vue`입니다:

```vue
<!-- src/components/SeoMeta.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useSeo, createTouristGuideSchema, type TouristGuideSchemaOptions } from '@/composables/useSeo'

const props = withDefaults(
  defineProps<{
    title: string
    description: string
    imageUrl?: string
    path?: string
    isGuideDetail?: boolean
    guideInfo?: TouristGuideSchemaOptions
  }>(),
  {
    path: '/',
    isGuideDetail: false
  }
)

const jsonLdData = computed(() => {
  // 가이드 상세 정보가 들어온 경우: TouristGuide 스키마 생성
  if (props.isGuideDetail && props.guideInfo) {
    return createTouristGuideSchema(props.guideInfo)
  }
  // 일반 페이지인 경우: 서비스 대표 TravelAgency 스키마 생성
  return {
    '@context': 'https://schema.org',
    '@type': 'TravelAgency',
    'name': 'Samarkand Local Mate',
    'description': props.description,
    'url': 'https://samarkand-local-mate.pages.dev'
  }
})

useSeo({
  title: props.title,
  description: props.description,
  imageUrl: props.imageUrl,
  path: props.path,
  jsonLd: jsonLdData.value
})
</script>

<template>
  <!-- Renderless SEO component: 화면에는 아무 DOM 요소도 그리지 않고 오직 <head>만 제어 -->
</template>
```

---

### 9-5. 실제 앱(`App.vue`)에서의 실전 활용

실제 `samarkand-local-mate/src/App.vue`에서는 이 `<SeoMeta>`를 어떻게 사용할까요?

사용자가 처음 방문했을 때와, 가이드 매칭 결과가 나왔을 때 **페이지의 메타 정보가 상황에 맞게 동적으로 전환**되도록 구현되어 있습니다:

```vue
<!-- src/App.vue (실제 프로젝트 코드 발췌) -->
<template>
  <div class="bg-background text-on-background min-h-screen">
    <!-- 1. 기본 첫 화면: 서비스 대표 SEO 메타데이터 -->
    <SeoMeta
      v-if="!showResult || guides.length === 0"
      title="사마르칸트 로컬 가이드 매칭 서비스"
      description="역사 · 미식 · 사진 — 당신의 취향에 딱 맞는 사마르칸트 현지 로컬 가이드를 AI가 30초 만에 추천합니다."
      path="/"
    />

    <!-- 2. 가이드 매칭 결과 화면: 맞춤 가이드 정보로 TouristGuide 스키마 즉시 반영! -->
    <SeoMeta
      v-else
      :title="`${resUserName}님의 맞춤 사마르칸트 가이드 추천`"
      :description="aiRecommendation || `${guides[0]?.name || '알리셰르'} 가이드의 사마르칸트 맞춤 투어 안내`"
      :path="`/guides/${guides[0]?.id || 1}`"
      :is-guide-detail="true"
      :guide-info="{
        name: guides[0]?.name || '알리셰르 (Alisher)',
        description: guides[0]?.description || '사마르칸트 국립대 역사학과 출신 역사 및 고건축 전문 가이드',
        languages: guides[0]?.languages || ['한국어', '우즈베크어', '러시아어'],
        ratingValue: guides[0]?.rating || '4.9',
        imageUrl: 'https://samarkand-local-mate.pages.dev/images/alisher.jpg'
      }"
    />

    <TopNavbar @open-modal="openModal" />
    <HeroSection @open-modal="openModal" />
    <!-- ... 나머지 UI 섹션들 -->
  </div>
</template>
```

향후 라우터 기반의 가이드 개별 상세 페이지(`/guides/:id`)를 분리하더라도, 이 `<SeoMeta>` 컴포넌트에 해당 가이드 데이터만 넘겨주면 동일하게 동작합니다.

---

### 9-6. 프롬프팅: 사이트맵(sitemap.xml)과 robots.txt

> 💬 "Cloudflare Pages에 배포할 때 구글 검색 봇이 사이트 구조를 쉽게 긁어갈 수 있도록 sitemap.xml과 robots.txt 생성 규칙을 만들어줘."

```xml
<!-- public/sitemap.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://samarkand-local-mate.pages.dev/</loc>
    <priority>1.0</priority>
    <changefreq>weekly</changefreq>
  </url>
  <url>
    <loc>https://samarkand-local-mate.pages.dev/guides/alisher</loc>
    <priority>0.8</priority>
    <changefreq>monthly</changefreq>
  </url>
  <url>
    <loc>https://samarkand-local-mate.pages.dev/guides/dilshod</loc>
    <priority>0.8</priority>
    <changefreq>monthly</changefreq>
  </url>
  <url>
    <loc>https://samarkand-local-mate.pages.dev/guides/nigora</loc>
    <priority>0.8</priority>
    <changefreq>monthly</changefreq>
  </url>
  <url>
    <loc>https://samarkand-local-mate.pages.dev/guides/jamshid</loc>
    <priority>0.8</priority>
    <changefreq>monthly</changefreq>
  </url>
</urlset>
```

```text
# public/robots.txt
User-agent: *
Allow: /

Sitemap: https://samarkand-local-mate.pages.dev/sitemap.xml
```

사이트맵은 구글 봇에게 "우리 사이트의 지도"를 제공합니다. 이 파일을 **Google Search Console**에 등록하면, 구글이 우리 가이드 페이지들을 누락 없이 빠르게 인덱싱합니다.

{% hint style="warning" %}
**⚠️ 실무 팁: 연습용/학습용 사이트의 구글 검색 노출 방지하기**

- **도메인 변경 필수**: `useSeo.ts`, `sitemap.xml`, `robots.txt`에 적힌 `samarkand-local-mate.pages.dev`는 책의 예시 도메인입니다. 실습할 때는 반드시 **독자 본인의 Cloudflare Pages 주소**(예: `https://my-tour-project.pages.dev`)로 변경해야 합니다.
- **연습 단계에서 구글 노출을 막고 싶다면?**: 아직 테스트용 데이터(가짜 가이드 정보)만 들어있고 실제 여행자를 받을 준비가 되지 않았다면, 구글 봇이 사이트를 긁어가지 못하게 `robots.txt`를 차단 모드로 설정해두는 것이 실무 표준입니다:
  ```text
  # public/robots.txt (개발 및 연습 단계 — 검색 노출 차단)
  User-agent: *
  Disallow: /
  ```
  *(또는 HTML `<head>`에 `<meta name="robots" content="noindex, nofollow" />` 메타 태그를 추가해도 됩니다.)*
- **정식 오픈 시점 체크리스트**: 나만의 진짜 도메인을 연결하고 실제 고객을 맞이할 준비가 끝났을 때, 비로소 `Allow: /`로 변경하여 배포하고 Google Search Console에 사이트맵을 제출하세요!
{% endhint %}

---

### 9-7. 크롤러 검증과 핵심 웹 지표(Core Web Vitals)

Chrome 개발자 도구(F12)의 Lighthouse를 실행해 봅니다:

```
개발자 도구 → Lighthouse → [SEO] & [Performance] 탭 체크 → "Analyze page load" 실행
```

| 핵심 지표 | 의미 | 목표 | Vite-SSG + Cloudflare 효과 |
| :--- | :--- | :--- | :--- |
| **LCP** (Largest Contentful Paint) | 대표 이미지나 메인 텍스트가 뜨는 시간 | 2.5초 이내 | **0.8초 달성** (미리 구워둔 정적 HTML을 글로벌 CDN에서 즉시 반환) |
| **FID / INP** (First Input Delay) | 사용자가 클릭했을 때 반응 속도 | 100ms 이내 | **초고속 반응** (가벼운 번들과 빠른 Hydration 완료) |
| **CLS** (Cumulative Layout Shift) | 로딩 중 화면이 덜컹거리며 밀리는 현상 | 0.1 이내 | **0에 수렴** (정적 뼈대와 고정 이미지 영역 유지) |

Vite-SSG와 `@unhead/vue`의 조합은 단순한 "기술적 만족"이 아니라, 구글 검색 결과 1페이지 노출과 사용자 이탈률 방지라는 **실제 비즈니스 성공 지표**로 이어집니다.

{% hint style="info" %}
**💡 SEO 점수 변화 전/후 정리**

8절에서 확인한 Lighthouse 점수를 다시 보면:

- **SEO 72 → 98**: SSG로 구글 봇이 실제 콘텐츠를 읽을 수 있게 됨
- **+ 메타 태그 추가**: 각 가이드 페이지별 `<title>`, `description`, JSON-LD 삽입으로 리치 스니펫 활성화
- **+ 사이트맵 제출**: Google Search Console에 sitemap.xml을 제출하면 새 가이드 페이지도 빠르게 인덱싱됨
{% endhint %}

---

## 마무리: 이 장에서 배운 것들

```
웹 렌더링의 역사:
  전통 SSR    →  CSR/SPA          →  SSG (현재 우리의 선택)
  ↑ 서버 부담     ↑ 흰 화면/SEO 취약     ↑ 둘의 장점 결합
```

- **전통 SSR**: PHP 시대처럼 서버가 매번 HTML을 만드는 방식 — 항상 실행 중인 서버가 필요
- **CSR/SPA**: Vue/React로 브라우저가 HTML을 만드는 방식 — 편리하지만 흰 화면과 SEO 문제
- **SSG의 원리**: 빌드 시점에 HTML을 미리 생성해두는 방식 — 두 방식의 장점만 결합
- **Vite-SSG 설정**: `main.ts` 수정, `vite-ssg build` 스크립트로 단 두 가지 변경으로 적용
- **빌드 결과물**: `dist/` 폴더에 각 라우트별 완성된 HTML 파일이 생성됨
- **Hydration**: SSG HTML + Vue 인터랙션의 결합 — 보여주고 나서 살아있게 만들기
- **동적 라우트**: `includedRoutes`로 생성할 페이지 목록 지정
- **Cloudflare Pages 배포**: 기존 1권 배포와 동일한 흐름 + `_redirects` 파일 추가
- **SEO 최적화**: `SeoMeta.vue`로 각 가이드 페이지별 메타 태그 + JSON-LD 구조화 데이터 삽입
- **컴포저블(Composable)**: 화면(UI) 없는 순수 로직과 상태를 `use...` 함수로 캡슐화하여 재사용하는 Vue 3 기법
- **핵심 웹 지표**: LCP · FID · CLS — Vite-SSG + Cloudflare CDN이 자동으로 개선

다음 장에서는 완성된 앱에 회원 가입, 로그인, 로그아웃을 구현하는 **인증 시스템**을 추가합니다.
