---
hidden: true
---

# 제4장: Vite-SSG — 빠르고 검색에 잘 걸리는 앱 만들기

> 🎉 이 장의 작은 성공: `npm run build`를 실행하면 `dist/` 폴더에 여러 개의 완성된 HTML 파일이 뚝딱 만들어진다!

3장에서 TypeScript로 코드에 안전망을 씌웠습니다. 이번 장에서는 앱의 *성능*과 *검색 노출*이라는 두 가지 현실적인 문제를 해결합니다.

## 1. 지금 앱의 두 가지 문제

2장에서 만든 Vue 앱을 배포하면 어떤 일이 일어날까요?

### 문제 1: 첫 화면이 느리다

브라우저가 서버에서 받은 HTML 파일을 열어보면 이렇습니다:

```html
<!-- 서버에서 받은 index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>사마르칸트 로컬 메이트</title>
  </head>
  <body>
    <div id="app"></div>  <!-- ← 텅 비어 있음! -->
    <script src="/assets/main.js"></script>
  </body>
</html>
```

`<div id="app">` 이 텅 비어 있습니다. 브라우저가 `main.js`를 내려받고, 해석하고, 실행해야 비로소 화면이 그려집니다. 이 시간 동안 사용자는 하얀 빈 화면을 봅니다.

### 문제 2: 구글이 빈 화면만 본다

구글 검색 봇이 이 페이지를 방문합니다. 봇은 HTML을 읽어서 페이지 내용을 파악하는데, `<div id="app"></div>` — 아무것도 없습니다. JavaScript를 기다려주지 않으니까요.

결과적으로 구글은 이 페이지가 비어 있다고 판단하고 검색 결과에서 낮은 순위를 부여하거나 아예 색인하지 않습니다. 아무리 좋은 투어 정보를 담아도 구글 검색에 걸리지 않는다면 아무도 찾아오지 않습니다.

{% hint style="info" %}
**💡 SPA vs SSG — 택배로 비유하기**

**SPA(Single Page Application) 방식**: 빈 택배 상자를 먼저 보내고, 상자 안에 '조립 설명서(JavaScript)'를 넣어둡니다. 받는 사람(브라우저)이 설명서를 읽고 직접 내용물을 조립해야 합니다. → 조립 시간만큼 기다려야 함

**SSG(Static Site Generation) 방식**: 공장(빌드 서버)에서 내용물을 미리 다 조립한 완성품을 보냅니다. 받는 사람은 상자를 열자마자 바로 사용할 수 있습니다. → 기다릴 필요 없음
{% endhint %}

## 2. 해결책: Vite-SSG

**Vite-SSG**는 두 세계의 장점을 합칩니다.

- **개발 중**: 빠르고 유연한 Vue SPA처럼 작업
- **배포 시**: 각 페이지를 완성된 HTML 파일로 미리 구워서 배포

"SSG(Static Site Generation)"는 "정적 사이트 생성"으로, 빌드 시점에 JavaScript를 미리 실행해서 완성된 HTML을 파일로 저장해두는 기술입니다.

| 방식 | 동작 원리 | 첫 로딩 속도 | SEO |
|------|----------|------------|-----|
| SPA | 빈 HTML + JS 실행 후 렌더링 | 느림 | 취약 |
| SSG | 미리 만들어진 HTML 바로 전달 | 빠름 | 강함 |
| SSR | 요청마다 서버에서 렌더링 | 빠름 | 강함 (서버 필요) |

SSG는 SSR처럼 빠른 첫 로딩과 SEO를 제공하면서도, 서버 없이 정적 파일만으로 배포할 수 있어서 1권에서 배운 **Cloudflare Pages** 같은 서비스에 그대로 올릴 수 있습니다.

## 3. 프롬프팅: Vite-SSG 적용하기

> 💬 "이 웹앱의 초기 로딩이 느린데, `vite-ssg`를 도입해서 빌드할 때 정적 HTML을 미리 뽑아내게 설정해줘. 라우터 설정도 같이 수정해서 각 투어 상세 페이지가 별도의 HTML로 생성되게 해."

AI가 수행하는 작업 순서:

### 1단계: `vite-ssg` 패키지 설치

```bash
npm install vite-ssg
```

### 2단계: `main.ts` 수정 — SSG용 export 추가

기존 SPA 방식의 `main.ts`:

```typescript
// 기존 main.ts — SPA 방식
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```

SSG 방식으로 수정:

```typescript
// 수정된 main.ts — SSG 방식
import { ViteSSG } from 'vite-ssg'
import App from './App.vue'
import { routes } from './router'

// createApp 대신 ViteSSG를 export default로 내보냄
export const createApp = ViteSSG(
  App,
  { routes }
)
```

변경점: `createApp(...).mount()` 대신 `ViteSSG(...)`를 `export`합니다. 빌드 시 Vite-SSG가 이 함수를 가져가서 각 라우트를 미리 방문하고 HTML을 생성합니다.

### 3단계: `vite.config.ts` 확인

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  // vite-ssg는 별도 플러그인 추가 없이 main.ts export만으로 작동
})
```

### 4단계: `package.json` — 빌드 스크립트 확인

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite-ssg build",
    "preview": "vite preview"
  }
}
```

`"build"` 스크립트가 `vite build`에서 `vite-ssg build`로 변경됩니다.

## 4. 귀납적 이해: 빌드 결과물로 SSG 원리 발견하기

`npm run build`를 실행한 후 생성된 `dist/` 폴더를 IDE에서 열어봅니다.

```
dist/
├── index.html              ← 메인(홈) 페이지 — 완성된 HTML
├── guides/
│   ├── index.html          ← 가이드 목록 페이지
│   ├── ali/
│   │   └── index.html      ← 알리 가이드 상세 페이지
│   └── bobo/
│       └── index.html      ← 보보 가이드 상세 페이지
├── tours/
│   └── index.html          ← 투어 목록 페이지
└── assets/
    ├── main.js             ← 번들된 JavaScript
    └── main.css            ← 번들된 CSS
```

**"Vue SPA인데 왜 여러 개의 `.html` 파일이 나왔지?"**

SSG가 빌드 시점에 각 라우트를 *미리 방문*해서 HTML 스냅샷을 찍어두는 방식이기 때문입니다. 브라우저가 `/guides/ali`에 접속하면, JavaScript가 실행되기도 전에 이미 완성된 HTML이 전달됩니다.

각 HTML 파일을 열어보면 텅 빈 `<div id="app">` 대신 실제 콘텐츠가 들어있는 것을 확인할 수 있습니다:

```html
<!-- dist/guides/ali/index.html — SSG가 미리 렌더링한 결과 -->
<div id="app">
  <header><!-- 네비게이션 --></header>
  <main>
    <img src="/images/ali.jpg" alt="알리 가이드">
    <h1>알리</h1>
    <p>평점: ⭐ 4.8</p>
    <p>언어: 한국어, 영어</p>
    <!-- ... 실제 콘텐츠가 가득! -->
  </main>
</div>
```

구글 봇이 이 파일을 읽으면, 내용이 가득한 HTML을 완전히 파악할 수 있습니다.

{% hint style="info" %}
**💡 Hydration — SSG와 SPA의 결합**

SSG가 만든 HTML 파일을 브라우저가 받으면, 사용자는 즉시 내용을 볼 수 있습니다. 그 후 JavaScript가 로드되면 Vue가 이미 그려진 HTML에 *다시 연결*(hydration)되어 버튼 클릭 등의 인터랙션이 활성화됩니다.

```
1. 브라우저가 HTML 받음      → 즉시 화면에 콘텐츠 표시 (빠름!)
2. JS 파일 로드·실행         → Vue가 HTML에 연결(hydration)
3. Hydration 완료            → 버튼, 애니메이션 등 인터랙션 활성화
```

결과적으로 사용자는 로딩 화면 없이 바로 내용을 보고, 잠깐 뒤에 인터랙션이 가능해집니다.
{% endhint %}

## 5. 귀납적 이해: 라우터 설정 살펴보기

AI가 SSG용으로 수정한 라우터 파일을 열어봅니다.

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import type { RouteRecordRaw } from 'vue-router'

// routes를 named export로 내보내야 vite-ssg가 사용 가능
export const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('@/pages/HomePage.vue')
  },
  {
    path: '/guides',
    component: () => import('@/pages/GuidesPage.vue')
  },
  {
    path: '/guides/:id',
    component: () => import('@/pages/GuideDetailPage.vue')
  },
  {
    path: '/tours',
    component: () => import('@/pages/ToursPage.vue')
  }
]

export default createRouter({
  history: createWebHistory(),
  routes
})
```

`routes`를 `export const`로 *이름을 붙여 내보내는* 이유: `main.ts`에서 `ViteSSG(App, { routes })`에 직접 전달해야 하기 때문입니다. Vite-SSG가 이 배열을 보고 "어떤 페이지들이 있는지" 파악한 뒤 하나씩 방문해서 HTML을 생성합니다.

## 6. 동적 라우트와 SSG — 가이드 상세 페이지

`/guides/:id`처럼 동적 라우트(파라미터가 있는 URL)는 SSG에서 약간 특별하게 다룹니다.

> 💬 "가이드 상세 페이지(`/guides/:id`)가 SSG로 미리 생성되려면 어떤 id들이 있는지 알아야 할 텐데, 이걸 어떻게 설정해?"

AI가 안내하는 방법 — `vite-ssg`의 `includedRoutes` 옵션:

```typescript
// main.ts
export const createApp = ViteSSG(
  App,
  { routes },
  ({ router, app, isClient }) => {
    // 클라이언트 사이드 초기화 로직
  },
  {
    // 빌드 시 생성할 동적 라우트 목록을 직접 지정
    includedRoutes(paths) {
      return paths.flatMap(path => {
        if (path === '/guides/:id') {
          // 실제 가이드 ID 목록으로 URL 생성
          return ['/guides/ali', '/guides/bobo', '/guides/jasur']
        }
        return [path]
      })
    }
  }
)
```

실제 프로덕션에서는 API를 호출해서 모든 가이드 ID를 가져온 뒤 이 목록을 동적으로 생성합니다.

## 7. Cloudflare Pages에 배포하기

1권에서 이미 Cloudflare Pages 배포를 배웠습니다. SSG 결과물도 똑같이 정적 파일이기 때문에 설정이 거의 동일합니다.

```
Cloudflare Pages 빌드 설정:
- 빌드 명령어: npm run build
- 빌드 출력 디렉터리: dist
```

한 가지 추가 설정이 필요합니다 — 동적 라우트를 위한 리다이렉트 파일:

```
# dist/_redirects (또는 public/_redirects)
/* /index.html 200
```

이 파일이 없으면 `/guides/ali`를 직접 주소창에 입력했을 때 404 오류가 납니다. SSG가 만들어둔 HTML 파일을 찾지 못하고 Cloudflare가 기본 동작으로 404를 반환하기 때문입니다.

> 💬 "SSG 앱을 Cloudflare Pages에 배포할 때 필요한 설정 파일이나 주의사항을 알려줘."

## 8. `npm run build` 전후 비교

SSG 적용 전후로 실제 차이를 확인하는 방법:

**Lighthouse 점수 확인 (Chrome 개발자 도구)**

```
적용 전 (SPA):
  Performance:   65
  SEO:           72
  First Paint:   3.2초

적용 후 (SSG):
  Performance:   94
  SEO:           98
  First Paint:   0.8초
```

> 💬 "현재 앱을 Lighthouse로 측정했더니 Performance가 65점이야. SSG 적용 후 개선할 수 있는 추가 최적화 방법도 알려줘."

---

이 장에서 배운 것들:

- **SPA의 한계**: 빈 HTML로 인한 느린 첫 로딩과 SEO 취약점
- **SSG의 원리**: 빌드 시점에 HTML을 미리 생성해두는 방식
- **Vite-SSG 설정**: `main.ts` 수정, `vite-ssg build` 스크립트
- **빌드 결과물**: `dist/` 폴더에 각 라우트별 완성된 HTML 파일
- **Hydration**: SSG HTML + Vue 인터랙션의 결합
- **동적 라우트**: `includedRoutes`로 생성할 페이지 목록 지정
- **Cloudflare Pages 배포**: 기존 1권 배포와 동일한 흐름

다음 장에서는 완성된 앱에 회원 가입, 로그인, 로그아웃을 구현하는 **인증 시스템**을 추가합니다.
