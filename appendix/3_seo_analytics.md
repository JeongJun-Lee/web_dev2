# 부록 C: AI 주도 SEO 최적화 — 구글 검색에 잘 걸리는 앱 만들기

## 왜 SEO가 비즈니스의 생명선인가?
아무리 잘 만든 앱이라도 구글 검색 결과 3페이지에 묻혀 있으면 존재하지 않는 것과 같습니다. "사마르칸트 여행 가이드"로 검색했을 때 우리 앱이 1페이지에 나와야 비용 없이 사용자를 유입시킬 수 있습니다. 이것이 SEO(Search Engine Optimization, 검색 엔진 최적화)입니다.

3장에서 도입한 Vite-SSG가 SEO와 강력하게 연결됩니다. SPA는 초기 HTML이 비어 있어 구글 봇이 내용을 읽지 못하지만, SSG는 미리 만들어진 HTML에 내용이 가득 차 있어 구글 봇이 바로 인덱싱할 수 있습니다.

## 1. 프롬프팅: 메타 태그와 구조화 데이터를 한 번에
> 💬 "우리 투어 서비스의 각 페이지별로 구글 검색에 잘 걸리게 Meta 태그, Open Graph 태그를 작성하고, 검색엔진이 읽기 쉬운 JSON-LD 구조화 데이터를 동적으로 생성하는 Vue 컴포넌트를 만들어줘. 각 가이드 상세 페이지는 `TourService` 스키마로 만들어."

AI가 생성하는 파일들:
- `src/components/SeoMeta.vue` — 재사용 가능한 SEO 메타 컴포넌트
- `src/composables/useSeo.ts` — SEO 데이터를 관리하는 컴포저블

## 2. 귀납적 코드 이해: 메타 태그 — 구글 봇에게 보내는 명함
AI가 생성한 `SeoMeta.vue` 컴포넌트를 열어봅니다.

```vue
<script setup lang="ts">
import { useHead } from '@vueuse/head'

const props = defineProps<{
  title: string
  description: string
  imageUrl?: string
  path: string
}>()

useHead({
  title: () => `${props.title} | Samarkand Tour`,
  meta: [
    // 구글 검색 결과에 표시되는 설명
    { name: 'description', content: () => props.description },

    // Open Graph — SNS 공유 시 미리보기 카드
    { property: 'og:title', content: () => props.title },
    { property: 'og:description', content: () => props.description },
    { property: 'og:image', content: () => props.imageUrl },
    { property: 'og:url', content: () => `https://samarkandtour.com${props.path}` },
    { property: 'og:type', content: 'website' },

    // Twitter 카드
    { name: 'twitter:card', content: 'summary_large_image' },
  ]
})
</script>

<template>
  <!-- 이 컴포넌트는 화면에 아무것도 그리지 않음. 오직 <head> 태그만 수정 -->
</template>
```

읽으면서 파악하는 것들:
- **`<title>` 태그**: 구글 검색 결과에서 파란색 링크로 표시되는 텍스트입니다. 60자 이내가 권장됩니다.
- **`description` 메타 태그**: 제목 아래 나오는 회색 설명 텍스트입니다. 클릭률(CTR)에 직접 영향을 줍니다.
- **Open Graph(og:)**: Facebook, KakaoTalk, Slack 등에서 링크를 공유할 때 나오는 미리보기 카드를 제어합니다.
- **`useHead`**: HTML의 `<head>` 태그를 Vue 컴포넌트에서 동적으로 수정하는 라이브러리입니다.

## 3. 귀납적 코드 이해: JSON-LD — 구글 봇의 언어로 말하기
단순한 텍스트 설명을 넘어, 구글이 우리 서비스를 **정형화된 데이터(Structured Data)**로 이해할 수 있게 해줍니다.

AI가 생성한 가이드 상세 페이지의 JSON-LD를 봅니다:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "TouristGuide",
  "name": "아흐마드 카림",
  "description": "10년 경력의 사마르칸트 전문 역사 가이드",
  "priceRange": "₩150,000~₩300,000/1일",
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 39.6547,
    "longitude": 66.9597
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.8",
    "reviewCount": "127"
  }
}
</script>
```

- **구조화 데이터의 효과**: 구글 검색 결과에서 별점(⭐⭐⭐⭐⭐)이 표시되는 **리치 스니펫(Rich Snippet)**이 나타납니다. 클릭률이 일반 결과보다 20~30% 높습니다.
- **`schema.org`**: 구글, MS, Yahoo 등이 공동으로 만든 구조화 데이터 표준입니다. AI에게 "우리 서비스에 맞는 schema.org 타입을 추천해줘"라고 물어보면 최적의 스키마를 골라줍니다.

## 4. 프롬프팅: 사이트맵과 robots.txt 자동 생성
> 💬 "vite-ssg로 빌드할 때 모든 페이지의 URL을 담은 sitemap.xml과 robots.txt 파일을 자동으로 생성해줘. 가이드 상세 페이지의 우선순위(priority)는 0.8로, 메인 페이지는 1.0으로 설정해."

```xml
<!-- 생성된 sitemap.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://samarkandtour.com/</loc>
    <priority>1.0</priority>
    <changefreq>weekly</changefreq>
  </url>
  <url>
    <loc>https://samarkandtour.com/guides/ahmad-karim</loc>
    <priority>0.8</priority>
    <changefreq>monthly</changefreq>
  </url>
</urlset>
```

사이트맵은 구글 봇에게 "우리 사이트의 지도"를 제공합니다. 이 파일을 Google Search Console에 제출하면, 구글이 우리 페이지를 더 빠르게 인덱싱합니다.

## 5. 귀납적 코드 이해: 크롤러의 시선으로 바라보기
구글 봇이 우리 사이트를 어떻게 보는지 직접 확인해봅니다.

> 💬 "Google Search Console의 URL 검사 도구에서 우리 사이트를 분석했더니 이런 경고가 나왔어: [경고 내용 붙여넣기]. 어떻게 수정해야 해?"

또는 Chrome 개발자 도구(F12)에서:
```
개발자 도구 → Lighthouse → SEO 탭 → 분석 실행
```

Lighthouse가 0~100점으로 SEO 점수를 매기고, 개선 방법을 구체적으로 알려줍니다. AI에게 Lighthouse 결과를 붙여넣으면 항목별 수정 코드를 바로 받을 수 있습니다.

## 6. 귀납적 이해: 핵심 웹 지표(Core Web Vitals)와 순위의 관계
> 💬 "구글이 검색 순위를 결정할 때 Core Web Vitals를 본다던데, 우리 앱의 LCP, FID, CLS 점수를 어떻게 개선할 수 있는지 알려줘."

AI가 설명하는 핵심 지표:
- **LCP (Largest Contentful Paint)**: 가장 큰 콘텐츠가 화면에 나타나는 데 걸리는 시간. 2.5초 이내가 목표.
- **FID (First Input Delay)**: 사용자가 처음 클릭했을 때 반응하는 데 걸리는 시간. 100ms 이내가 목표.
- **CLS (Cumulative Layout Shift)**: 이미지나 광고가 로딩되면서 레이아웃이 갑자기 밀리는 현상. 0.1 이내가 목표.

3장에서 도입한 Vite-SSG와 Cloudflare의 글로벌 CDN이 이 지표들을 자동으로 개선해준다는 사실을 연결하며, 기술적 선택이 비즈니스에 미치는 영향을 귀납적으로 이해합니다.
