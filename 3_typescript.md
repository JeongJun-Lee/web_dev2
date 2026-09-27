# 제3장: TypeScript — 코드에 안전망 씌우기

> 🎉 이 장의 작은 성공: 에디터에 빨간 밑줄이 생기기 시작한다! 그리고 그 빨간 줄 덕분에 런타임 버그가 사라진다.

지금까지 만든 JS(JavaScript) 앱은 잘 동작했습니다. 하지만 그럼에도 불구하고, 지금 우리가 쓰는 JavaScript에는 한 가지 잠재적인 위험이 있습니다. 바로 데이터의 _종&#xB958;_&#xB97C; 아무도 강제하지 않는다는 점입니다. 이 장에서는 그 문제를 해결해주는 TypeScript를 배웁니다.

TypeScript를 처음 보면 낯설어 보이지만, 실은 여러분이 이미 아는 JavaScript에 딱 한 가지만 추가한 것입니다. 그 한 가지가 무엇인지, 왜 필요한지부터 천천히 살펴봅시다.

## 1. TypeScript가 필요한 이유 — 아주 단순한 예시부터

### 🧃 상자에 라벨 붙이기

택배 창고를 상상해보세요. 상자가 수백 개 있는데, 라벨이 하나도 없습니다. 어떤 상자에 유리가 들어있는지, 어떤 상자에 옷이 들어있는지 뜯어보기 전까지는 모릅니다. 누군가 유리 상자를 발로 차도, 뜯어보기 전에는 모릅니다.

JavaScript 변수가 딱 이런 상황입니다.

```javascript
// samarkand-local-mate에서 하루 투어 비용(달러)을 계산할 때
let tourPrice = 50; // $50

// 나중에 누군가 실수로 문자열을 넣어도 아무도 모름
tourPrice = "오십달러"; // 😱 에러 없이 그냥 통과

// 이제 tourPrice를 숫자로 써야 하는 곳에서 앱이 터짐
console.log(tourPrice * 1.1); // NaN (숫자가 아님)
```

TypeScript는 상자에 라벨을 붙이는 것입니다.

```typescript
// TypeScript — 변수에 라벨(타입)이 붙음
let tourPrice: number = 50;

// 다른 종류를 넣으려 하면 즉시 에러!
tourPrice = "오십달러"; // ❌ 에디터에 빨간 줄!
// 오류: 'string' 형식은 'number' 형식에 할당할 수 없습니다
```

에러가 _앱이 실행될 때_ 나는 것이 아니라, _코드를 저장하는 순간_ 납니다. 사용자가 앱을 쓰다가 하얀 화면을 보는 상황을 개발자 혼자 에디터에서 조용히 처리할 수 있습니다.

### 실제 사고 사례

투어 가격을 화면에 표시하는 함수를 봅시다.

```javascript
// JavaScript — 함수가 무엇을 받는지 아무 보장이 없음
function formatTourPrice(price) {
  return "$" + price.toFixed(0); // toFixed는 숫자에만 있는 기능
}

// 누군가 문자열을 실수로 넘김
formatTourPrice("50");
// 💥 실행 시 오류: "price.toFixed is not a function"
//    사용자가 실제로 앱을 쓰다가 이 오류를 만남!
```

TypeScript였다면:

```typescript
// TypeScript — 함수가 무엇을 받는지 미리 선언
function formatTourPrice(price: number) {
  return "$" + price.toFixed(0);
}

// 에디터에서 즉시 빨간 줄! 실행 전에 잡힘 ✅
formatTourPrice("50");
//              ^^^^
// 오류: 'string' 형식의 인수는 'number' 형식의 매개 변수에 할당할 수 없습니다
```

**이것이 TypeScript의 핵심입니다.** 실수를 가장 저렴한 순간(코드 저장)에 잡아줍니다.

{% hint style="info" %}
**💡 TypeScript는 새로운 언어가 아닙니다**

TypeScript는 JavaScript에 타입 표기를 _추&#xAC00;_&#xD55C; 언어입니다. 브라우저는 TypeScript를 직접 실행할 수 없기 때문에, Vite가 빌드할 때 TypeScript를 다시 JavaScript로 변환합니다. 즉, 런타임 성능은 동일하고, 개발 중의 안전성만 높아집니다.

```
내가 작성         Vite 빌드       브라우저 실행
TypeScript  →  JavaScript   →   실행
 (타입 검사)      (타입 제거)      (JS와 동일)
```
{% endhint %}

***

## 2. 기본 타입 — 딱 5가지만 알면 시작할 수 있습니다

TypeScript의 타입 표기법은 단순합니다. 변수 이름 뒤에 **`: 타입이름`** 을 붙이는 것이 전부입니다. `samarkand-local-mate`에서 실제로 쓰이는 변수들을 예로 살펴봅시다.

```typescript
// JavaScript — 타입 없음
let guideName = "알리셰르";
let tourPrice = 50;
let isModalOpen = false;

// TypeScript — 콜론(:) 뒤에 타입만 추가
let guideName: string = "알리셰르";
let tourPrice: number = 50;
let isModalOpen: boolean = false;
```

### 5가지 기본 타입

| 타입         | 의미            | `samarkand-local-mate` 실제 예시 |
| ---------- | ------------- | ------------------ |
| `string`   | 문자열 (텍스트)     | `'알리셰르'`, `'history'`, `'홍길동'` |
| `number`   | 숫자 (정수·소수 모두) | `4.9` (평점), `50` (투어 비용), `1` (가이드 ID) |
| `boolean`  | 참 또는 거짓       | `isModalOpen` (모달 열림 여부), `isLoading` |
| `string[]` | 문자열 배열        | `['한국어', '우즈베크어', '러시아어']` (구사 언어 목록) |
| `number[]` | 숫자 배열         | `[4.9, 4.8, 5.0]` (가이드 평점 목록) |

일상 언어로 대응시켜보면:

* `string` = 이름, 주소, 설명처럼 **글자로 된 것**
* `number` = 가격, 평점, 나이처럼 **숫자인 것**
* `boolean` = 모달 열림 여부, 로딩 여부처럼 **예/아니오 두 가지 중 하나인 것**

### 타입을 잘못 쓰면 어떻게 되나요?

```typescript
let guideName: string = "알리셰르";
guideName = 42; // ❌ 빨간 줄: string에 number를 넣을 수 없음

let tourPrice: number = 50;
tourPrice = "오십달러"; // ❌ 빨간 줄: number에 string을 넣을 수 없음

let isModalOpen: boolean = false;
isModalOpen = "yes"; // ❌ 빨간 줄: boolean에 string을 넣을 수 없음
```

에디터에서 즉시 빨간 줄로 알려줍니다. 실행해보기 전에 실수를 알 수 있습니다.

{% hint style="info" %}
**💡 타입을 꼭 직접 써야 하나요?**

아닙니다! TypeScript는 값을 보면 타입을 스스로 추론합니다. 이것을 **타입 추론**이라고 합니다.

```typescript
// 타입을 직접 쓰지 않아도
let guideName = "알리셰르";
// TypeScript가 자동으로 string으로 추론함

// 이후 숫자를 넣으려 하면 자동으로 에러 발생
guideName = 42; // ❌ 오류: string에 number를 넣을 수 없음
```

초보자는 처음엔 AI가 써주는 타입 코드를 그냥 따라 쓰면서 감을 익히는 것이 좋습니다. 나중에 익숙해지면 직접 쓰게 됩니다.
{% endhint %}

## 3. 배열 타입 — `string[]`이 의미하는 것

`[]`를 타입 뒤에 붙이면 "이 타입의 배열"을 뜻합니다.

```typescript
// 문자열 하나
let language: string = "한국어";

// 문자열 여러 개 (배열) — 가이드가 구사하는 언어 목록
let languages: string[] = ["한국어", "우즈베크어", "러시아어"];

// 숫자 배열 — 등록된 가이드들의 평점 목록
let ratings: number[] = [4.9, 4.8, 5.0];
```

투어 가이드 한 명이 여러 언어를 구사할 수 있으므로 `string[]`으로 선언합니다.

### 배열 타입이 왜 유용한가요?

배열 타입이 있으면 배열 안에 이상한 것이 섞이는 걸 막아줍니다.

```typescript
let languages: string[] = ["한국어", "우즈베크어"];

// 배열에 숫자를 넣으려 하면 에러
languages.push(42); // ❌ 빨간 줄!
languages.push("러시아어"); // ✅ 정상

// 배열 요소를 꺼내 쓸 때 타입도 자동으로 알고 있음
const first = languages[0]; // TypeScript가 first는 string임을 앎
first.toUpperCase(); // ✅ 문자열 메서드 자동완성 가능
```

***

## 4. 유니온 타입 — "이 중 하나"

`|`(파이프) 기호는 "이 중 하나여야 해"를 의미합니다.

`samarkand-local-mate`의 가이드 매칭 폼을 떠올려보세요. 여행자가 선택할 수 있는 관심 투어는 역사(`history`), 미식(`food`), 사진(`photo`) 딱 3가지뿐이어야 합니다. 이 외의 오타나 엉뚱한 값이 들어오면 즉시 에러로 잡아야 합니다.

```typescript
// ❌ 유니온 타입 없이 — 어떤 문자열이든 들어올 수 있어 오타에 무방비
let tourType: string = "histroy"; // 😱 'history'의 오타지만 에러 없이 통과!

// ✅ 유니온 타입으로 — samarkand-local-mate가 제공하는 3가지 투어 유형만 허용
let tourType: "history" | "food" | "photo";
tourType = "histroy"; // ← TypeScript 에러! 오타를 즉시 잡아줌 ✅
```

에러 메시지: `Type '"histroy"' is not assignable to type '"history" | "food" | "photo"'`

단순히 문자열(`string`)로 선언했다면 런타임에 엉뚱한 결과가 나오고 나서야 알았을 오타를, TypeScript가 코드를 쓰는 순간 잡아줍니다.

### 유니온 타입 활용 예시

```typescript
// samarkand-local-mate의 실제 하루 예산 구간 (GuideModal.vue)
let budget: "under_30" | "30_60" | "over_60";

budget = "30_60"; // ✅ 정상 ($30 ~ $60)
budget = "cheap"; // ❌ 빨간 줄 — 'cheap'은 허용된 값이 아님

// 허용된 값만 쓸 수 있으니, 에디터가 자동완성도 제시해줌
// budget = 'u...' 까지만 쳐도 'under_30'을 자동 추천!
```

***

## 5. 객체란 무엇인가 — 인터페이스 이해의 전 단계

이 절은 인터페이스를 배우기 전에 꼭 읽어야 합니다. **객체**가 무엇인지 모른다면 인터페이스도 이해하기 어렵기 때문입니다.

### 객체 = 관련된 데이터를 묶은 것

투어 가이드 한 명의 정보를 코드로 표현한다고 생각해보세요. 이름, 전문 분야, 평점, 구사 언어가 있습니다. 각각 따로 변수를 만들 수 있습니다.

```javascript
// 변수를 따로따로 만드는 방식
let guideName = "알리셰르";
let guideSpecialty = "역사 및 고건축";
let guideRating = "4.9";
let guideLanguages = "한국어, 우즈베크어, 러시아어";
```

하지만 가이드가 10명이면? 변수가 40개가 됩니다. 관리가 불가능해집니다.

**객체(Object)**&#xB294; 관련된 여러 정보를 하나로 묶는 방법입니다.

```javascript
// 객체로 묶기 — 중괄호 {} 안에 key: value 쌍을 나열 (samarkand-local-mate의 실제 가이드 데이터)
const guide = {
  id: 1,
  name: "알리셰르 (Alisher)",
  specialty: "역사 및 고건축",
  tour_type: "history",
  rating: "4.9",
  languages: "한국어, 우즈베크어, 러시아어",
  description: "사마르칸트 국립대 역사학과 출신으로 레기스탄과 샤히진다의 숨겨진 역사를 깊이 있게 전달합니다."
};
```

중괄호 `{}` 안에 **이름(key): 값(value)** 형태로 정보를 넣습니다. 각 쌍은 쉼표로 구분합니다.

객체 안의 값을 꺼내려면 점(`.`)을 씁니다.

```javascript
console.log(guide.name);      // '알리셰르 (Alisher)'
console.log(guide.specialty); // '역사 및 고건축'
console.log(guide.rating);    // '4.9'
```

### 객체의 비유

명함을 떠올려보세요. 명함 한 장에는 이름, 직함, 전화번호, 이메일이 함께 적혀 있습니다. 이름만 따로, 전화번호만 따로 관리하지 않고 한 장에 묶어서 관리합니다. 객체가 바로 이 명함과 같습니다.

```javascript
// 명함 = 객체
const businessCard = {
  name: "홍길동",
  title: "웹 개발자",
  phone: "010-1234-5678",
  email: "honggd@example.com",
};
```

***

## 6. 인터페이스(interface) — 데이터의 설계도

이제 객체가 무엇인지 알았으니, 인터페이스를 이해할 준비가 됐습니다.

여러 개의 투어 가이드 객체를 만들 때, 모양이 다 다르면 어떻게 될까요?

```javascript
// 이 객체는 specialty를 씀
const guide1 = { name: "알리셰르", specialty: "역사 및 고건축" };

// 저 객체는 expertArea를 씀 (같은 의미인데 이름이 다름!)
const guide2 = { name: "딜쇼드", expertArea: "로컬 미식" };

// 이 객체는 category를 씀
const guide3 = { name: "니고라", category: "포토스팟" };
```

세 객체가 모두 "전문 분야"를 가지고 있지만 이름이 제각각입니다. 이 객체들을 받아 화면에 표시하는 컴포넌트는 어떤 속성 이름을 써야 할지 알 수가 없습니다.

**인터페이스(interface)**&#xB294; 객체의 _모&#xC591;_&#xC744; 미리 약속해두는 것입니다. 건물 설계도처럼, 만들기 전에 어떤 필드를 가져야 하는지 정의합니다.

`interface` 키워드 다음에 이름을 쓰고, 중괄호 안에 각 필드의 이름과 타입을 선언합니다. `samarkand-local-mate`의 실제 가이드 인터페이스를 봅시다:

```typescript
// samarkand-local-mate의 가이드 설계도 (src/components/ResultSection.vue)
export interface Guide {
  id?: number;
  name: string;
  specialty?: string;
  tour_type?: string;
  rating?: string;
  languages?: string;
  description?: string;
}
```

이 설계도를 그려두면, 이 모양을 따르지 않는 객체는 즉시 에러가 됩니다.

```typescript
// 설계도에 맞는 객체 ✅
const guide: Guide = {
  id: 1,
  name: "알리셰르 (Alisher)",
  specialty: "역사 및 고건축",
  tour_type: "history",
  rating: "4.9",
  languages: "한국어, 우즈베크어, 러시아어",
  description: "사마르칸트 국립대 역사학과 출신으로 레기스탄과 샤히진다의 숨겨진 역사를 깊이 있게 전달합니다."
};

// 설계도를 어긴 객체 ❌ — 저장하는 순간 에러!
const guide2: Guide = {
  id: "일", // ← number여야 하는데 string이 들어옴
  // name이 빠짐! ← 필수 필드 누락
  specialty: "역사 및 고건축"
};
```

{% hint style="info" %}
**💡 건축 설계도 비유**

인터페이스는 건물의 설계도면과 같습니다.

* **설계도(`interface`)**: "이 건물은 101호, 102호, 103호가 있어야 하고, 각 호수에는 창문이 하나씩 있어야 해"
* **실제 건물(데이터 객체)**: 설계도대로 지어야 함. 호수가 빠지거나, 창문 대신 문을 달면 허가가 안 남

TypeScript는 설계도와 다른 건물(데이터)을 짓지 못하게 막아줍니다.
{% endhint %}

### 인터페이스가 없으면 생기는 문제

인터페이스가 없을 때와 있을 때를 비교해봅시다.

```javascript
// ❌ 인터페이스 없음 — 함수가 어떤 모양의 객체를 받는지 모름
function showGuideCard(guide) {
  return guide.name + " (" + guide.specialty + " · ⭐ " + guide.rating + ")";
  // guide.specialty나 rating이 존재하는지 아무도 보장 못함
}

// 잘못된 데이터를 넘겨도 에러 없이 통과
showGuideCard({ name: "알리셰르", expertArea: "역사 및 고건축", score: "4.9" });
// specialty 대신 expertArea를 씀! 결과: "알리셰르 (undefined · ⭐ undefined)" 😱
```

```typescript
// ✅ 인터페이스 있음 — 함수가 정확히 무엇을 받는지 선언됨
function showGuideCard(guide: Guide) {
  return `${guide.name} (${guide.specialty} · ⭐ ${guide.rating})`;
}

// 잘못된 데이터를 넘기면 즉시 에러
showGuideCard({ name: "알리셰르", expertArea: "역사 및 고건축", score: "4.9" });
//             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
// 오류: 개체 리터럴은 알려진 속성만 지정할 수 있으며 'Guide' 형식에 'expertArea'이(가) 없습니다.
```

***

## 7. 프롬프팅: 컴포넌트 간 데이터 전달에 인터페이스 적용하기

이제 AI를 통해 `samarkand-local-mate` 프로젝트의 컴포넌트들에 타입을 명확히 연결합니다.

> 💬 "우리 `samarkand-local-mate` 앱의 컴포넌트들(`GuideModal.vue`, `ResultSection.vue`, `App.vue`)에서 사용하는 `Guide`와 `FormData` 같은 인터페이스를 점검하고, 컴포넌트 간 데이터를 주고받는 `props`, `emit`, 그리고 `ref` 상태 변수에 TypeScript 타입을 엄격하게 연결해줘."

AI가 컴포넌트들에 작성한 실제 코드를 살펴봅시다:

### 1. `src/components/GuideModal.vue` — 폼 입력 규격과 이벤트 정의

```vue
<!-- src/components/GuideModal.vue -->
<script setup lang="ts">
import { ref, computed, watch } from 'vue';

// 1. 여행자가 입력하는 취향 폼 데이터 설계도 선언 및 export
export interface FormData {
  userName: string;
  tourType: string;
  tourLabel: string;
  budget: string;
  budgetLabel: string;
  startDate: string;
  endDate: string;
}

// 2. 부모(App.vue)로부터 전달받는 props 타입
const props = defineProps<{
  isOpen: boolean; // 모달이 열려 있는지 여부 (참/거짓)
}>();

// 3. 부모(App.vue)로 올려보내는 emit 이벤트 타입 정의
const emit = defineEmits<{
  (e: 'close'): void;
  (e: 'submit', data: FormData): void; // submit 이벤트 발생 시 반드시 FormData 규격 강제!
}>();
</script>
```

### 2. `src/components/ResultSection.vue` — 추천 가이드 목록 타입 정의

```vue
<!-- src/components/ResultSection.vue -->
<script setup lang="ts">
// 가이드 데이터 설계도 선언 및 export
export interface Guide {
  id?: number;
  name: string;
  specialty?: string;
  tour_type?: string;
  rating?: string;
  languages?: string;
  description?: string;
}

// 부모 컴포넌트로부터 받는 props 타입 정의
defineProps<{
  isVisible: boolean;
  isLoading: boolean;
  errorMessage?: string;
  userName: string;
  tourName: string;
  budgetName: string;
  guides: Guide[];              // ← Guide 객체들의 배열!
  isAiLoading: boolean;
  aiRecommendation: string;
}>();
</script>
```

### 3. `src/App.vue` — 최상위 컴포넌트에서 타입 연결

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { ref, nextTick } from 'vue';
import GuideModal, { FormData } from './components/GuideModal.vue';
import ResultSection, { Guide } from './components/ResultSection.vue';

const isModalOpen = ref(false);
const showResult = ref(false);
const isLoading = ref(false);
const isAiLoading = ref(false);
const errorMessage = ref('');

// ref 상태 변수에 Guide[] 타입 명시 (가이드 배열만 들어올 수 있음)
const guides = ref<Guide[]>([]);
const aiRecommendation = ref('');

// 자식 컴포넌트(GuideModal)에서 올려준 formData의 타입 검증
const handleFormSubmit = async (formData: FormData) => {
  resUserName.value = formData.userName;
  resTourName.value = formData.tourLabel;
  resBudgetName.value = formData.budgetLabel;
  // ...
};
</script>
```

### `export`와 `import`는 왜 쓰나요?

`GuideModal.vue`에서 `export interface FormData`로 설계도를 내보내고, `App.vue`에서 `import { FormData } from './components/GuideModal.vue'`로 가져옵니다.

컴포넌트가 달라도 하나의 동일한 설계도를 공유하므로, 모달에서 올려준 데이터의 필드명(`formData.userName`)과 부모가 받는 필드명이 100% 일치함을 TypeScript가 보증합니다.

```
   설계도를 만든 곳                       가져다 쓰는 곳
GuideModal.vue               →              App.vue
(export interface FormData)      (import { FormData }, handleFormSubmit)
```

***

### `specialty?: string` — 선택적 필드(Optional)

`?`가 붙은 필드는 **있어도 되고 없어도 되는 필드**입니다. `Guide` 인터페이스를 다시 봅시다:

```typescript
export interface Guide {
  id?: number;          // ? 붙음: ID가 없어도 됨 (임시 데이터 등)
  name: string;         // ? 없음: 이름은 필수!
  specialty?: string;   // ? 붙음: 전문 분야 (생략 가능)
  tour_type?: string;   // ? 붙음: 투어 유형
  rating?: string;      // ? 붙음: 평점
  languages?: string;   // ? 붙음: 구사 언어
  description?: string; // ? 붙음: 상세 설명
}
```

```typescript
// 필수 필드 name만 있어도 유효한 Guide ✅
const simpleGuide: Guide = {
  name: "알리셰르"
  // description이나 specialty가 없어도 에러 없음!
};

// 모든 상세 정보가 다 채워진 Guide ✅
const fullGuide: Guide = {
  id: 1,
  name: "알리셰르 (Alisher)",
  specialty: "역사 및 고건축",
  tour_type: "history",
  rating: "4.9",
  languages: "한국어, 우즈베크어, 러시아어",
  description: "사마르칸트 국립대 역사학과 출신으로 레기스탄과 샤히진다의 숨겨진 역사를 깊이 있게 전달합니다."
};
```

## 8. 귀납적 이해: 인터페이스 확장(extends)

실제 프로젝트에서는 인터페이스를 상황에 맞게 _확&#xC7A5;_&#xD558;기도 합니다. AI가 생성한 코드에서 이런 패턴을 발견할 수 있습니다.

```typescript
// 기본 가이드 인터페이스
interface Guide {
  id?: number;
  name: string;
  specialty?: string;
  rating?: string;
}

// extends — 기존 가이드 설계도를 물려받고, AI 추천 분석 정보를 확장
interface RecommendedGuide extends Guide {
  // Guide의 모든 필드(id, name, specialty, rating)를 자동으로 포함하면서 추가:
  aiReason: string;    // Gemini AI가 이 여행자에게 맞춤 추천한 이유
  matchScore: number;  // 취향 일치도 점수 (예: 98%)
}
```

`RecommendedGuide`는 `Guide`의 모든 필드를 자동으로 포함하면서, 추가 필드를 더 가집니다.

언제 확장을 쓰나요?

* 기본 목록 화면: 간단한 `Guide`만 필요 (이름, 전문 분야, 평점)
* AI 추천 화면: `RecommendedGuide`가 필요 (AI 추천 사유, 매칭 점수)
* API 응답마다 돌아오는 필드가 다를 때

### 상속의 비유

가게 점원의 명함과 점장의 명함을 생각해봅시다.

* 점원 명함: 이름, 전화번호, 부서
* 점장 명함: 이름, 전화번호, 부서 **+ 직책, 결재 권한**

점장의 명함은 점원 명함의 내용을 그대로 가지면서 추가 내용이 더 있습니다. `extends`가 바로 이런 관계입니다.

```typescript
interface StaffCard {
  name: string;
  phone: string;
  department: string;
}

interface ManagerCard extends StaffCard {
  // name, phone, department 자동 포함
  title: string;
  approvalLevel: number;
}
```

***

## 9. TypeScript 에러와 대화하기

TypeScript를 도입하면 초반에 에디터에 빨간 밑줄이 가득 생깁니다. 당황하지 마세요. 이것은 TypeScript가 이미 존재하던 잠재적 버그를 전부 찾아낸 것입니다.

### 자주 만나는 에러 메시지

**에러 1: `string` 타입을 `number`에 넣으려 할 때**

```
Argument of type 'string' is not assignable to parameter of type 'number'
```

한국어 번역: "string 타입의 값을 number 타입의 매개변수에 전달할 수 없습니다"

```typescript
// 원인: HTML <input>이나 URL 파라미터에서 가져온 값은 항상 문자열!
// samarkand-local-mate에서 URL 파라미터의 가이드 ID("1")를 조회 함수에 넘길 때
const inputId = "1"; // string (문자열)

function findGuideById(id: number) {
  // id는 반드시 숫자여야 함
  return sampleGuides.find(g => g.id === id);
}

findGuideById(inputId);
//            ^^^^^^^
// 에러: string은 number에 할당할 수 없음

// 해결: Number()로 숫자로 변환 후 전달
findGuideById(Number(inputId)); // ✅ 정상 동작
```

**에러 2: 없을 수도 있는 값(`null`)을 그냥 쓸 때**

```
Object is possibly 'null'
```

`samarkand-local-mate`의 `src/App.vue`에 실제로 작성되어 있는 코드를 봅시다:

```typescript
// src/App.vue의 실제 코드 (결과 섹션으로 화면 스크롤 이동)
await nextTick();
const section = document.getElementById('resultSection');

// ❌ if 체크 없이 바로 쓰면 TypeScript 에러 발생!
// section.scrollIntoView({ behavior: 'smooth', block: 'center' });
// ^^^^^^^ 오류: 'section'은(는) 'null'일 수 있습니다 (Object is possibly 'null')

// ✅ 해결: null 체크로 안전망 확보 (App.vue에 실제 적용된 패턴)
if (section) {
  section.scrollIntoView({ behavior: 'smooth', block: 'center' }); // ✅ null이 아닐 때만 안전하게 실행
}
```

`document.getElementById`는 해당 ID의 HTML 요소를 찾지 못하면 `null`을 반환합니다. TypeScript는 "요소가 없을 수도 있는데 바로 함수를 호출하면 앱이 멈춘다"고 경고해주는 것입니다.

**에러 3: 인터페이스에 없는 필드를 쓸 때**

```
Property 'score' does not exist on type 'Guide'
```

한국어 번역: "'Guide' 타입에 'score' 속성이 없습니다"

```typescript
const guide: Guide = {
  name: "알리셰르 (Alisher)",
  rating: "4.9"
};

// Guide 인터페이스에는 score가 없고 rating이 있음
console.log(guide.score);  // ❌ 오류!
console.log(guide.rating); // ✅ 정상: "4.9"
```

이 에러는 오히려 좋은 것입니다. 오타를 쳤거나 잘못된 필드명을 썼다는 걸 즉시 알 수 있습니다.

### 에러를 AI에게 붙여넣기

에러 메시지가 낯설더라도, 그대로 AI에게 던지면 됩니다.

> 💬 "이 에러가 왜 났어? `Argument of type 'string' is not assignable to parameter of type 'number'` 이게 무슨 뜻인지 예시로 설명해주고 고쳐줘."

{% hint style="info" %}
**💡 빠른 수정 단축키**

Antigravity IDE에서 빨간 밑줄이 있는 줄에 커서를 놓고 `Cmd+.`(맥) 또는 `Ctrl+.`(윈도우)을 누르면 빠른 수정 메뉴가 나옵니다. 여기서 **AI Fix**를 선택하면 AI가 바로 수정안을 제시합니다.
{% endhint %}

## 10. 왜 AI와 TypeScript가 궁합이 좋은가?

TypeScript 인터페이스가 정의되어 있으면, AI가 코드를 훨씬 정확하게 만들어 줍니다.

```
❌ 타입 없이 프롬프팅:
   "추천 결과 화면 컴포넌트(ResultSection) 만들어줘"
   → AI가 가이드 객체에 어떤 필드가 있는지 추측해서 코드를 작성함
   → guide.score? guide.rating? guide.price? 뭐가 맞는지 몰라 엉뚱한 필드를 참조

✅ 타입 있는 상태에서 프롬프팅:
   "Guide 인터페이스 배열(guides: Guide[])을 props로 받는 ResultSection 컴포넌트 만들어줘"
   → AI가 Guide 설계도를 보고 guide.name, guide.specialty, guide.rating을 정확하게 화면에 바인딩
   → 필드명 오타나 누락 없는 무결점 코드 생성!
```

인터페이스는 AI와의 소통 채널이기도 합니다. 인터페이스가 잘 정의될수록 AI가 더 정확한 코드를 만들어 줍니다.

### 자동완성의 힘

타입이 있으면 에디터가 쓸 수 있는 것들을 미리 알려줍니다.

```typescript
const guide: Guide = { /* ... */ };

guide.  // ← 점을 찍으면 에디터가 name, specialty, tour_type, rating, languages, description 목록을 자동완성으로 제시
        // 오타 없이 정확한 필드명만 선택할 수 있음
```

이것이 TypeScript를 쓰는 또 다른 이유입니다. 실수를 줄이는 것과 동시에, 코드 작성 속도도 빨라집니다.

***

## 11. `tsconfig.json` — TypeScript 엄격함 조절하기

`samarkand-local-mate` 프로젝트 루트의 TypeScript 컴파일러 설정 파일 `tsconfig.json`을 열어봅니다.

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "paths": {
      "@/*": [
        "./src/*"
      ]
    },
    "noEmit": true
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue"
  ]
}
```

| 옵션 | 설정값 | 의미 |
| ---- | ------ | ---- |
| `"target": "ES2022"` | ES2022 | 최신 브라우저가 지원하는 모던 JavaScript 표준으로 변환 |
| `"moduleResolution": "bundler"` | bundler | Vite 같은 최신 번들러에 맞게 모듈을 해석 |
| `"paths": { "@/*": ["./src/*"] }` | — | `@/components/GuideModal.vue`처럼 깔끔한 절대 경로 별칭 제공 |
| `"include": [..., "src/**/*.vue"]` | — | 일반 `.ts` 파일뿐만 아니라 `.vue` 싱글 파일 컴포넌트 안의 `<script setup lang="ts">` 영역까지 완벽하게 타입 검사 |

`"paths"` 덕분에 상대 경로 `../../components/...` 대신 `@/components/...`로 간결하게 임포트할 수 있고, `"include"`에 `src/**/*.vue`가 들어있어 모든 Vue 파일에서 TypeScript의 강력한 안전망을 누릴 수 있습니다.

{% hint style="info" %}
**💡 더 엄격한 검사를 원한다면: `"strict": true`**

더 꼼꼼한 코드 검사를 원한다면 `compilerOptions`에 `"strict": true`를 추가할 수 있습니다.
* `null`이나 `undefined`일 수 있는 값을 그냥 쓰는 실수 방지
* 함수의 매개변수 타입 누락 방지
처음에 빨간 줄이 많이 생겨도 하나씩 AI에게 물어보며 고치다 보면, 버그 없는 튼튼한 앱을 완성할 수 있습니다.
{% endhint %}

***

이 장에서 배운 것들:

* **기본 타입** (`string`, `number`, `boolean`, `[]`): 변수 하나에 붙이는 가장 기본적인 타입 표기
* **유니온 타입** (`|`): 정해진 값 중 하나만 허용
* **객체**: 관련된 데이터를 `{}` 안에 묶은 것
* **인터페이스** (`interface`): 객체의 모양을 설계도로 미리 약속
* **선택적 필드** (`?`): 없어도 되는 필드
* **확장** (`extends`): 기존 인터페이스를 물려받아 확장
* **에러와 대화하기**: 빨간 줄 에러 메시지를 AI에게 던지는 습관

다음 장에서는 이 TypeScript 타입 안전성 위에, 빠른 초기 로딩과 SEO를 동시에 해결하는 **Vite-SSG**를 적용합니다.
