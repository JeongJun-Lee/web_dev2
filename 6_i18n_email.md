# 제6장: 글로벌 진출 — vue-i18n 다국어 지원과 자동 이메일 발송

> 🎉 이 장의 작은 성공: 언어 전환 스위치를 누르면 앱 전체가 영어로 바뀌고, 예약 완료 시 확정 이메일이 날아온다!

## 1. 프롬프팅: 다국어 텍스트 추출과 번역을 한 방에
수많은 하드코딩된 텍스트를 일일이 찾아서 분리하는 노가다를 AI에게 전면 위임합니다. 텍스트를 손으로 찾아내고, 변수로 바꾸고, 번역 파일을 만드는 작업 모두를 한 번의 프롬프트로 끝냅니다.

> 💬 "프로젝트 내의 모든 하드코딩된 한글 텍스트를 찾아서 vue-i18n 용 JSON 파일로 뽑아내줘. 키 이름은 영어로 의미 있게 짓고, 영어와 우즈벡어 번역 JSON도 같이 만들어. 코드 내의 텍스트는 `$t()` 함수로 교체해."

AI가 생성하는 파일 구조:
```
src/
└── locales/
    ├── ko.json    ← 한국어 (원본)
    ├── en.json    ← 영어 번역
    └── uz.json    ← 우즈벡어 번역
```

`ko.json` 파일을 열어보면:
```json
{
  "nav": {
    "home": "홈",
    "guides": "가이드 목록",
    "booking": "예약하기"
  },
  "hero": {
    "title": "사마르칸트의 숨겨진 이야기를 함께 나누다",
    "subtitle": "현지 전문 가이드와 함께하는 맞춤형 투어"
  }
}
```

우즈벡어 번역이 제대로 됐는지 확인이 안 되면:
> 💬 "우즈벡어 번역이 맞는지 확인하고 싶어. 이 JSON 파일을 DeepL API로 검증하는 방법을 알려줘."

## 2. 귀납적 코드 이해: `$t` 함수의 비밀 파헤치기
AI가 싹 다 교체해놓은 Vue 템플릿 코드를 엽니다.

```vue
<!-- 변환 전 -->
<h1>사마르칸트의 숨겨진 이야기를 함께 나누다</h1>

<!-- 변환 후 -->
<h1>{{ $t('hero.title') }}</h1>
```

코드를 읽으며 파악하는 원리:
- **`$t('hero.title')`**: `$t`는 "translate"의 약자입니다. 현재 선택된 언어(`locale`)에 따라 `ko.json` 또는 `en.json`에서 `hero.title` 키에 해당하는 값을 꺼내옵니다.
- **`locale` 변경의 마법**: `const { locale } = useI18n()` 으로 가져온 `locale.value`를 `'en'`으로 바꾸면, Vue의 반응형 시스템이 `$t()`를 사용하는 **화면 전체를 자동으로 다시 그립니다**. 새로고침 없이 즉각 반영됩니다.

AI에게 물어보면 나오는 추가 기능:
> 💬 "사용자가 선택한 언어를 localStorage에 저장해서, 다음에 방문했을 때도 같은 언어로 시작하게 해줘."

## 3. 귀납적 코드 이해: 언어 전환 컴포넌트 해부
AI가 생성한 `LanguageSwitcher.vue` 컴포넌트를 분석합니다.

```vue
<script setup lang="ts">
import { useI18n } from 'vue-i18n'

const { locale } = useI18n()
const languages = [
  { code: 'ko', label: '한국어', flag: '🇰🇷' },
  { code: 'en', label: 'English', flag: '🇺🇸' },
  { code: 'uz', label: "O'zbekcha", flag: '🇺🇿' },
]
</script>

<template>
  <div class="language-switcher">
    <button
      v-for="lang in languages"
      :key="lang.code"
      @click="locale = lang.code"
      :class="{ active: locale === lang.code }"
    >
      {{ lang.flag }} {{ lang.label }}
    </button>
  </div>
</template>
```

- **`v-for`**: 배열을 반복해서 요소를 생성합니다. `languages` 배열의 3개 항목이 3개의 버튼으로 렌더링됩니다.
- **`:class="{ active: locale === lang.code }"`**: 현재 선택된 언어 버튼에만 `active` CSS 클래스를 붙입니다. 조건부 클래스 바인딩입니다.

## 4. 사전 준비: 트랜잭션 이메일 서비스 — Resend

이메일 발신 서비스를 고르기 전에 잠깐 정리가 필요합니다.

**왜 Cloudflare Email Routing을 쓰지 않는가?**

책의 초안에는 "Cloudflare Email Routing으로 이메일을 보낸다"고 적혀 있었는데, 이것은 사실과 다릅니다. **Cloudflare Email Routing은 수신(받기) 전용 서비스**입니다. 예를 들어 `info@내도메인.com`으로 오는 메일을 내 Gmail로 전달하는 역할을 합니다. 사용자에게 예약 확정 메일을 **보내는** 용도로는 사용할 수 없습니다.

트랜잭션 이메일(예: 예약 확인, 영수증, 인증 코드)을 코드로 발송하려면 별도의 이메일 발송 서비스가 필요합니다.

| 서비스 | 무료 한도 | 특징 |
|---|---|---|
| **Resend** | 100건/일, 3,000건/월 | API 하나로 즉시 연동, 개발자 친화적 |
| SendGrid | 100건/일 | 대형 서비스, 설정이 다소 복잡 |
| Mailgun | 100건/일 | 오래된 서비스, 무료 플랜이 제한적 |

이 책에서는 **Resend**를 사용합니다. API 키 하나면 바로 발송이 가능하고, Cloudflare Workers에서 `fetch`로 직접 호출할 수 있어 AI가 코드 전부를 대리 작성해줄 수 있습니다.

**Resend 사용을 위한 수동 설정 (1회)**

Resend는 가입 후 두 가지를 수동으로 설정해야 합니다. 이 단계는 Cloudflare 대시보드와 마찬가지로 웹 브라우저에서 직접 해야 하며, AI가 대신할 수 없습니다.

1. [resend.com](https://resend.com) 가입 후 **API 키 발급** (대시보드에서 클릭)
2. 발신 도메인 인증: "이 도메인에서 보내는 메일이 내 메일이 맞다"는 DNS 레코드를 추가

> 💬 "Resend로 이메일을 발송하기 위해 내 도메인을 어디에 어떻게 인증하면 되는지 단계별로 알려줘."

위 작업이 완료되면 Resend API 키를 Cloudflare 환경변수에 등록합니다:

> 💬 "Resend API 키를 Cloudflare Pages 환경변수와 로컬 `.dev.vars` 파일에 추가해줘."

## 4-2. 프롬프팅: 결제 완료 시 확인 이메일 발송
Resend 준비가 끝나면, 5장의 Stripe 웹훅에 이메일 발송 코드를 이어붙입니다.

> 💬 "Stripe 웹훅에서 결제 성공이 확인되면, Resend API를 사용해서 예약 확정 메일을 사용자에게 자동으로 보내줘. 메일 본문에는 예약 날짜, 가이드 이름, 결제 금액, 취소 정책이 들어가게 해. HTML 형식으로 예쁘게 만들어줘."

AI가 생성하는 코드의 핵심:
```typescript
// functions/api/webhooks/stripe.ts 내부에 추가
await fetch('https://api.resend.com/emails', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${env.RESEND_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    from: 'booking@내도메인.com',
    to: userEmail,
    subject: '예약이 확정되었습니다 ✅',
    html: emailHtml,  // AI가 만든 HTML 템플릿
  }),
})
```

Resend가 준비된 상태에서 이 코드는 AI가 완전히 생성해줍니다. 이메일 본문도 로케일에 따라 다르게 보내도록 추가 지시할 수 있습니다.

> 💬 "이메일도 사용자의 선호 언어에 맞춰서 한국어 또는 영어로 발송되게 수정해줘."



## 5. 귀납적 코드 이해: 이벤트 기반 아키텍처(Event-Driven Architecture)
5장과 6장을 합치면 다음과 같은 연쇄 흐름이 완성됩니다:

```
결제 완료
    ↓ (Stripe Webhook)
DB에 예약 저장
    ↓
이메일 발송 (사용자 언어에 맞춰)
    ↓
(선택) 가이드에게도 알림 발송
```

이 구조의 핵심:
- **하나의 이벤트(결제 완료)가 연쇄적으로 여러 행동을 트리거**합니다.
- 각 행동(DB 저장, 이메일 발송)은 독립적이어서, 이메일 발송이 실패해도 예약 저장은 이미 완료됩니다.
- 이것이 현실 세계의 상용 서비스에서 사용하는 **이벤트 기반 아키텍처(EDA)**의 기초 형태입니다.

> 💬 "이벤트 기반 아키텍처가 왜 좋은지, 그리고 우리 코드의 어떤 부분이 EDA 패턴인지 초보자가 이해할 수 있게 설명해줘."

## 6. 귀납적 이해: i18n의 숨겨진 도전 — 복수형과 날짜 형식
다국어 지원은 단순히 텍스트를 바꾸는 것 이상입니다.

> 💬 "한국어에서는 '1개의 투어', '2개의 투어'인데, 영어에서는 '1 tour', '2 tours'처럼 복수형이 달라. vue-i18n에서 복수형을 처리하는 방법을 알려줘."

AI가 `$tc()` 함수와 복수형 JSON 작성 방법을 알려주는데, 이 과정에서 **언어마다 문법 규칙이 다르다는 것**을 코드를 통해 자연스럽게 배우게 됩니다. 우즈벡어는 한국어처럼 복수형 변화가 없다는 사실도 덤으로 알게 됩니다.
