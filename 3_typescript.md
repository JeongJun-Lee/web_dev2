---
hidden: true
---

# 제3장: TypeScript — 코드에 안전망 씌우기

> 🎉 이 장의 작은 성공: 에디터에 빨간 밑줄이 생기기 시작한다! 그리고 그 빨간 줄 덕분에 런타임 버그가 사라진다.

지금까지 만든 Vue 앱은 잘 동작하지만, 한 가지 잠재적인 위험이 있습니다. 데이터의 *모양*을 아무도 강제하지 않는다는 것입니다. 투어 가이드 객체에 `name`이 없어도, `rating`에 문자열을 넣어도, 실행 전까지는 아무도 모릅니다. TypeScript는 이 위험을 코드를 실행하기 *전에*, 심지어 저장하는 순간에 잡아줍니다.

## 1. TypeScript가 필요한 이유 — 실제 사고 사례로 보기

현실에서 이런 일이 벌어집니다.

```javascript
// 😱 JavaScript — 에러가 나기 전까지 아무도 모름
function showGuideCard(guide) {
  return guide.rating.toFixed(1)  // toFixed는 숫자에만 존재
}

// 개발자가 실수로 rating에 문자열을 넣음
showGuideCard({ name: '알리', rating: '4.8' })
// 💥 실행 시 오류: "rating.toFixed is not a function"
//    사용자가 앱을 쓰다가 하얀 화면을 마주침!
```

TypeScript였다면:

```typescript
// ✅ TypeScript — 저장하는 순간 에러를 잡아줌
interface TourGuide {
  name: string
  rating: number  // ← 반드시 숫자여야 함
}

function showGuideCard(guide: TourGuide) {
  return guide.rating.toFixed(1)
}

// 에디터에서 즉시 빨간 줄! ❌
showGuideCard({ name: '알리', rating: '4.8' })
//                             ^^^^^^^^^^^^^^
// 오류: 'string' 타입은 'number' 타입에 할당할 수 없습니다
```

사용자가 앱을 쓰다 터지는 것과, 개발자가 코드를 저장하는 순간 잡히는 것. 어느 쪽이 더 낫습니까? TypeScript는 실수를 가장 저렴한 순간에 잡아줍니다.

{% hint style="info" %}
**💡 TypeScript는 새로운 언어가 아닙니다**

TypeScript는 JavaScript에 타입 표기를 *추가*한 언어입니다. 브라우저는 TypeScript를 직접 실행할 수 없기 때문에, Vite가 빌드할 때 TypeScript를 다시 JavaScript로 변환합니다. 즉, 런타임 성능은 동일하고, 개발 중의 안전성만 높아집니다.

```
내가 작성      Vite 빌드       브라우저 실행
TypeScript  →  JavaScript  →  실행
(타입 검사)    (타입 제거)     (JS와 동일)
```
{% endhint %}

## 2. 기본 타입 — 딱 5가지만 알면 시작할 수 있습니다

TypeScript의 타입 표기법은 단순합니다. 변수 이름 뒤에 **`: 타입이름`** 을 붙이는 것이 전부입니다.

```typescript
// JavaScript — 타입 없음
let guideName = '알리'
let price = 50000
let isAvailable = true

// TypeScript — 콜론(:) 뒤에 타입만 추가
let guideName: string = '알리'
let price: number = 50000
let isAvailable: boolean = true
```

가장 자주 쓰이는 5가지 기본 타입:

| 타입 | 의미 | 예시 |
|------|------|------|
| `string` | 문자열 (텍스트) | `'알리'`, `"안녕하세요"` |
| `number` | 숫자 (정수·소수 모두) | `42`, `3.14`, `-7` |
| `boolean` | 참 또는 거짓 | `true`, `false` |
| `string[]` | 문자열 배열 | `['한국어', '영어']` |
| `number[]` | 숫자 배열 | `[1, 2, 3]` |

{% hint style="info" %}
**💡 타입을 꼭 직접 써야 하나요?**

아닙니다! TypeScript는 값을 보면 타입을 스스로 추론합니다. 이것을 **타입 추론**이라고 합니다.

```typescript
// 타입을 직접 쓰지 않아도
let guideName = '알리'
// TypeScript가 자동으로 string으로 추론함

// 이후 숫자를 넣으려 하면 자동으로 에러 발생
guideName = 42  // ❌ 오류: string에 number를 넣을 수 없음
```

초보자는 처음엔 AI가 써주는 타입 코드를 그냥 따라 쓰면서 감을 익히는 것이 좋습니다. 나중에 익숙해지면 직접 쓰게 됩니다.
{% endhint %}

## 3. 배열 타입 — `string[]`이 의미하는 것

`[]`를 타입 뒤에 붙이면 "이 타입의 배열"을 뜻합니다.

```typescript
// 문자열 하나
let language: string = '한국어'

// 문자열 여러 개 (배열)
let languages: string[] = ['한국어', '영어', '우즈벡어']

// 숫자 배열
let ratings: number[] = [4.5, 4.8, 4.2]
```

투어 가이드 한 명이 여러 언어를 구사할 수 있으므로 `string[]`으로 선언합니다.

## 4. 유니온 타입 — "이 중 하나"

`|`(파이프) 기호는 "이 중 하나여야 해"를 의미합니다.

예약의 상태는 딱 세 가지 값 중 하나여야 합니다 — 대기 중, 확정, 취소. 이 외의 값이 들어오면 바로 에러로 잡아야 합니다.

```typescript
// ❌ 유니온 타입 없이 — 어떤 문자열이든 들어올 수 있어 위험
let status: string = 'cancellled'  // 오타! 하지만 에러 없음 😱

// ✅ 유니온 타입으로 — 세 값 외에는 에러 발생
let status: 'pending' | 'confirmed' | 'cancelled'
status = 'cancellled'  // ← TypeScript 에러! 오타를 즉시 잡아줌 ✅
```

에러 메시지: `Type '"cancellled"' is not assignable to type '"pending" | "confirmed" | "cancelled"'`

단순히 문자열로 선언했다면 절대 잡지 못할 오타를 TypeScript가 잡아줍니다.

## 5. 인터페이스(interface) — 데이터의 설계도

지금까지 배운 기본 타입들은 변수 하나하나에 타입을 붙이는 것이었습니다. 하지만 실제 앱에서 데이터는 여러 필드가 묶인 *객체* 형태로 다닙니다.

```typescript
// 투어 가이드 데이터 — 여러 필드가 묶인 객체
const guide = {
  id: 1,
  name: '알리',
  rating: 4.8,
  languages: ['한국어', '영어'],
  pricePerDay: 80000,
  imageUrl: '/guides/ali.jpg'
}
```

이 객체의 *모양*을 미리 약속해두는 것이 바로 **인터페이스(interface)**입니다.

```typescript
// 인터페이스 — 데이터의 설계도
interface TourGuide {
  id: number
  name: string
  rating: number        // 1.0 ~ 5.0
  languages: string[]   // ['한국어', '영어', '우즈벡어']
  pricePerDay: number
  imageUrl: string
}
```

이 설계도를 그려두면, 이 모양을 따르지 않는 객체는 즉시 에러가 됩니다.

```typescript
// 설계도에 맞는 객체 ✅
const guide: TourGuide = {
  id: 1,
  name: '알리',
  rating: 4.8,
  languages: ['한국어', '영어'],
  pricePerDay: 80000,
  imageUrl: '/guides/ali.jpg'
}

// 설계도를 어긴 객체 ❌ — 저장하는 순간 에러!
const guide2: TourGuide = {
  id: '일',       // ← number여야 하는데 string이 들어옴
  name: '알리',
  // rating이 빠짐! ← 필수 필드 누락
  pricePerDay: 80000,
  imageUrl: '/guides/ali.jpg'
}
```

{% hint style="info" %}
**💡 건축 설계도 비유**

인터페이스는 건물의 설계도면과 같습니다.

- **설계도(`interface`)**: "이 건물은 101호, 102호, 103호가 있어야 하고, 각 호수에는 창문이 하나씩 있어야 해"
- **실제 건물(데이터 객체)**: 설계도대로 지어야 함. 호수가 빠지거나, 창문 대신 문을 달면 허가가 안 남

TypeScript는 설계도와 다른 건물(데이터)을 짓지 못하게 막아줍니다.
{% endhint %}

## 6. 프롬프팅: 인터페이스 파일 만들기

이제 AI를 통해 프로젝트 전체의 데이터 구조를 명확히 정의합니다.

> 💬 "우리 앱에서 사용하는 투어 데이터, 가이드 데이터, 사용자 예약 데이터의 구조를 `src/types/index.ts` 파일에 `interface`로 깔끔하게 정의해줘. 그리고 모든 컴포넌트와 함수에서 이 interface를 받아 쓰도록 타입을 다듬어주고, 혹시 남아있는 `any` 타입이나 애매한 타입 선언이 있다면 모두 엄격하게 고쳐줘."

AI가 생성하는 `src/types/index.ts`:

```typescript
// src/types/index.ts — 프로젝트 전체의 데이터 설계도 모음

export interface TourGuide {
  id: number
  name: string
  rating: number          // 1.0 ~ 5.0
  languages: string[]     // ['한국어', '영어', '우즈벡어']
  pricePerDay: number
  imageUrl: string
  bio?: string            // ? 가 붙으면 선택적(없어도 됨)
}

export interface Tour {
  id: number
  title: string
  description: string
  guideId: number
  maxParticipants: number
  durationHours: number
  tags: string[]
}

export interface Booking {
  id: number
  guideId: number
  userId: number
  date: string            // 'YYYY-MM-DD'
  status: 'pending' | 'confirmed' | 'cancelled'
  totalPrice: number
}

export interface User {
  id: number
  email: string
  name: string
  createdAt: string
}
```

AI가 이 파일을 만들면서 동시에 각 `.vue` 컴포넌트에도 타입을 연결합니다:

```typescript
// GuideCard.vue — 타입을 불러와 사용
<script setup lang="ts">
import type { TourGuide } from '@/types'

const props = defineProps<{
  guide: TourGuide  // TourGuide 설계도를 따르는 객체만 받을 수 있음
}>()
</script>
```

### `bio?: string` — 선택적 필드(Optional)

`?`가 붙은 필드는 있어도 되고 없어도 됩니다. 모든 가이드가 자기소개문을 가지고 있지는 않으니, `bio`는 선택적으로 선언합니다.

```typescript
// bio 없이도 유효한 TourGuide ✅
const guide: TourGuide = {
  id: 1,
  name: '알리',
  rating: 4.8,
  languages: ['한국어'],
  pricePerDay: 80000,
  imageUrl: '/guides/ali.jpg'
  // bio 없어도 OK
}

// bio 있어도 유효 ✅
const guide2: TourGuide = {
  id: 2,
  name: '보보',
  rating: 4.5,
  languages: ['영어', '우즈벡어'],
  pricePerDay: 70000,
  imageUrl: '/guides/bobo.jpg',
  bio: '사마르칸트 태생, 10년 경력의 현지 가이드입니다.'
}
```

## 7. 귀납적 이해: 인터페이스 확장(extends)

실제 프로젝트에서는 인터페이스를 상황에 맞게 *확장*하기도 합니다. AI가 생성한 코드에서 이런 패턴을 발견할 수 있습니다.

```typescript
// 기본 가이드 인터페이스
interface TourGuide {
  id: number
  name: string
  rating: number
}

// extends — 기존 설계도를 그대로 물려받아 확장
interface DetailedTourGuide extends TourGuide {
  // TourGuide의 모든 필드(id, name, rating)를 자동으로 포함
  bio: string
  availableDates: string[]
}
```

`DetailedTourGuide`는 `TourGuide`의 모든 필드를 자동으로 포함하면서, 추가 필드를 더 가집니다.

언제 확장을 쓰나요?

- 목록 화면: 간단한 `TourGuide`만 필요 (이름, 평점, 가격)
- 상세 화면: `DetailedTourGuide`가 필요 (바이오, 예약 가능 날짜)
- API 응답마다 돌아오는 필드가 다를 때

## 8. TypeScript 에러와 대화하기

TypeScript를 도입하면 초반에 에디터에 빨간 밑줄이 가득 생깁니다. 당황하지 마세요. 이것은 TypeScript가 이미 존재하던 잠재적 버그를 전부 찾아낸 것입니다.

### 자주 만나는 에러 메시지

**에러 1: `string` 타입을 `number`에 넣으려 할 때**

```
Argument of type 'string' is not assignable to parameter of type 'number'
```

한국어 번역: "string 타입의 값을 number 타입의 매개변수에 전달할 수 없습니다"

```typescript
// 원인: HTML <input>에서 가져온 값은 항상 문자열!
const priceInput = document.querySelector<HTMLInputElement>('#price')
const price = priceInput.value  // "50000" ← 따옴표 있음! string!

saveGuide({ pricePerDay: price })
//                        ^^^^^
// 에러: string은 number에 할당할 수 없음

// 해결: Number()로 숫자로 변환
saveGuide({ pricePerDay: Number(price) })  // 50000 ← 숫자로 변환됨 ✅
```

**에러 2: 없을 수도 있는 값(`null`)을 그냥 쓸 때**

```
Object is possibly 'null'
```

```typescript
// querySelector가 요소를 못 찾으면 null을 반환함
const button = document.querySelector('#submit-btn')
button.click()  // ❌ button이 null일 수도 있어서 에러!

// 해결 1: null 체크 후 사용
if (button) {
  button.click()  // ✅ null이 아닌 경우에만 실행
}

// 해결 2: 옵셔널 체이닝 — null이면 그냥 건너뜀
button?.click()  // ✅
```

### 에러를 AI에게 붙여넣기

에러 메시지가 낯설더라도, 그대로 AI에게 던지면 됩니다.

> 💬 "이 에러가 왜 났어? `Argument of type 'string' is not assignable to parameter of type 'number'` 이게 무슨 뜻인지 예시로 설명해주고 고쳐줘."

{% hint style="info" %}
**💡 빠른 수정 단축키**

Antigravity IDE에서 빨간 밑줄이 있는 줄에 커서를 놓고 `Cmd+.`(맥) 또는 `Ctrl+.`(윈도우)을 누르면 빠른 수정 메뉴가 나옵니다. 여기서 **AI Fix**를 선택하면 AI가 바로 수정안을 제시합니다.
{% endhint %}

## 9. 왜 AI와 TypeScript가 궁합이 좋은가?

TypeScript 인터페이스가 정의되어 있으면, AI가 코드를 훨씬 정확하게 만들어 줍니다.

```
❌ 타입 없이 프롬프팅:
   "가이드 카드 컴포넌트 만들어줘"
   → AI가 어떤 필드가 있는지 추측해서 코드를 짜야 함
   → guide.price? guide.pricePerDay? guide.dailyFee? 뭐가 맞는지 모름

✅ 타입 있는 상태에서 프롬프팅:
   "TourGuide 인터페이스를 props로 받는 가이드 카드 컴포넌트 만들어줘"
   → AI가 인터페이스를 보고 정확한 필드명으로 코드를 생성
   → guide.pricePerDay, guide.languages 등 오타 없음
```

인터페이스는 AI와의 소통 채널이기도 합니다. 인터페이스가 잘 정의될수록 AI가 더 정확한 코드를 만들어 줍니다.

## 10. `tsconfig.json` — TypeScript 엄격함 조절하기

TypeScript 컴파일러 설정 파일 `tsconfig.json`을 열어봅니다.

> 💬 "이 tsconfig.json에서 가장 중요한 옵션 3개만 골라서 초보자가 이해할 수 있게 설명해줘."

```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ESNext",
    "moduleResolution": "bundler",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

| 옵션 | 값 | 의미 |
|------|-----|------|
| `"strict": true` | true | TypeScript의 가장 엄격한 검사 모드 활성화 |
| `"target": "ESNext"` | ESNext | 최신 JavaScript 문법으로 변환 |
| `"moduleResolution": "bundler"` | bundler | Vite 같은 번들러에 맞게 모듈 해석 |
| `"paths": { "@/*": ["./src/*"] }` | — | `@/components/...` 같은 절대 경로 별칭 |

**`"strict": true`가 가장 중요합니다.** 이것이 활성화되면 TypeScript가 가장 엄격하게 타입을 검사합니다. 처음에는 에러가 많아 불편하지만, 이 옵션 덕분에 나중에 실제 사용자가 마주할 수 있는 런타임 버그의 대부분이 미리 걸러집니다.

{% hint style="info" %}
**💡 `strict` 모드가 잡아주는 대표적인 실수들**

- `null`이나 `undefined`일 수 있는 값을 그냥 쓰는 경우
- 함수의 매개변수 타입을 선언하지 않은 경우
- 함수가 `return`하지 않을 수도 있는 경우

처음에 빨간 줄이 많이 생겨도 하나씩 AI에게 물어보며 고치다 보면, 어느새 타입을 자연스럽게 쓰고 있는 자신을 발견하게 됩니다.
{% endhint %}

---

이 장에서 배운 것들:

- **기본 타입** (`string`, `number`, `boolean`, `[]`): 변수 하나에 붙이는 가장 기본적인 타입 표기
- **유니온 타입** (`|`): 정해진 값 중 하나만 허용
- **인터페이스** (`interface`): 객체의 모양을 설계도로 미리 약속
- **선택적 필드** (`?`): 없어도 되는 필드
- **확장** (`extends`): 기존 인터페이스를 물려받아 확장
- **에러와 대화하기**: 빨간 줄 에러 메시지를 AI에게 던지는 습관

다음 장에서는 이 TypeScript 타입 안전성 위에, 빠른 초기 로딩과 SEO를 동시에 해결하는 **Vite-SSG**를 적용합니다.
