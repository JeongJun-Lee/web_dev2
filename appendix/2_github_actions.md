# 부록 B: GitHub Actions CI/CD — 검증된 코드만 배포되는 파이프라인

## 왜 Cloudflare 자동 배포만으로는 부족한가?
1권에서 Cloudflare Pages와 GitHub 저장소를 연동했습니다. GitHub에 코드를 `push`하면 Cloudflare가 알아서 빌드하고 배포해줬습니다. 이것으로 충분하지 않을까요?

단순한 랜딩 페이지라면 충분합니다. 하지만 이제 우리 앱에는 결제, 인증, 예약 로직이 있습니다. **TypeScript 에러가 있는 코드나 테스트가 실패한 코드가 라이브 서버에 올라가면 실제 사용자의 결제가 실패합니다.** 이것이 GitHub Actions가 필요한 이유입니다.

```
[기존 방식] push → Cloudflare 자동 배포
[개선된 방식] push → TS 타입 검사 → E2E 테스트 → 모두 통과 시에만 배포
```

## 1. 프롬프팅: CI/CD 파이프라인 한 번에 구축하기
> 💬 "GitHub에 코드를 push하면, 1) TypeScript 에러를 검사하고 2) Playwright E2E 테스트를 돌린 뒤 3) 모두 통과했을 때만 Wrangler(Cloudflare CLI)를 통해 배포하는 GitHub Actions workflow 파일을 짜줘."

AI가 생성하는 파일:
```
.github/
└── workflows/
    └── deploy.yml     ← CI/CD 파이프라인 설정 파일
```

## 2. 귀납적 코드 이해: YAML 파일 한 줄씩 해부하기
AI가 생성한 `.github/workflows/deploy.yml` 파일을 펼쳐봅니다.

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]   # main 브랜치에 push될 때만 실행

jobs:
  # Job 1: 타입 검사
  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4          # 코드를 가져온다
      - uses: actions/setup-node@v4        # Node.js를 설치한다
        with:
          node-version: '20'
      - run: npm ci                        # 의존성 설치
      - run: npm run type-check            # TypeScript 에러 검사

  # Job 2: E2E 테스트 (타입 검사가 통과돼야 시작)
  e2e-test:
    runs-on: ubuntu-latest
    needs: type-check                      # type-check가 끝난 후 실행
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npx playwright install --with-deps  # 브라우저 설치
      - run: npm run build                       # 앱 빌드
      - run: npx playwright test                 # E2E 테스트 실행

  # Job 3: 배포 (E2E 테스트가 통과돼야 시작)
  deploy:
    runs-on: ubuntu-latest
    needs: e2e-test                        # e2e-test가 끝난 후 실행
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - name: Deploy to Cloudflare Pages
        uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: samarkand-tour
          directory: dist
```

읽으면서 파악하는 핵심 개념들:
- **`on: push: branches: [main]`**: 언제 이 파이프라인이 실행되는지 정의합니다. `main` 브랜치에 push할 때만 발동됩니다.
- **`jobs`**: 파이프라인이 수행할 작업 목록입니다. 각 Job은 독립된 가상 컴퓨터(ubuntu-latest)에서 실행됩니다.
- **`needs`**: Job 간의 의존 관계입니다. `deploy`는 `e2e-test`가 성공해야만 실행됩니다. 이 한 줄이 "검증된 코드만 배포된다"는 보증을 만들어냅니다.
- **`${{ secrets.CLOUDFLARE_API_TOKEN }}`**: 비밀 API 키를 코드에 직접 쓰지 않고, GitHub의 암호화된 비밀 저장소(Secrets)에서 가져오는 방식입니다.

## 3. 프롬프팅 + 이해: GitHub Secrets 설정하기
> 💬 "GitHub Actions에서 Cloudflare API 토큰을 안전하게 사용하려면 어떻게 설정해야 해? 단계별로 알려줘."

AI가 안내하는 설정 과정:
1. Cloudflare 대시보드 → API 토큰 생성
2. GitHub 저장소 → Settings → Secrets and Variables → Actions
3. `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID` 등록

**왜 Secrets을 쓰는가?**: API 토큰은 계정 전체에 접근할 수 있는 비밀번호와 같습니다. 코드에 직접 쓰면 GitHub 저장소가 공개될 때 토큰이 노출됩니다. Secrets에 저장하면 암호화되어 보관되고, GitHub Actions 실행 시 주입됩니다.

## 4. 귀납적 이해: PR(Pull Request)과 함께 쓰는 법
단순히 `main`에 push하지 않고, **Pull Request 방식**을 도입하면 더 안전해집니다.

> 💬 "main 브랜치에 직접 push하지 않고, feature 브랜치에서 PR을 올리면 테스트를 돌리고, PR이 머지될 때만 배포되도록 workflow를 수정해줘."

이 과정에서 소프트웨어 팀이 실제로 사용하는 **GitHub Flow** 브랜치 전략을 자연스럽게 배웁니다:
- `main` 브랜치 = 항상 배포 가능한 안정 버전
- `feature/기능명` 브랜치 = 개발 중인 기능
- PR → 리뷰 → 머지 → 자동 배포

## 5. 귀납적 이해: 실패한 파이프라인 읽기
의도적으로 TypeScript 에러를 하나 만들어서 push해봅니다.

> 💬 "GitHub Actions에서 type-check Job이 실패했어. 로그 내용이 이래: [로그 붙여넣기]. 어떻게 고치면 돼?"

GitHub Actions 로그의 빨간 ❌ 표시와 에러 메시지를 AI에게 전달하면, 정확히 어떤 파일 몇 번째 줄에서 문제가 생겼는지 분석해줍니다. **파이프라인이 실패한 덕분에 결함 있는 코드가 배포되지 않았다는 것**을 체감합니다.
