# 제7장: 모바일 앱으로의 전환 — PWA와 Capacitor 패키징

> 🎉 이 장의 작은 성공: 스마트폰에서 내 앱이 네이티브 앱처럼 실행된다!

## 1. PWA vs Capacitor — 뭘 써야 하나?
모바일 앱으로 만드는 방법에는 크게 두 가지가 있습니다. 둘의 차이를 먼저 파악합니다.

| | PWA | Capacitor (웹뷰) |
|---|---|---|
| 설치 방법 | 브라우저에서 '홈 화면에 추가' | 앱스토어에서 다운로드 |
| 앱스토어 등록 | 구글 플레이 가능 / 애플 앱스토어 불가 | 양대 스토어 모두 가능 |
| 업데이트 | 웹 배포만 하면 즉시 반영 | 앱스토어 심사 필요 |
| 디바이스 기능 | 제한적 (카메라 등 일부 불가) | 네이티브 플러그인으로 모두 가능 |
| 개발 난이도 | 쉬움 | 다소 복잡 |

이 책에서는 두 가지를 모두 구현합니다. PWA는 "앱 같은 경험"을, Capacitor는 "앱스토어 등록"을 위해 사용합니다.

## 2. 프롬프팅: PWA로 '앱 같은 느낌' 입히기
먼저 다운로드 없이도 스마트폰 홈 화면에 아이콘을 추가하고 오프라인에서도 작동하게 만드는 PWA를 적용합니다.

> 💬 "Vite-PWA 플러그인을 설치하고, 이 웹앱이 스마트폰 홈 화면에 아이콘으로 추가될 수 있게 manifest.json과 서비스 워커 등 필요한 설정을 전부 작성해줘. 앱 이름은 'Samarkand Tour', 테마 색상은 #1a1a2e로 해줘."

AI가 수행하는 작업:
1. `npm install vite-plugin-pwa` 설치
2. `vite.config.ts`에 플러그인 설정 추가
3. `public/` 폴더에 아이콘 파일 생성 (512x512, 192x192 등 여러 크기)

## 3. 귀납적 코드 이해: 매니페스트(Manifest) 파일의 정체
AI가 생성한 `manifest.json` 파일을 열어봅니다.

```json
{
  "name": "Samarkand Tour",
  "short_name": "SamTour",
  "description": "사마르칸트 현지 가이드 매칭 서비스",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#1a1a2e",
  "theme_color": "#1a1a2e",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

- **`display: "standalone"`**: 이 값이 핵심입니다. 브라우저 주소창과 탭바를 숨기고, 진짜 네이티브 앱처럼 전체화면으로 실행됩니다.
- **`start_url: "/"`**: 앱을 열었을 때 시작할 페이지입니다.
- **아이콘 크기**: 다양한 크기가 필요한 이유는 iOS의 홈 화면, Android의 앱 서랍, 스플래시 화면 등 각각 요구하는 해상도가 다르기 때문입니다.

### 서비스 워커(Service Worker) 이해하기
> 💬 "서비스 워커가 정확히 뭐하는 건지, 우리 앱에서는 어떤 파일들을 캐싱하고 있는지 설명해줘."

AI의 설명을 들으면서 파악할 것:
- 서비스 워커는 브라우저와 네트워크 사이에 앉아 있는 **중간 프록시**입니다.
- 한 번 방문한 페이지의 파일들을 캐시해두었다가, 오프라인 상태에서도 저장된 버전을 보여줍니다.
- `vite-plugin-pwa`가 이 복잡한 서비스 워커 코드를 자동으로 생성해줍니다.

## 4. 프롬프팅: Capacitor로 '진짜 앱'으로 변환하기
PWA만으로는 애플 앱스토어에 정식 등록이 어렵습니다. 웹 코드를 네이티브 앱 껍데기(웹뷰)에 담아주는 Capacitor를 도입합니다.

> 💬 "이 Vue 앱을 Capacitor를 사용해서 iOS와 Android 네이티브 앱으로 감싸줘. 빌드 후 로컬 에뮬레이터에서 실행할 수 있게 필요한 설정과 명령어를 순서대로 알려줘."

AI가 안내하는 순서:
```bash
# 1. Capacitor 설치
npm install @capacitor/core @capacitor/cli

# 2. 프로젝트 초기화
npx cap init "Samarkand Tour" "com.samarkandtour.app"

# 3. 웹 앱을 먼저 빌드
npm run build

# 4. iOS와 Android 플랫폼 추가
npx cap add ios
npx cap add android

# 5. 빌드 결과물을 네이티브 프로젝트에 복사
npx cap sync

# 6. 에뮬레이터에서 실행
npx cap run ios
npx cap run android
```

## 5. 귀납적 코드 이해: `ios`, `android` 폴더의 해부
Capacitor 세팅 후 프로젝트 루트에 생성된 `ios/`, `android/` 폴더를 들여다봅니다.

```
ios/
└── App/
    ├── App/
    │   └── public/     ← npm run build의 결과물(dist/)이 여기에 복사됨!
    └── App.xcodeproj   ← Xcode 프로젝트 파일
android/
└── app/
    ├── src/main/assets/public/  ← 동일하게 빌드 결과물이 복사됨
    └── build.gradle
```

- **핵심 발견**: 우리가 만든 웹 코드(HTML/JS/CSS)가 `public/` 폴더에 그대로 복사됩니다. 그리고 iOS/Android의 `WKWebView`/`WebView`가 이 파일들을 읽어서 실행합니다.
- **`capacitor.config.ts`**: 앱의 번들 ID(`com.samarkandtour.app`), 앱 이름, 웹뷰 설정 등이 정의됩니다. 앱스토어 등록 시 이 번들 ID가 앱을 고유하게 식별하는 이름표가 됩니다.
- **왜 이 방식이 강력한가**: 웹 코드 하나로 웹 + iOS + Android 세 플랫폼을 동시에 커버합니다. 유지보수도 웹 코드 하나만 수정하면 됩니다.

