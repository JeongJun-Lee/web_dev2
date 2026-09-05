# 제3장: TypeScript와 Vite-SSG — 안전성과 성능을 한 번에

> 🎉 이 장의 작은 성공: 빌드하면 정적 HTML 파일이 뿅 하고 생성되고, 배포 시 체감 속도가 확 빨라진다!

## 1. 프롬프팅: 데이터 타입 명세서(Interface)와 엄격한 타입 다듬기
2장에서 이미 Vite + Vue 3 프로젝트를 만들 때 TypeScript 기반(`lang="ts"`)으로 기본 뼈대를 잡았습니다. 하지만 지금까지는 타입 명세(Interface) 없이 데이터를 자유롭게 쓰고 있었습니다.

이제 Antigravity IDE를 통해 프로젝트 전체의 데이터 구조를 명확히 정의하고, 명확한 타입(Interface)을 부여하여 코드의 안전성을 극대화합니다.

> 💬 "우리 앱에서 사용하는 투어 데이터, 가이드 데이터, 사용자 예약 데이터의 구조를 `src/types/index.ts` 파일에 `interface`로 깔끔하게 정의해줘. 그리고 모든 컴포넌트와 함수에서 이 interface를 받아 쓰도록 타입을 다듬어주고, 혹시 남아있는 `any` 타입이나 애매한 타입 선언이 있다면 모두 엄격하게 고쳐줘."

AI가 작업을 진행하면서 다음 파일들을 정돈합니다:
- `src/types/index.ts` 파일이 새로 생기고, 데이터 명세서(interface)들이 정의됩니다.
- 각 `.vue` 컴포넌트의 `<script setup lang="ts">`에서 `import type { TourGuide, Booking } from '@/types'` 형태로 타입을 불러와 사용합니다.
- `tsconfig.json` 파일의 타입 검사 설정이 정교해집니다.

{% hint style="info" %}
**💡 TypeScript는 왜 필요할까요?**
2장에서 Vue 프로젝트를 생성할 때 `lang="ts"`를 붙여 이미 TypeScript 환경을 갖췄습니다. 하지만 타입 명세서(`interface`) 없이 개발하면 JavaScript처럼 데이터 모양이 불투명해집니다.

TypeScript는 JavaScript에 **"타입(종류) 검사"**를 강제하는 언어입니다. `이 변수는 반드시 숫자여야 해`, `가이드 객체에는 name과 rating이 필수야`라고 미리 선언해두면, 실수하는 순간 에디터가 빨간 밑줄로 바로 알려줍니다. 실제 사용자가 앱을 쓰다 터지는 버그를 개발 단계에서 원천 봉쇄하는 것입니다.

> 1권 부록 「JS 프레임워크와 TypeScript 소개」에서 왜 JavaScript만으론 부족하고 TypeScript가 실무 표준이 되었는지를 자세히 다뤘습니다. 아직 읽지 않으셨다면 먼저 훑어보시면 이 장의 내용이 훨씬 잘 들어옵니다.

**TypeScript가 인식하는 기본 타입**

TypeScript는 변수나 함수의 입·출력값에 아래와 같은 타입을 붙일 수 있습니다.

| 타입 | 의미 | 예시 |
|---|---|---|
| `string` | 텍스트(문자열) | `'알리'`, `"안녕하세요"` |
| `number` | 숫자 (정수·소수 모두) | `42`, `3.14`, `-7` |
| `boolean` | 참 또는 거짓 | `true`, `false` |
| `null` | 값이 명시적으로 비어 있음 | `null` |
| `undefined` | 값이 아직 할당되지 않음 | `undefined` |
| `string[]` | 문자열 배열 | `['한국어', '영어']` |
| `number[]` | 숫자 배열 | `[1, 2, 3]` |
| `any` | 아무 타입이나 허용 (사용 최소화 권장) | — |
| `void` | 함수가 아무것도 반환하지 않을 때 | `function log(): void {}` |
| `object` / `interface` | 여러 필드를 가진 객체 | `{ id: number; name: string }` |

이 표의 타입들을 다 외울 필요는 없습니다. AI가 코드를 만들어줄 때 자연스럽게 사용하므로, "아, 이 타입이 저 칸에 해당하는 거구나"라고 대조해보는 용도로 활용하세요.

**그렇다면 `interface`란?**
'Interface(인터페이스)'란 **"이 데이터 객체는 반드시 이런 모양이어야 해"**라고 미리 약속해두는 설계도입니다. 예를 들어 투어 가이드 데이터가 `이름`, `평점`, `하루 금액`이라는 세 가지 항목으로 이루어져야 한다고 약속해두면, 항목이 빠진 데이터가 들어오는 순간 TypeScript가 에러를 냅니다.

건축으로 비유하면, 설계도면(interface)을 먼저 그린 뒤 실제 건물(데이터)을 짓는 것입니다. 설계도와 다른 건물은 지을 수 없습니다.
{% endhint %}

## 2. 귀납적 코드 이해: '타입'의 선언 방식 깨우치기
AI가 만들어낸 `src/types/index.ts` 파일을 열어봅니다.

```typescript
// AI가 생성한 타입 명세서
export interface TourGuide {
  id: number
  name: string
  rating: number        // 1.0 ~ 5.0
  languages: string[]   // ['한국어', '영어', '우즈벡어']
  pricePerDay: number
  imageUrl: string
}

export interface Booking {
  id: number
  guideId: number
  userId: number
  date: string          // 'YYYY-MM-DD'
  status: 'pending' | 'confirmed' | 'cancelled'
}
```

처음에는 낯설어 보이지만, 하나씩 뜯어보면 모두 직관적인 규칙입니다.

### `interface` — 데이터의 설계도

`interface`는 **"이 데이터 객체는 반드시 이런 모양이어야 해"**라고 미리 선언하는 설계도입니다.

건축 설계도에 비유하면 이렇습니다. 설계도(`interface`)를 그려두면, 실제 건물(데이터 객체)은 반드시 그 설계도를 따라야 합니다. `id` 필드가 빠졌거나, `rating`에 숫자 대신 문자열을 넣으면 TypeScript가 즉시 빨간 줄로 경고합니다.

```typescript
// 설계도 선언
interface TourGuide {
  id: number
  name: string
}

// 설계도에 맞는 객체 ✅
const guide: TourGuide = { id: 1, name: '알리' }

// 설계도를 어긴 객체 ❌ — TypeScript 에러 발생!
const guide: TourGuide = { id: '일', name: '알리' }
//                               ^^^
//                               id는 number여야 하는데 string이 들어왔음
```

### `: string`, `: number` — 타입 지정 문법

변수 이름 뒤에 **: 타입이름**을 붙이는 것이 TypeScript의 전부입니다. JavaScript와의 차이는 이것뿐입니다.

```typescript
// JavaScript
let name = '알리'
let price = 50000

// TypeScript — 콜론(:) 뒤에 타입만 추가됨
let name: string = '알리'
let price: number = 50000
```

TypeScript의 기본 타입들:
| 타입 | 의미 | 예시 |
|---|---|---|
| `string` | 문자열 | `'알리'`, `'안녕'` |
| `number` | 숫자 (정수, 소수 모두) | `42`, `3.14` |
| `boolean` | 참/거짓 | `true`, `false` |
| `string[]` | 문자열 배열 | `['한국어', '영어']` |
| `number[]` | 숫자 배열 | `[1, 2, 3]` |

### `string[]` — 배열을 나타내는 방법

`[]`를 타입 뒤에 붙이면 **"이 타입의 배열"**을 의미합니다.

```typescript
// 그냥 문자열 하나
let language: string = '한국어'

// 문자열 여러 개의 배열
let languages: string[] = ['한국어', '영어', '우즈벡어']
```

가이드 한 명이 여러 언어를 구사할 수 있으므로 `string[]`로 선언합니다.

### `'pending' | 'confirmed' | 'cancelled'` — 유니온 타입

`|`(파이프 기호)는 **"이 중 하나"**를 의미합니다. 예약 상태는 딱 세 가지 값 중 하나만 가능해야 하는데, 이를 유니온 타입으로 강제합니다.

```typescript
// 유니온 타입 없이 — 어떤 문자열이든 들어올 수 있어 위험
let status: string = 'cancellled'  // 오타! 하지만 에러 없음 😱

// 유니온 타입으로 — 세 값 외에는 에러 발생
let status: 'pending' | 'confirmed' | 'cancelled'
status = 'cancellled'  // ← TypeScript 에러! 오타를 즉시 잡아줌 ✅
```

### 왜 AI가 TypeScript를 좋아하는가?
TypeScript가 있으면 AI는 "이 함수가 어떤 데이터를 기대하는지"를 명확히 알 수 있습니다. `TourGuide` 인터페이스가 정의되어 있으면, "가이드 카드를 그리는 함수를 만들어줘"라고 했을 때 AI가 `guide.name`, `guide.rating` 등의 필드를 자동으로 정확하게 사용합니다. 타입 정보가 없으면 AI가 추측으로 코드를 짜야 해서 오류가 많아집니다.

## 3. 프롬프팅 + 이해: 붉은 밑줄(에러)과 대화하기
TypeScript를 도입하면 초반에 에디터에 빨간 밑줄이 가득 생깁니다. 당황하지 마세요. 이것은 TypeScript가 이미 존재하던 잠재적 버그를 전부 찾아낸 것입니다.

Antigravity IDE에서 빨간 줄이 생긴 코드 위에 커서를 올리면 에러 메시지가 팝업됩니다. 해당 에러 메시지를 AI에게 보여줍니다.

> 💬 "이 에러가 왜 났어? `Argument of type 'string' is not assignable to parameter of type 'number'` 이게 무슨 뜻인지 예시로 설명해주고 고쳐줘."

이 에러를 직접 번역하면: **"'string' 타입의 값을 'number' 타입의 매개변수에 전달할 수 없습니다"**입니다.

실제로 이런 상황에서 자주 발생합니다:

```typescript
// 문제 상황: HTML <input>에서 값을 가져오면 항상 문자열!
const priceInput = document.querySelector<HTMLInputElement>('#price')
const price = priceInput.value  // "50000" ← 따옴표 있음! string!

// 에러: saveGuide 함수는 number를 기대하는데, string이 들어옴
saveGuide({ pricePerDay: price })
//                        ^^^^^
//  Argument of type 'string' is not assignable to parameter of type 'number'

// 해결: Number()로 숫자로 변환
saveGuide({ pricePerDay: Number(price) })  // 50000 ← 숫자로 변환됨 ✅
```

수정 전후를 비교하며 발견합니다:
- "숫자를 넣어야 할 곳에 문자열을 넣으니까 에러가 났구나!"
- "HTML `<input>`에서 받아온 값은 항상 `string`인데, DB에 저장할 때는 `number`로 변환해야 하는구나!"
- TypeScript의 에러는 '제약 조건 위반 알림'이고, 이 제약 덕분에 런타임 오류(실제 사용자에게 발생하는 에러)가 줄어든다는 것을 체감합니다.

> **실전 팁**: Antigravity IDE에서 빨간 밑줄이 생긴 줄에서 `Cmd+.`(맥)를 누르면 '빠른 수정' 메뉴가 나옵니다. 여기서 AI Fix를 선택하면 AI가 바로 수정안을 제시합니다.

## 4. 프롬프팅: 페이지 로딩 개선 — Vite-SSG 적용
2장까지 만든 우리 앱은 전형적인 SPA(Single Page Application)입니다. 브라우저가 텅 빈 HTML을 먼저 받은 뒤, JavaScript가 뒤늦게 화면을 그려냅니다. 이 방식은 한 번 로딩되면 부드럽지만, **첫 화면 로딩이 느리고 검색 엔진(구글)이 텅 빈 화면만 보고 지나쳐버려 검색에 잘 안 걸리는 치명적인 단점**이 있습니다.

{% hint style="info" %}
**💡 SPA vs SSG — 택배로 비유하기**

**SPA 방식**: 빈 택배 상자를 먼저 보내고, 상자 안에 '조립 설명서(JavaScript)'를 넣어둡니다. 받는 사람(브라우저)이 설명서를 읽고 직접 내용물을 조립해야 합니다. → 조립 시간만큼 기다려야 함

**SSG 방식**: 공장(서버)에서 내용물을 미리 다 조립한 완성품을 보냅니다. 받는 사람은 상자를 열자마자 바로 사용할 수 있습니다. → 기다릴 필요 없음
{% endhint %}

이를 해결하기 위해 **Vite-SSG(Static Site Generation)**를 도입합니다.
- **Vite**: 앞서 1장에서 배운 초고속 프론트엔드 빌드 툴입니다.
- **SSG (정적 사이트 생성)**: 서버에서 JavaScript를 미리 렌더링해서, 완성된 HTML 화면을 마치 '사진 찍듯이' 파일로 미리 구워두는(Generate) 기술입니다.

즉, Vite-SSG는 개발할 때는 빠르고 유연한 Vue SPA처럼 작업하고, 실제 배포할 때는 검색 엔진이 좋아하는 완성된 정적 HTML 파일들로 쪼개어 배포할 수 있게 해주는 마법 같은 플러그인입니다.

> 💬 "이 웹앱의 초기 로딩이 느린데, vite-ssg를 도입해서 빌드할 때 정적 HTML을 미리 뽑아내게 설정해줘. 라우터 설정도 같이 수정해서 각 투어 상세 페이지가 별도의 HTML로 생성되게 해."

AI가 수행하는 작업:
1. `vite-ssg` 패키지 설치
2. `vite.config.ts` 수정
3. `main.ts`를 SSG용으로 변환 (export 추가)
4. 각 라우트가 정적 HTML로 빌드되도록 설정

## 5. 귀납적 코드 이해: 빌드 결과물에서 SSG의 원리 발견하기
`npm run build`를 실행한 후 생성된 `dist/` 폴더를 Antigravity IDE에서 열어봅니다.

```
dist/
├── index.html          ← 메인 페이지
├── tours/
│   ├── index.html      ← 투어 목록 페이지
│   ├── guide-1.html    ← 가이드 1 상세 페이지
│   └── guide-2.html    ← 가이드 2 상세 페이지
└── assets/
    └── main.js
```

- **"SPA인데 왜 여러 개의 `.html` 파일이 나왔지?"**: SSG(Static Site Generation)가 빌드 시점에 각 라우트를 미리 방문해서 HTML 스냅샷을 찍어두는 방식이기 때문입니다.
- **1권의 Cloudflare Pages와의 호환**: 1권에서 이미 '정적 파일 배포'를 배웠습니다. SSG의 결과물도 똑같이 정적 HTML/JS 파일이기 때문에, 기존 Cloudflare Pages 설정을 그대로 사용해 배포할 수 있습니다.

| 방식 | 동작 원리 | 첫 로딩 | SEO |
|---|---|---|---|
| SPA | 빈 HTML + JS 실행 후 렌더링 | 느림 | 취약 |
| SSG | 미리 만들어진 HTML 바로 전달 | 빠름 | 강함 |

## 6. 귀납적 이해: `tsconfig.json` 들여다보기
TypeScript 컴파일러 설정 파일인 `tsconfig.json`을 열고 AI에게 물어봅니다.

> 💬 "이 tsconfig.json에서 가장 중요한 옵션 3개만 골라서 초보자가 이해할 수 있게 설명해줘."

AI가 생성하는 `tsconfig.json`은 대략 이런 모습입니다:

```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "lib": ["ESNext", "DOM"]
  }
}
```

가장 중요한 옵션들:

| 옵션 | 값 | 의미 |
|---|---|---|
| `"strict": true` | true | TypeScript의 가장 엄격한 검사 모드 활성화 |
| `"target": "ESNext"` | ESNext | 최신 JavaScript 문법으로 변환 |
| `"moduleResolution": "bundler"` | bundler | Vite 같은 번들러에 맞게 모듈 해석 |

**`"strict": true`**가 가장 중요합니다. 이것이 활성화되면 TypeScript가 가장 엄격하게 타입을 검사합니다. 처음에는 에러가 많아 불편하지만, 이 옵션 덕분에 나중에 실제 사용자가 마주할 수 있는 런타임 버그의 대부분이 미리 걸러집니다.

{% hint style="info" %}
**💡 strict 모드가 잡아주는 대표적인 실수들**
- `null`이나 `undefined`일 수 있는 값을 그냥 쓰는 경우
- 함수의 매개변수 타입을 선언하지 않은 경우
- 사용하지 않는 변수나 함수 등

처음에 빨간 줄이 많이 생겨도 하나씩 AI에게 물어보며 고치다 보면, 어느새 타입을 자연스럽게 쓰고 있는 자신을 발견하게 됩니다.
{% endhint %}
