---
coverY: 0
layout:
  width: default
  cover:
    visible: true
    size: background
    mask: radial
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
---

# 바이브 코딩 심화편

## 프롤로그

### 1권에서 우리는 아이디어를 화면으로 만들었습니다

순수 HTML·CSS·자바스크립트로 사마르칸트 투어 가이드 매칭 서비스를 처음부터 만들었습니다.\
Google Stitch로 디자인을 그리고, AI에게 코드를 부탁하고, Cloudflare에 배포하는 과정을 경험했습니다.

하지만 그 서비스를 실제로 운영하려는 순간, 이런 질문이 생겼을 겁니다.

> "로그인은 어떻게 구현하지?"\
> "카드 결제를 받으려면 무엇이 필요하지?"\
> "코드가 점점 복잡해지는데, 관리할 수 있을까?"\
> "모바일에서도 앱처럼 쓸 수 있게 만들 수 있을까?"

### 2권은 그 질문들에 답합니다

이 책은 1권의 정적 웹페이지를 **실제 돈을 받을 수 있는 글로벌 서비스**로 고도화하는 여정입니다.

코드를 암기하지 않습니다.\
AI에게 지시하고 → 결과를 보며 원리를 이해하는 **귀납적 접근 방식**은 동일합니다.\
달라진 것은 다루는 기술의 깊이입니다.

Vue.js로 코드를 체계적으로 정리하고, TypeScript로 오류를 미리 잡고, JWT로 사용자를 인증하고, Stripe로 결제를 연동하고, 앱스토어에 출시합니다.

### 이 책에서 만들 결과물

1권에서 만든 사마르칸트 투어 가이드 서비스가 이 책을 마치면 이렇게 변합니다.

| 구분 영역        | 1권 결과물 (기초편)                | 🚀 2권 결과물 (심화편)                           |
| ------------ | --------------------------- | ----------------------------------------- |
| **웹 아키텍처**   | 프레임워크 없는 순수JS를 사용한  간단한 웹 앱 | **TypeScript + Vue.js 프레임워크를 사용한 모던 SPA** |
| **사용자 인증**   | 없음 (단순 모달 폼)                | **JWT + D1 보안 회원가입 · 로그인**                |
| **수익화 (결제)** | 없음                          | **Stripe 글로벌 카드 결제 연동**                   |
| **다국어 지원**   | 한국어 단일 언어                   | **vue-i18n (한국어 · 영어 · 러시아어 등)**          |
| **타깃 플랫폼**   | 일반 웹 브라우저                   | **iOS · Android 앱스토어 패키징 출시**             |

과정에서 다음을 경험합니다.

* **Antigravity IDE**에서 Vue.js 프로젝트를 세팅하고 AI와 함께 개발합니다.
* **TypeScript**로 타입을 정의하며 런타임 오류를 사전에 방지합니다.
* **JWT + Cloudflare D1**으로 안전한 사용자 인증 시스템을 구축합니다.
* **Stripe**로 카드 결제를 연동하고 웹훅(Webhook)으로 예약을 처리합니다.
* **vue-i18n**으로 한국어·영어·러시아어를 지원하는 글로벌 서비스를 만듭니다.
* **Capacitor**로 웹 앱을 iOS·Android 앱으로 변환하고 앱스토어에 출시합니다.

### 기술 스택

* **개발 환경**: Antigravity IDE
* **프론트엔드**: Vue.js 3, TypeScript, Vite + vite-ssg
* **백엔드 및 배포**: Cloudflare Pages Functions, D1 (SQLite)
* **비즈니스 기능**: Stripe (결제), vue-i18n (다국어), JWT (인증)
* **모바일**: PWA + Capacitor + 푸시 알림 (앱스토어 출시)

### 완벽하지 않아도 시작하세요

에러 메시지가 낯설어도 괜찮습니다.\
Stripe 문서가 영어라 막막해도 괜찮습니다.\
앱스토어 심사에서 리젝을 받아도 괜찮습니다.

빌더는 처음부터 완벽하지 않습니다.\
작게 만들고, AI에게 물어보고, 고칩니다.

이 책은 그 과정을 함께 걷습니다. 🚀

**변경내역:**

| 버전   | 날짜         | 내역        |
| ---- | ---------- | --------- |
| v0.1 | 2026/09/05 | 프리뷰 버전 공개 |
|      |            |           |

**저자: JJ \_with AI**\_\*\* ([**comseong@gmail.com**](https://app.gitbook.com/s/nDUP8xZ7pbezrK2wo5dX/#jj-comseong-gmail.com))\*\*

**저작권:** [**https://creativecommons.org/licenses/by-nc-sa/4.0/**](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.ko)**​**

### 여러분의 후원은 컨텐트 제작에 큰 힘이 됩니다!!​​ <mark style="color:$success;">후원해주시는 분들께 작은 선물로....</mark>

* <mark style="color:$success;">\[</mark><mark style="color:$success;">**혜택 1**</mark><mark style="color:$success;">]</mark> 해당 **책 내용 전체를 하나로 묶은 PDF 파일**을 드립니다. (_후원완료 후, **후원하신 날짜와 시각, 입금자명을 적어서** 위에 언급된 저저의 이메일로 메일 한 통 남겨주세요. 해당 메일의 회신으로 파일 보내드리겠습니다. 감사합니다.)_
* <mark style="color:$success;">\[</mark><mark style="color:$success;">**혜택 2**</mark><mark style="color:$success;">]</mark> 마스터편 출간 전 책을 가장 먼저 읽어보실 수 있게 **베타 리더**로 초대됩니다.

{% include "https://app.gitbook.com/s/6FcF5JzEabIky96TfV3Z/~/reusable/3pUVkHOkYOOZOHKLWWJ5/" %}
