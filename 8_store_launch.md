# 제8장: 앱스토어 배포 — 심사 제출부터 리젝 대응과 유지보수까지

> 🎉 이 장의 작은 성공: 앱스토어에 내 앱이 올라가고, 전 세계 누구나 다운로드할 수 있다!

## 1. 배포 전 체크리스트

앱스토어 심사를 제출하기 전에 AI에게 검토를 요청합니다.

> 💬 "iOS 앱스토어에 앱을 제출하기 전에 확인해야 할 체크리스트를 만들어줘. 우리 앱처럼 웹뷰 기반 앱이 자주 걸리는 심사 거절 사유도 포함해서."

AI가 만들어주는 체크리스트에는 다음과 같은 항목들이 포함됩니다:

- [ ] 앱 아이콘 1024x1024 PNG (투명도 없이)
- [ ] 스크린샷 (iPhone 6.7인치, 6.5인치, iPad 12.9인치)
- [ ] 개인정보 처리 방침 URL
- [ ] 앱 설명글 (4,000자 이내)
- [ ] 연령 등급 설정
- [ ] 테스트 계정 정보 (심사관이 로그인해볼 수 있게)
- [ ] 네트워크 없이도 앱이 최소한 뭔가를 보여주는가 (PWA 오프라인 지원)

## 2. 프롬프팅: 스토어 등록 서류 작업 위임하기

코딩이 아닌 '마케팅 서류 작업'도 AI에게 맡길 수 있습니다.

> 💬 "우리의 사마르칸트 투어 앱을 구글 플레이와 애플 앱스토어에 올릴 건데, 매력적인 앱 설명글(한국어/영어), 핵심 키워드 10개, 개인정보 처리 방침 문서를 작성해줘. 타겟 사용자는 중앙아시아 여행에 관심 있는 한국인이야."

AI가 생성하는 것들:

- **앱 설명글**: 첫 3줄이 검색 결과에 표시되는 "접는 부분" 이전 내용이므로 가장 강렬한 문장이 먼저 오도록 구성됩니다.
- **키워드 10개**: 앱스토어 검색 최적화(ASO)를 위한 키워드입니다. 경쟁 앱이 쓰지 않는 틈새 키워드를 AI가 찾아줍니다.
- **개인정보 처리 방침**: 앱스토어가 필수적으로 요구하는 문서입니다. 우리가 수집하는 데이터(이메일, 결제 정보)를 어떻게 처리하는지 명시합니다.

## 3. 프롬프팅 + 이해: 앱 빌드 파일 생성

실제 앱스토어에 올릴 수 있는 빌드 파일을 생성하는 과정입니다.

> 💬 "iOS 앱스토어에 제출할 .ipa 파일을 만드는 방법을 단계별로 알려줘. Apple Developer Program 등록은 이미 돼 있어."

AI가 안내하는 핵심 단계:

1. Xcode에서 `App Store Distribution` 빌드
2. Archive 생성
3. App Store Connect에 업로드
4. 앱 메타데이터 입력 후 심사 제출

Android는 더 간단합니다:

> 💬 "Android 앱의 서명된 AAB(Android App Bundle) 파일을 생성하는 방법을 알려줘."

## 4. 프롬프팅 + 실습: 리젝(Reject) 대응과 네이티브 기능 추가

앱스토어 심사는 깐깐합니다. 특히 웹뷰 기반 하이브리드 앱은 "웹사이트와 다를 바 없다"며 "최소 기능 미충족(Minimum Functionality)"으로 심사에서 반려되는 경우가 매우 흔합니다. 반려(Reject) 이메일을 받았다면 당황하지 않고 AI에게 전달하여 실질적인 앱의 기능을 보강하는 실습을 진행합니다.

> 💬 "애플 심사에서 'Guideline 4.2 — Minimum Functionality' 사유로 반려됐어. 메일 내용: [반려 메일 전체 붙여넣기]. 웹뷰 기반이라 네이티브 기능이 부족하다는 거 같은데, 푸시 알림(Push Notification) Capacitor 플러그인으로 추가해서 심사를 통과할 수 있게 코드를 수정해줘. 예약 결제 완료 알림은 이미 이메일로 처리하고 있으니, 푸시는 투어 24시간 전에 리마인더를 보내는 기능으로 넣어보자."

AI가 분석하는 Guideline 4.2의 핵심과 해결책:

- 앱이 단순히 웹사이트를 래핑한 것에 불과하다면 거절됩니다.
- 해결책: **Capacitor 플러그인으로 푸시 알림** 기능을 실질적으로 추가해야 합니다. 특히 푸시 알림은 웹 브라우저로는 안정적으로 제공할 수 없어 네이티브 앱 필요성을 심사에서 납득시키는 데 가장 직접적인 이유가 됩니다.

AI가 즉각적으로 다음과 같이 **투어 24시간 전 리마인더 푸시 알림** 코드를 작성해 줍니다.

```typescript
// AI가 생성한 푸시 알림 코드 (예약 DB와 연동)
import { PushNotifications } from "@capacitor/push-notifications";

// 앱 실행 시 권한 요청 + 토큰 등록
async function initPushNotifications() {
  // 1. iOS/Android에 알림 권한 요청
  const permission = await PushNotifications.requestPermissions();
  if (permission.receive !== "granted") return;

  // 2. 기기를 FCM(Firebase)/APNs에 등록 — 장치별 고유 Push Token 발급
  await PushNotifications.register();

  // 3. 발급된 Push Token을 서버(D1)에 저장
  PushNotifications.addListener("registration", async (token) => {
    await fetch("/api/users/push-token", {
      method: "POST",
      body: JSON.stringify({ token: token.value }),
    });
  });
}

// functions/api/bookings/remind.ts — 매일 자정 Cron으로 실행: 내일 투어가 있는 유저에게 리마인더 발송
export const onRequest: PagesFunction = async ({ env }) => {
  // D1에서 '내일' 투어 날짜인 예약 목록을 조회
  const tomorrow = new Date();
  tomorrow.setDate(tomorrow.getDate() + 1);
  const tomorrowStr = tomorrow.toISOString().split("T")[0]; // 'YYYY-MM-DD'

  const { results } = await env.DB.prepare(
    "SELECT u.push_token, b.tour_date FROM bookings b JOIN users u ON b.user_id = u.id WHERE b.tour_date = ? AND b.status = 'confirmed'",
  )
    .bind(tomorrowStr)
    .all();

  // 해당하는 유저 전원에게 FCM으로 리마인더 발송
  for (const booking of results) {
    await fetch(
      "https://fcm.googleapis.com/v1/projects/YOUR_PROJECT/messages:send",
      {
        method: "POST",
        headers: { Authorization: `Bearer ${env.FCM_TOKEN}` },
        body: JSON.stringify({
          message: {
            token: booking.push_token,
            notification: {
              title: "내일 투어가 있어요! 🗺️",
              body: `${booking.tour_date} 오전 10시, 레기스탄 광장 집합입니다. 준비되셨나요?`,
            },
          },
        }),
      },
    );
  }
  return new Response("OK");
};
```

- `@capacitor/push-notifications` 플러그인을 추가하면 iOS에서는 `APNs(Apple Push Notification service)`, Android에서는 `FCM(Firebase Cloud Messaging)`이라는 운영체제 본연의 푸시 인프라를 내부적으로 호출하게 됩니다.
- **D1과의 연동**: 5장에서 만들어두었던 `bookings` 테이블과 `users` 테이블을 JOIN하여, 내일 투어 날짜(`tour_date`)가 있는 유저만 골라내는 구조를 귀납적으로 파악하게 됩니다.
- **이메일과의 역할 분담**: 결제 직후 확정 안내는 이메일이, 당일 전날 "잊지 마세요" 리마인더는 푸시 알림이 담당합니다. 두 채널이 겹치지 않고 서로 다른 타이밍에 보완적으로 작동하는 설계입니다.

> 📖 **Cron이란?**
>
> **Cron**은 "정해진 시간에 자동으로 실행되는 예약 작업"을 의미합니다. 이름은 그리스어로 '시간'을 뜻하는 크로노스(Chronos)에서 유래했습니다.
>
> 쉽게 말해 **스마트폰의 알람 앱**과 같습니다. 알람은 내가 직접 버튼을 누르지 않아도 설정해둔 시각이 되면 자동으로 울립니다. Cron도 마찬가지로, "매일 밤 12시에 이 코드를 실행해"라고 한 번 설정해두면 서버가 알아서 반복 실행합니다.
>
> 전통적인 리눅스 서버에서는 `crontab`이라는 파일로 이 설정을 관리했는데, Cloudflare도 동일한 개념을 **Cron Triggers**라는 이름으로 제공합니다.

> ⚙️ **Cloudflare Cron Trigger 설정 방법**
>
> `remind.ts`는 누군가 URL을 호출해야 실행되는 일반 API가 아니라, **매일 자정 Cloudflare가 자동으로 실행**해주는 예약 작업입니다. 이 기능을 **Cron Triggers**라고 합니다.
>
> **1단계 — `wrangler.toml`에 스케줄 등록**
>
> ```toml
> # wrangler.toml
> [triggers]
> crons = ["0 13 * * *"]   # UTC 13:00 = 사마르칸트 저녁 6시(UTC+5)
> ```
>
> **2단계 — 함수에서 Cron 이벤트 처리**
>
> `functions/api/bookings/remind.ts`의 핸들러를 `onRequest` 대신 `scheduled`로 선언하면 Cron 이벤트를 받습니다.
>
> ```typescript
> export default {
>   async scheduled(event: ScheduledEvent, env: Env) {
>     // 위의 remind 로직 그대로
>   },
> };
> ```
>
> **3단계 — 대시보드에서 확인**
>
> Cloudflare 대시보드 → Workers & Pages → 프로젝트 선택 → **Settings → Triggers → Cron Triggers** 탭에서 등록된 스케줄과 마지막 실행 시각을 확인할 수 있습니다.
>
> - **과금**: Cloudflare Free 플랜도 하루 1회 Cron 실행은 무료 범위 내에 포함됩니다.

> 🌏 **"한국 자정이 맞나요?" — 타임존 설계 이야기**
>
> 전 세계에서 사용자가 온다면, "어느 나라 시간 기준으로 보내야 하나?"가 실제로 중요한 설계 질문입니다.
>
> **이 앱의 정답: 투어 현지(사마르칸트) 기준**
>
> 핵심은 "**투어가 어디서 열리는가**"입니다. 투어는 사마르칸트(UTC+5) 오전 10시에 시작합니다. 사용자의 국적이 한국이든 유럽이든, 투어 전날 저녁 현지 시각에 알림을 보내는 것이 가장 자연스럽습니다. 위 코드의 `UTC 13:00`은 바로 사마르칸트 현지 저녁 6시입니다.
>
> **더 정교한 접근: 사용자별 타임존 저장 (심화)**
>
> 사용자가 자국에서 투어 전날 오후 6시에 받길 원한다면, 회원가입 시 브라우저의 타임존 정보를 `users` 테이블에 저장하는 방법을 쓸 수 있습니다.
>
> ```typescript
> // 회원가입 시 브라우저 타임존 자동 감지 후 저장
> const timezone = Intl.DateTimeFormat().resolvedOptions().timeZone // 예: "Asia/Seoul"
> await fetch('/api/users/register', { body: JSON.stringify({ ..., timezone }) })
> ```
>
> 이렇게 저장된 `timezone` 값을 서버에서 읽어 각 사용자에게 적합한 현지 시각에 알림을 계산해 발송하는 것이 진정한 글로벌 앱의 설계입니다. 다만 이는 Cron을 여러 번 실행하거나 시간대별로 발송을 분류하는 추가 로직이 필요한 심화 과제입니다.

> 💡 **FCM(Firebase Cloud Messaging) 사전 준비 & 과금 방식**
>
> - **회원가입 및 계정**: Google 계정만 있으면 [Firebase 콘솔](https://console.firebase.google.com)에서 무료로 프로젝트를 생성할 수 있습니다.
> - **과금 체계**: FCM은 **100% 완전 무료**입니다. 푸시 알림 발송 건수 무제한이며, 추가 요금이 전혀 발생하지 않습니다.
> - **코드 설명**: `YOUR_PROJECT` 자리에는 본인의 Firebase 프로젝트 ID를 넣으며, `env.FCM_TOKEN`은 Firebase 콘솔에서 발급받은 서비스 계정 인증 토큰(OAuth 2.0 Access Token)을 Cloudflare Pages 환경변수에 등록하여 보안을 유지합니다.

반려 → AI에게 사유 분석 요청 → 푸시 알림 코드 추가 → 재제출이라는 **실무적인 디버깅 사이클**을 직접 체험합니다. 이 과정이 실제 모바일 앱 개발자들이 가장 빈번하게 겪는 일상입니다.

## 5. 귀납적 이해: 유지보수의 세계 엿보기

출시는 끝이 아니라 시작입니다. 앱이 스토어에 올라간 후의 유지보수 시나리오를 AI와 함께 예행연습합니다.

> 💬 "앱을 출시한 이후에 버그 리포트가 들어왔을 때, 어떤 프로세스로 수정하고 업데이트를 배포해야 하는지 알려줘. 웹 코드 수정 vs 네이티브 코드 수정에 따라 어떻게 다른지도 설명해줘."

AI가 리팩토링한 코드의 전후를 비교하며:

- **웹 코드 버그**: Cloudflare Pages에 재배포하면 앱스토어 심사 없이 즉시 반영됩니다 (웹뷰 특권!).
- **네이티브 코드 버그**: Capacitor 플러그인이나 앱 권한 설정 변경은 앱스토어 재심사가 필요합니다.
- **모듈(플러그인) 단위 구조**: AI가 기능을 추가할 때마다 스파게티 코드가 아닌 독립적인 모듈로 분리하는 패턴을 의식적으로 확인합니다.

## 6. 귀납적 이해: 앱스토어 리뷰 관리와 사용자 피드백

출시 후 사용자 리뷰가 쌓이기 시작합니다.

> 💬 "구글 플레이 스토어의 사용자 리뷰에 답변하는 데 도움을 줘. 이 리뷰에 어떻게 답변하는 게 좋을지 한국어와 영어 버전을 모두 작성해줘: [리뷰 내용 붙여넣기]"

사용자 리뷰 중 자주 등장하는 기능 요청을 정리해서 AI에게 주면, 우선순위와 개발 난이도를 분석해주고 다음 버전의 개발 방향을 제안받을 수 있습니다.

**축하합니다!** 바닐라 HTML 랜딩 페이지로 시작한 여정이, Vue.js + TypeScript + Stripe + PWA + 앱스토어 출시까지 완성되었습니다. AI와 함께라면 이 모든 것이 가능합니다.
