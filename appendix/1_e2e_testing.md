# 부록 A: E2E 테스트 자동화 — 봇이 대신 앱을 눌러보게 하기

## 왜 E2E 테스트가 필요한가?
1권에서 만든 랜딩 페이지는 단순했습니다. 하지만 이제 우리 앱에는 로그인, 예약, 결제, 다국어 전환 등 수십 가지의 사용자 시나리오가 있습니다. 코드를 수정할 때마다 이 모든 것을 손으로 눌러보며 확인한다면 하루가 금방 지나갑니다.

E2E(End-to-End) 테스트는 **실제 브라우저를 로봇처럼 조종해서, 사용자가 하는 행동을 자동으로 반복 실행**하는 기법입니다. 한 번 테스트를 만들어두면, 코드가 바뀔 때마다 30초 만에 전체 기능 검증이 끝납니다.

## 1. Antigravity 자체 기능 테스트 vs E2E 자동화
"Antigravity IDE는 내장 브라우저로 AI가 알아서 클릭해 보고 기능 테스트도 해주지 않나요?" — 아주 예리한 질문입니다!

Antigravity IDE의 AI 에이전트는 코딩 도중에 "결제 버튼 눌러서 잘 되는지 확인해줘"라고 지시하면, 스스로 브라우저를 띄우고 클릭하며 **개발 단계의 1회성 기능 검증**을 훌륭하게 수행합니다.

그렇다면 왜 굳이 Playwright로 테스트 코드를 짜야 할까요?
- **AI의 자체 테스트**: 개발 중인 내 로컬 컴퓨터 안에서, AI가 '사람(QA)'을 대신해 즉석에서 임시로 눌러보는 용도입니다.
- **Playwright E2E 자동화**: 사람도 AI도 없는 텅 빈 서버 환경(부록 B의 GitHub Actions)에서, 코드가 배포되기 직전에 수십 가지 시나리오를 **정해진 규칙대로 완벽하게** 검증해내는 '영구적인 방어막'입니다.

즉, AI의 자체 테스트는 '빠른 개발'을 위한 것이고, E2E 테스트 코드는 시간이 지나도 앱이 망가지지 않게 하는 '안전한 운영'을 위한 것입니다.

## 2. 프롬프팅: Playwright 세팅과 핵심 시나리오 작성
AI에게 테스트 프레임워크 도입과 첫 테스트 시나리오 작성을 한 번에 위임합니다.

> 💬 "Playwright를 이 프로젝트에 세팅하고, 다음 세 가지 E2E 테스트 시나리오를 작성해줘:
> 1. 메인 페이지 진입 → 로그인 → 투어 날짜 선택 → 예약 버튼 클릭
> 2. 언어 전환 스위치를 누르면 텍스트가 영어로 바뀌는지 확인
> 3. 로그인 없이 예약 페이지 접근 시 로그인 페이지로 리다이렉트되는지 확인"

AI가 생성하는 파일 구조:
```
e2e/
├── booking.spec.ts       ← 예약 플로우 테스트
├── i18n.spec.ts          ← 다국어 전환 테스트
└── auth.spec.ts          ← 인증 가드 테스트
playwright.config.ts      ← Playwright 설정
```

## 3. 귀납적 코드 이해: 테스트 코드 해부하기
AI가 생성한 `e2e/booking.spec.ts`를 열어봅니다.

```typescript
import { test, expect } from '@playwright/test'

test('로그인 후 투어 예약 플로우', async ({ page }) => {
  // 1. 메인 페이지 이동
  await page.goto('http://localhost:5173')

  // 2. 로그인 버튼 클릭
  await page.click('button#login-btn')

  // 3. 이메일과 비밀번호 입력
  await page.fill('input[type="email"]', 'test@example.com')
  await page.fill('input[type="password"]', 'testpassword123')
  await page.click('button[type="submit"]')

  // 4. 로그인 완료 확인 (헤더에 이름이 뜨면 성공)
  await expect(page.locator('.user-name')).toBeVisible()

  // 5. 달력에서 날짜 선택
  await page.click('.calendar-day[data-date="2025-10-15"]')

  // 6. 예약 버튼 클릭
  await page.click('button#book-now')

  // 7. Stripe 결제 팝업이 떴는지 확인
  await expect(page.locator('.stripe-payment')).toBeVisible()
})
```

코드를 읽으면서 발견하는 것들:
- `page.click()`, `page.fill()`: 사람이 마우스로 클릭하고 키보드로 입력하는 동작을 코드로 표현한 것입니다.
- `await`: 각 행동이 완료될 때까지 기다렸다가 다음 행동을 합니다. 사람이 로딩을 기다리는 것과 동일합니다.
- `expect(...).toBeVisible()`: "이 요소가 화면에 보여야 한다"는 검증(Assertion)입니다. 보이지 않으면 테스트가 실패합니다.
- **로봇이 사람처럼 행동한다**: `page.click('button#login-btn')`은 사람이 "로그인 버튼을 클릭한다"는 행동과 완전히 동일합니다.

## 4. 프롬프팅 + 이해: 테스트 실행과 리포트 해석
```bash
npx playwright test          # 모든 테스트 실행
npx playwright test --ui     # UI 모드로 시각적으로 실행 확인
npx playwright show-report   # 결과 리포트 보기
```

> 💬 "테스트 중 하나가 실패했어. 에러 메시지가 `TimeoutError: page.click: Timeout 30000ms exceeded`인데, 왜 났는지 설명하고 고쳐줘."

AI의 분석: "30초 안에 클릭할 요소를 찾지 못했다는 뜻입니다. 두 가지 원인이 있는데, 첫째는 CSS 선택자가 잘못됐거나, 둘째는 페이지 로딩이 완료되기 전에 클릭을 시도했기 때문입니다."

이 디버깅 과정에서 **테스트 코드의 견고성(Flakiness)**을 높이는 방법을 배웁니다. 특정 텍스트나 ID에 의존하는 대신 `data-testid` 속성을 사용하는 방식:

> 💬 "테스트가 특정 텍스트나 클래스 이름에 의존하지 않도록, 주요 버튼과 입력 필드에 `data-testid` 속성을 추가하고 테스트도 그에 맞게 수정해줘."

## 5. 귀납적 이해: 테스트 피라미드 — 얼마나 많이 테스트해야 하나?
> 💬 "테스트에는 E2E 테스트 말고도 유닛 테스트, 통합 테스트가 있다던데, 우리 프로젝트에서는 어떤 비율로 작성하는 게 좋을지 추천해줘."

AI가 설명하는 테스트 피라미드:
- **유닛 테스트** (많이): 개별 함수, 컴포넌트의 작은 동작 검증
- **통합 테스트** (보통): 여러 모듈이 연결될 때 동작 검증
- **E2E 테스트** (적게, 핵심만): 실제 사용자 시나리오 검증

이 책에서는 E2E 테스트에 집중하지만, AI에게 "우리 앱에서 유닛 테스트가 가장 필요한 함수 3개를 골라서 테스트 코드 작성해줘"라고 추가 도전해볼 수 있습니다.
