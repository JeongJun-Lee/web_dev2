---
hidden: true
---

# 제2장: Vue.js로 앱으로 부활시키기 — 바닐라 JS에서 모던 프레임워크로

> 🎉 이 장의 작은 성공: 1권에서 만든 사마르칸트 투어 앱이 Vue 컴포넌트로 다시 살아난다!

## 1. 프롬프팅: 기존 원본 코드를 직접 Vue.js로 변환하기

1장에서 **IDE에 1권 원본 폴더를 열어둔 상태**를 그대로 이어갑니다. 특별히 새 폴더를 따로 만들거나, 어딘가에서 코드를 복사해 넣을 필요가 없습니다. **지금 IDE에 열린 바로 이 원본 폴더에, AI가 Vue.js 구조를 직접 밀어 넣어 변환합니다.**

Antigravity IDE의 AI는 현재 열려 있는 폴더의 전체 구조와 파일 내용을 이미 파악하고 있습니다. 따라서 별도의 사전 준비 없이, 곧바로 마이그레이션을 지시하기만 하면 됩니다.

### 💡 마이그레이션 프롬프트 입력

AI 채팅창에 다음과 같이 프롬프트를 입력합니다.

> 💬 "현재 열려 있는 프로젝트는 1권에서 만든 순수 자바스크립트(Vanilla JS) 사마르칸트 투어 앱이야. 이것을 Vue.js와 Vite를 도입해서 변환해줘. 다만, 기존의 `functions/api` 백엔드 코드는 폴더 구조 그대로 유지하고, Tailwind CSS 스타일도 깨지지 않게 유지해줘. 변환이 끝나면 빠진 기능이 있는지 직접 확인하고 체크리스트를 보여줘."

### 📋 프롬프트 제출 후 진행 과정: Implementation Plan, `[Proceed]`, 그리고 권한 승인

프롬프트를 전송하면 AI가 코드 작성부터 화면 검증까지의 과정을 단계별로 진행합니다.

1. **Implementation Plan (구현 계획서) 생성**: AI가 기존 코드 분석에 따른, "어떤 파일들을 생성/수정할 것인지" 정리한 청사진(Implementation Plan)을 먼저 만듭니다.
2. **계획 검토 및 승인**: 이제 우리 편에서 응답할 차례입니다. AI 시대에서 우리의 역할은 실무담당인 AI 부하직원에 과업을 지시하고, AI 부하직원이 가져온 과업 이행 계획서에 대해 판단과 결정을 내려주는 기업의 부서장 또는 대표의 역할입니다. 따라서, 계획서를 읽어보고, 의도했던 바가 아니면 다시 정확한 의도와 방향을 알려주면서 계획서 수정을 요구해야 하며, 만약, 미흡한 부분이 보이면 더 보완할 것을 지시할 수 있어야 합니다.

> 현재는 이 책의 도입부이기 때문에 AI에 의해 제시된 구현 계획을 열었을 때, 생소한 용어도 보이고, 아직은 전체적으로 다 이해할 수 없는 것이 당연합니다. 그럼에도 불구하고, 늘 한번 읽어보고 작업승락을 하는 습관을 들이는 것이 팔요하기 때문에 일단 여러분 수준에서 이해할 수 있는 선까지 확인한 뒤, AI 채팅창에 있는 **`[Proceed]` (진행하기)** 버튼을 클릭합니다.

1. **실제 코드 생성 및 파일 변환**: `[Proceed]` 버튼을 누르면 비로소 AI가 파일들을 하나씩 생성하고 코드를 변환해 적용하기 시작합니다.
2. **자율 화면 점검 & 도구 실행 권한 승인 (`Approve`)**:
   - 고성능 AI 모델은 파일 수정 후 끝내는 것이 아니라, 개발 서버(`npm run dev`)를 직접 켜고 브라우저 도구를 실행해 화면이 깨지지 않는지, 버튼이나 애니메이션이 정상 동작하는지 직접 테스트합니다.
   - 언급된 테스트를 수행하기 위해 **브라우저 실행의 허용**(`Approve` / `Allow`)**을 요구하는 팝업** 뜹니다. 이때 **`[Approve]` (승인)** 버튼을 클릭해 주면 AI가 직접 브라우저를 컨트롤하며 최종 기능 검증까지 마치게 됩니다.

{% hint style="warning" %}
**AI가 직접 브라우저를 컨트롤하며 점검하는 과정을 무조건 끝까지 지켜봐야 할까요?**

이렇게 AI가 브라우저를 직접 컨트롤하며 점검까지 해주는 건 놀라운 진보임에 틀림 없지만, **그렇다고 해당 과정을 빠르고 정확하게 수행하는 것은 아닙니다.** 예를 들어 눌러야 할 버튼을 찾지 못하며 버벅이며 헤메면서 많은 시간을 보낼 때가 있습니다. 이 경우엔 구지 끝까지 기다리지 말고, 프롬프트 채팅창에 있는 지시사항 수행중지 버튼을 눌러 현재 작업을 중단시킨 후, 우리가 메뉴얼로 직접 테스트 하고 만약 문제를 발견했을 때, 발견한 해당 문제를 건별 프롬프트로 지시하며 문제를 수정하는게 더 빠른 목표달성 일 수 있습니다.
{% endhint %}

AI가 작업을 마치면 1권의 단일 HTML 파일이 다음과 같은 모던한 프로젝트 구조로 재탄생합니다:

```
samarkand-local-mate/
├── index.html                  ← Vue 앱 진입점 (<div id="app">)
├── package.json                ← Vue 3, Vite, Tailwind 설정 및 라이브러리
├── vite.config.ts              ← 개발 서버 및 로컬 Mock API 미들웨어
├── tailwind.config.js          ← 실크로드/사마르칸트 맞춤 테마 팔레트
├── tsconfig.json               ← TypeScript 컴파일 설정
├── functions/api/              ← Cloudflare Pages 백엔드 API (기존 유지)
└── src/
    ├── main.ts                 ← 앱 마운트 진입점
    ├── assets/main.css         ← Tailwind 유틸리티 CSS
    ├── App.vue                 ← 전체 화면 조립 및 상태 관리 최상위 컴포넌트
    └── components/             ← 기능·화면별로 분리된 10개의 전용 컴포넌트
        ├── TopNavbar.vue           ← 상단 네비게이션 및 매칭 버튼
        ├── HeroSection.vue         ← 메인 비주얼 배너
        ├── SocialProofBar.vue      ← 실시간 평점 및 신뢰 지표
        ├── FeaturesSection.vue     ← 핵심 차별점 소개
        ├── HowItWorksSection.vue   ← 3단계 이용 안내
        ├── GuideModal.vue          ← 여행 취향/일정 입력 폼 모달
        ├── ResultSection.vue       ← 추천 가이드 목록 + Gemini AI 추천사
        ├── TestimonialsSection.vue ← 여행자 후기
        ├── CtaSection.vue          ← 하단 참여 유도 배너
        └── FooterSection.vue       ← 푸터
```

그러나, 생성된 파일이 책과 100% 같지 않을 수 있음을 늘 유념합니다. 그것은 AI가 기본적으로 갖는 특성인데 동일한 프롬프트에 대해 100% 동일한 결과를 보장하지 않기 때문입니다.

{% hint style="info" %}
**💡 앞으로 작업이 끝날 때마다 커밋을 AI에게 위임하세요**

이미 우리는 1권에서부터 매 작업단위마다 커밋하는 습관을 익혀와서 생소한 것은 아닙니다. 이번 장에서도 목표인 **Vue 마이그레이션이 성공적으로 끝나면**, 그때도 AI에게 **"지금까지 작업한 내용을 요약해서 커밋해줘"** 라고 지시해 매번 커멋을 통해 새로운 복구 지점을 만들면 됩니다. 이렇게 앞으로도 하나의 작업을 끝내고 나면 AI에게 커밋을 시키는 습관을 들이면 안전하게 개발할 수 있습니다.
{% endhint %}

현재 우리 프로젝트에는 다수의 `.vue` 파일이 들어있기 때문에 IDE 내 자동추천 기능으로 우측 하단에 **"Do you want to install 'Vue - Official' extension?**"이라는 추천 팝업이 뜰 수 있습니다. 이 때는 **\[Install]** 버튼을 눌러 설치해 주세요! Vue 전용 확장 프로그램이 설치되면 `.vue` 파일 안의 HTML, JS, CSS 코드에 알록달록 색상을 입혀주고(구문 강조) 자동 완성 기능을 켜주어 코드를 읽고 쓰기가 훨씬 수월해집니다.&#x20;

{% hint style="info" %}
**💡 Extension(확장 프로그램)이 뭔가요?**

IDE의 **Extension(확장 프로그램)**&#xC740; 스마트폰의 **'앱 스토어에서 전용 앱을 설치하는 것'**&#xACFC; 같습니다. 처음에 아무것도 없는 깨끗한 스마트폰을 사서 필요한 게임이나 번역 앱을 깔듯, 개발 환경(IDE)에 특정 언어나 프레임워크(Vue, Python 등)를 더 똑똑하게 지원해 주는 보조 도구를 추가하는 것입니다.
{% endhint %}

---

## 2. 귀납적 코드 이해: AI가 만든 `.vue` 파일 해부하기

좌측 탐색기(Explorer)에서 `src/components/TopNavbar.vue` 파일을 클릭해 열어봅시다. Vue를 처음 보는 분은 낯선 기호들에 당황할 수 있지만, 사실 **1권에서 배운 HTML, JS, CSS가 하나의 파일 안에 역할별로 단정하게 구역을 나눠 모여있는 것**뿐입니다.

```vue
<!-- src/components/TopNavbar.vue -->
<script setup lang="ts">
// ← ① JavaScript/TypeScript 로직 구역
import { ref } from "vue";

const mobileMenuOpen = ref(false); // 모바일 메뉴의 열림/닫힘 상태

const toggleMobileMenu = () => {
  mobileMenuOpen.value = !mobileMenuOpen.value; // 상태를 뒤집으면 화면이 자동 반영
};
</script>

<template>
  <!-- ← ② HTML 구역 (1권의 Tailwind 클래스가 그대로!) -->
  <nav class="bg-surface/80 shadow-sm sticky top-0 backdrop-blur-md z-50">
    <div class="flex justify-between items-center w-full px-gutter h-20">
      <a class="font-bold text-[#C8953C]" href="#">Samarkand Local Mate</a>

      <!-- 모바일 메뉴 버튼 -->
      <button @click="toggleMobileMenu">
        <!-- mobileMenuOpen 값에 따라 아이콘이 자동으로 바뀜! -->
        {{ mobileMenuOpen ? "close" : "menu" }}
      </button>
    </div>

    <!-- v-if: mobileMenuOpen이 true일 때만 이 div가 화면에 나타남 -->
    <div v-if="mobileMenuOpen" class="md:hidden px-gutter py-4">
      <a href="#features">Features</a>
      <a href="#how-it-works">How It Works</a>
    </div>
  </nav>
</template>

<style scoped>
/* ← ③ CSS 구역 (Tailwind를 쓰므로 대부분 비어있음) */
</style>
```

`.vue` 파일의 구조는 **딱 세 구역**으로 나뉩니다. 1권에서 배운 것들과 대응시켜 보면 금방 이해됩니다.

| `.vue` 파일의 구역 | 1권에서 배운 것    | 역할                            |
| :----------------- | :----------------- | :------------------------------ |
| `<script setup>`   | `<script>` 안의 JS | 데이터와 동작 처리              |
| `<template>`       | `<body>` 안의 HTML | 화면에 보이는 구조              |
| `<style scoped>`   | 별도 CSS 파일      | 이 컴포넌트에만 적용되는 스타일 |

{% hint style="info" %}
**💡 Tailwind CSS는 어떻게 넘어가나요?**

1권의 `index.html`에 들어있던 `class="flex items-center ..."` 같은 Tailwind 클래스들은 Vue 컴포넌트의 `<template>` 영역으로 그대로 이동됩니다. 프롬프트에 **"Tailwind CSS 스타일 유지"**를 명시해주면, AI가 Vite 환경에 맞게 Tailwind 패키지를 설치하고 `src/assets/main.css`에 자동 연동해주므로 스타일이 깨지지 않고 깔끔하게 유지됩니다.
{% endhint %}

**이제 각 구역에서 쓰이는 핵심 문법을 하나씩 살펴봅시다.**

---

### `{{ }}` — "이 데이터를 여기에 표시해줘"

위 `TopNavbar.vue` 코드에서 `{{ mobileMenuOpen ? 'close' : 'menu' }}`라는 표현이 보입니다. 이중 중괄호 `{{ }}`는 **"이 변수의 현재 값을 HTML에 끼워 넣어줘"**라는 Vue만의 표현 방식입니다.

1권에서는 이렇게 값을 직접 집어넣었습니다:

```javascript
// 1권 바닐라 JS: DOM을 직접 찾아서 텍스트를 교체해야 했음
document.getElementById("menu-icon").innerText = isOpen ? "close" : "menu";
```

Vue에서는 이렇게 합니다:

```html
<!-- Vue: 변수 이름만 써두면 값이 바뀔 때 화면이 알아서 업데이트됨 -->
<span>{{ mobileMenuOpen ? 'close' : 'menu' }}</span>
```

`mobileMenuOpen` 값이 `false`에서 `true`로 바뀌는 순간, `<span>` 안의 텍스트가 `menu`에서 `close`로 **자동으로** 바뀝니다. `document.getElementById`를 다시 호출할 필요가 없습니다.

---

### `v-if` — "이 조건이 맞을 때만 화면에 보여줘"

모바일 메뉴 드로어 부분을 보면 `v-if="mobileMenuOpen"`이라는 낯선 속성이 보입니다.

```html
<!-- v-if: mobileMenuOpen이 true일 때만 이 블록이 화면에 나타남 -->
<div v-if="mobileMenuOpen" class="px-gutter py-4">...메뉴 항목들...</div>
```

1권에서는 이렇게 했습니다:

```javascript
// 1권 바닐라 JS: display 속성을 직접 조작해야 했음
document.getElementById("mobile-menu").style.display = isOpen
  ? "block"
  : "none";
```

Vue에서는 `v-if`라는 "마법의 속성"을 HTML 태그에 붙여두면, 조건(`mobileMenuOpen`)이 `true`일 때는 그 요소가 화면에 나타나고, `false`일 때는 아예 DOM에서 사라집니다. 코드를 읽는 것만으로 "아, 이 부분은 메뉴가 열렸을 때만 보이는구나"를 바로 파악할 수 있습니다.

---

### `@click` — "클릭하면 이 동작을 실행해줘"

버튼에 `@click="toggleMobileMenu"`라고 붙어있는 것도 보입니다.

| 1권 바닐라 JS                                    | Vue                              |
| :----------------------------------------------- | :------------------------------- |
| `element.addEventListener('click', toggleMenu)`  | `@click="toggleMobileMenu"`      |
| `element.addEventListener('submit', handleForm)` | `@submit.prevent="handleSubmit"` |
| `element.addEventListener('input', updateValue)` | `@input="updateValue"`           |

`@`는 **"이벤트가 발생하면"**을 의미합니다. `@click="toggleMobileMenu"`는 "클릭 이벤트가 발생하면 `toggleMobileMenu` 함수를 실행해줘"라고 읽으면 됩니다. 훨씬 짧고 직관적이지요?

{% hint style="tip" %}
**💡 `@submit.prevent`는 무엇인가요?**

`GuideModal.vue`의 폼 태그를 보면 `@submit.prevent="handleSubmit"` 이라고 되어 있습니다. 여기서 `.prevent`는 브라우저의 기본 동작(폼 제출 시 페이지 새로고침)을 **막아주는(prevent)** 수식어입니다. 1권에서 `event.preventDefault()`를 직접 호출했던 것과 같은 역할이지만, `@submit.prevent`처럼 한 단어에 붙여쓰는 것만으로 처리됩니다.
{% endhint %}

---

### `<script setup>` — 왜 `setup`을 붙이는가?

`setup`은 Vue 3에서 도입된 최신 문법(Composition API)입니다. 이것이 없으면 더 길고 복잡한 방식으로 작성해야 합니다.

```vue
<!-- ❌ setup이 없는 옛날 방식 (Vue 2 스타일, 더 길고 복잡함) -->
<script>
export default {
  data() {
    return { mobileMenuOpen: false };
  },
  methods: {
    toggleMobileMenu() {
      this.mobileMenuOpen = !this.mobileMenuOpen;
    },
  },
};
</script>

<!-- ✅ setup을 붙인 최신 방식 (Vue 3 Composition API, 훨씬 짧고 직관적) -->
<script setup lang="ts">
import { ref } from "vue";
const mobileMenuOpen = ref(false);
const toggleMobileMenu = () => {
  mobileMenuOpen.value = !mobileMenuOpen.value;
};
</script>
```

AI가 생성하는 코드는 모두 최신 방식을 사용합니다. **"`<script setup>`이 보이면 '최신 Vue 3 방식이구나'"** 정도만 기억해두면 됩니다.

> **Antigravity IDE 팁**: 파일을 클릭하고 AI에게 "이 파일이 뭘 하는 파일인지 초보자도 이해할 수 있게 설명해줘"라고 물어보세요. 수백 줄짜리 코드도 순식간에 파악됩니다.

---

### `<style scoped>` — CSS 충돌 전쟁의 종결

1권에서 CSS를 전역 파일 하나로 관리하다 보면, `index.html` 어딘가에서 `.title { color: red; }`를 선언했는데 전혀 엉뚱한 곳의 제목까지 빨개지는 충돌 사고가 생기곤 했습니다. `scoped`는 이 문제를 근본적으로 해결합니다.

```vue
<!-- src/components/TopNavbar.vue -->
<style scoped>
/* 이 안의 CSS는 오직 TopNavbar.vue 컴포넌트 안에서만 작동합니다 */
/* 다른 컴포넌트의 같은 클래스명과 절대 충돌하지 않습니다 */
.nav-link {
  transition: color 0.3s;
}
</style>
```

우리 앱은 Tailwind CSS를 전면 사용하므로 `<style scoped>` 영역이 거의 비어 있습니다. 특별히 Tailwind로 표현하기 어려운 복잡한 애니메이션 같은 경우에만 여기에 작성합니다.

---

## 3. 귀납적 코드 이해: `ref()` — Vue의 "살아있는 변수"

`TopNavbar.vue`의 `<script setup>`에서 `const mobileMenuOpen = ref(false);`라는 코드를 봤습니다. `ref()`는 단순히 값을 저장하는 변수가 아닙니다. **Vue가 항상 감시하고 있는 특별한 변수**입니다.

1권의 일반 변수와 비교해 보면 차이가 명확합니다:

```javascript
// 1권 — 일반 JS 변수: 값이 바뀌어도 화면은 전혀 모름
let mobileMenuOpen = false;
mobileMenuOpen = true; // 화면에는 아무 변화 없음. 따로 DOM을 업데이트해야 함

// Vue — ref 변수: 값이 바뀌는 순간 Vue가 감지하고 화면을 자동으로 업데이트
import { ref } from "vue";
const mobileMenuOpen = ref(false);
mobileMenuOpen.value = true; // 화면의 모바일 메뉴가 즉시 나타남!
```

`ref` 변수의 실제 값을 읽거나 쓸 때는 JS 코드 안에서 `.value`를 붙여야 합니다. 하지만 `<template>` HTML 구역 안에서는 Vue가 자동으로 `.value`를 처리해주므로 `{{ mobileMenuOpen }}`처럼 그냥 쓰면 됩니다.

{% hint style="tip" %}
**💡 우리 앱 `App.vue`에 이미 `ref`가 여러 개 쓰이고 있습니다**

`src/App.vue`를 열어보면 이런 코드들이 보입니다:

```ts
// 각각 화면의 특정 상태를 감시하는 "살아있는 변수"들
const isModalOpen = ref(false); // 가이드 매칭 모달의 열림/닫힘
const showResult = ref(false); // 추천 결과 섹션의 표시 여부
const isLoading = ref(false); // API 로딩 스피너 표시 여부
const guides = ref<Guide[]>([]); // 추천된 가이드 목록 (배열)
```

"Find My Guide" 버튼을 클릭하면 `isModalOpen.value = true`로 바뀌고 → Vue가 즉시 감지하고 → 모달이 화면에 나타납니다. API 호출이 끝나면 `guides.value`에 결과 배열이 담기고 → 가이드 카드 목록이 화면에 자동으로 렌더링됩니다. 이 연쇄 반응이 Vue 반응성(Reactivity)의 전부입니다.
{% endhint %}

---

## 4. 귀납적 코드 이해: Props와 Emit — 컴포넌트 간 대화

AI가 생성한 코드를 보면 `App.vue`에서 `GuideModal.vue`나 `ResultSection.vue`로 무언가를 주고받는 구조가 보입니다. 컴포넌트끼리는 **Props(아래로 데이터 전달)**와 **Emit(위로 이벤트 보내기)**이라는 두 가지 방식으로 소통합니다.

### Props — 부모가 자식에게 데이터를 내려주는 방법

```vue
<!-- 부모: src/App.vue -->
<!-- isModalOpen 데이터를 GuideModal에 전달 (":is-open"의 앞에 붙은 ":"이 핵심!) -->
<GuideModal
  :is-open="isModalOpen"
  @close="closeModal"
  @submit="handleFormSubmit"
/>
```

```vue
<!-- 자식: src/components/GuideModal.vue -->
<script setup lang="ts">
// 부모로부터 "isOpen"이라는 boolean 값을 받겠다고 선언
const props = defineProps<{
  isOpen: boolean;
}>();
</script>

<template>
  <!-- isOpen이 true일 때만 모달 전체가 화면에 표시됨 -->
  <div
    v-if="isOpen"
    class="fixed inset-0 bg-black/50 flex items-center justify-center z-50"
  >
    ...모달 내용...
  </div>
</template>
```

**Props를 레고 블록으로 이해하기:**

```
[App.vue (부모)]
    │
    │  isModalOpen (true/false) 값을 아래로 전달
    │  ─────────────────────────────────────▶  :is-open="isModalOpen"
    ↓
[GuideModal.vue (자식)]  ←  defineProps<{ isOpen: boolean }>() 로 받아서 사용
```

- **`defineProps`**: "나(자식 컴포넌트)는 부모로부터 `isOpen`이라는 데이터를 받을 수 있어"라고 선언하는 것입니다.
- **앞의 `:`(콜론)이 핵심**: 콜론이 **있으면** 변수의 실제 값(true/false 같은 데이터)을 전달하고, 콜론이 **없으면** `"isModalOpen"`이라는 그냥 텍스트 문자열이 전달됩니다.

```html
<!-- ❌ 콜론 없음: "isModalOpen"이라는 글자가 전달됨 (원하는 동작이 아님!) -->
<GuideModal is-open="isModalOpen" />

<!-- ✅ 콜론 있음: isModalOpen 변수의 실제 값 (true 또는 false)이 전달됨 -->
<GuideModal :is-open="isModalOpen" />
```

이 `:` 하나의 차이가 "글자 전달"과 "실제 데이터 전달"을 나눕니다. 처음 Vue를 배울 때 가장 많이 하는 실수이므로 꼭 기억해두세요.

### Emit — 자식이 부모에게 사건을 알리는 방법

모달 안에서 "닫기(`✕`)" 버튼을 클릭하거나 폼을 제출하면, `GuideModal.vue`는 스스로 상태를 닫을 수 없습니다. 모달의 열림/닫힘 상태(`isModalOpen`)는 `App.vue`가 관리하기 때문입니다. 이럴 때 자식이 부모에게 "이런 일이 일어났어요!"라고 신호를 보내는 것이 **Emit(이벤트 발신)**입니다.

```vue
<!-- 자식: src/components/GuideModal.vue -->
<script setup lang="ts">
// 부모에게 보낼 수 있는 이벤트 종류를 미리 선언
const emit = defineEmits<{
  (e: "close"): void; // 닫기 이벤트
  (e: "submit", data: FormData): void; // 폼 제출 이벤트 (데이터와 함께)
}>();
</script>

<template>
  <!-- ✕ 버튼 클릭 시 → 부모에게 'close' 이벤트를 발신 -->
  <button @click="emit('close')">✕</button>

  <!-- 폼 제출 시 → 부모에게 'submit' 이벤트와 함께 사용자 입력 데이터를 전달 -->
  <form @submit.prevent="handleSubmit">...</form>
</template>
```

```vue
<!-- 부모: src/App.vue -->
<!-- @close: GuideModal이 'close'를 emit하면 closeModal() 함수를 실행 -->
<!-- @submit: GuideModal이 'submit'을 emit하면 handleFormSubmit() 함수를 실행 -->
<GuideModal
  :is-open="isModalOpen"
  @close="closeModal"
  @submit="handleFormSubmit"
/>
```

**Props와 Emit을 통한 데이터 흐름:**

```
[App.vue]  ──── :is-open="isModalOpen" ──▶  [GuideModal.vue]
            ◀── @close (닫기 버튼 클릭) ────
            ◀── @submit (폼 제출 완료) + 사용자입력 데이터
```

이처럼 데이터는 항상 **위(부모)에서 아래(자식)로 흐르고**, 이벤트는 항상 **아래(자식)에서 위(부모)로 흐릅니다**. 이 규칙을 지키면 코드의 흐름을 항상 예측할 수 있어 디버깅이 매우 쉬워집니다.

---

## 5. 귀납적 코드 이해: `computed()` — 자동으로 계산되는 값

`GuideModal.vue`를 더 살펴보면 `computed`라는 또 다른 Vue 기능이 등장합니다.

```vue
<!-- src/components/GuideModal.vue -->
<script setup lang="ts">
const tourType = ref(""); // 사용자가 선택한 투어 종류 (예: 'history')

// tourType 값이 바뀔 때마다 자동으로 계산되는 "설명 문자열"
const currentTourDesc = computed(() => {
  const descriptions = {
    history: "2,000년 역사의 실크로드 도슨트 투어",
    food: "현지인만 아는 숨은 맛집 탐방",
    photo: "인스타 인생샷 스팟 투어",
  };
  return tourType.value ? descriptions[tourType.value] || "" : "";
});
</script>

<template>
  <select v-model="tourType">
    <option value="history">역사 투어 🏛️</option>
    <option value="food">음식 투어 🍜</option>
    <option value="photo">사진 투어 📸</option>
  </select>

  <!-- currentTourDesc는 tourType이 바뀔 때마다 자동으로 업데이트됨 -->
  <p v-if="currentTourDesc">{{ currentTourDesc }}</p>
</template>
```

사용자가 "역사 투어"를 선택하는 순간 `tourType.value`가 `'history'`로 바뀌고 → `currentTourDesc`가 자동으로 재계산되어 `'2,000년 역사의 실크로드 도슨트 투어'` 텍스트가 화면에 즉시 나타납니다.

{% hint style="info" %}
**💡 `ref` vs `computed` 차이**

|             | `ref`                           | `computed`                                         |
| :---------- | :------------------------------ | :------------------------------------------------- |
| **역할**    | 직접 값을 담는 "저장소"         | 다른 `ref`로부터 계산되는 "자동 계산기"            |
| **값 변경** | `.value = 새값` 으로 직접 변경  | 직접 변경 불가. 의존하는 `ref`가 바뀔 때 자동 갱신 |
| **예시**    | `tourType` (사용자 선택값 저장) | `currentTourDesc` (선택값에 따른 설명 자동 계산)   |

{% endhint %}

또한 `v-model="tourType"`이라는 표현도 보입니다. 이것은 폼 입력(`<select>`, `<input>`)과 데이터를 **양방향으로 자동 연결**해주는 마법입니다. 사용자가 셀렉트 박스에서 다른 옵션을 고르면 `tourType.value`가 자동으로 바뀌고, 반대로 코드에서 `tourType.value = 'food'`로 바꾸면 셀렉트 박스의 선택 항목이 자동으로 "음식 투어"로 바뀝니다.

---

## 6. 귀납적 이해: 바닐라 JS vs Vue.js — 무엇이 달라졌나?

이제 우리 앱 코드를 실제로 열어보며 1권과 2권의 차이를 최종 정리해봅시다.

| 항목                 | 바닐라 JS (1권 `practice.html`)          | Vue.js (2권 `src/components/*.vue`) |
| :------------------- | :--------------------------------------- | :---------------------------------- |
| 화면에 값 표시       | `element.innerText = guide.name`         | `{{ guide.name }}`                  |
| 조건부 표시/숨김     | `element.style.display = 'none'`         | `v-if="조건"`                       |
| 이벤트 처리          | `addEventListener('click', fn)`          | `@click="fn"`                       |
| 데이터를 속성에 연결 | `element.setAttribute('class', ...)`     | `:class="..."`                      |
| 폼 입력 연동         | `input.addEventListener + element.value` | `v-model="변수"`                    |
| CSS 범위             | 전역 (충돌 위험)                         | `<style scoped>` (컴포넌트 격리)    |
| 코드 분리            | 파일 하나에 모두 혼재                    | 컴포넌트 단위로 깔끔하게 분리       |
| 재사용               | 같은 코드를 여러 곳에 복사-붙여넣기      | 컴포넌트를 `<TagName />`으로 조립   |

{% hint style="success" %}
**🎯 핵심 패러다임 전환: "데이터를 바꾸면 화면이 따라온다"**

- **1권 방식**: HTML 요소를 직접 찾아(getElementById) → 값을 집어넣음 → 화면 변경
- **Vue 방식**: `ref` 데이터를 바꾸면 → Vue가 자동으로 화면을 업데이트

이 차이가 Vue 같은 프레임워크를 쓰는 가장 큰 이유입니다. 화면이 늘어나고 기능이 복잡해질수록, 일일이 DOM을 찾아서 수동으로 업데이트하는 1권 방식은 관리가 불가능에 가까워집니다. Vue에서는 데이터만 제대로 관리하면 화면은 알아서 따라옵니다.
{% endhint %}

이 표는 AI에게 설명을 요청해서 더 풍부하게 만들어달라고 해도 됩니다. "바닐라 JS와 Vue.js의 차이를 마크다운 표로 비교해줘"라고 하면 이보다 더 자세한 비교표를 즉시 만들어줍니다.
