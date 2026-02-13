# Cloudflare Pages - 빌드 및 배포

Cloudflare Pages 프로젝트의 배포, 빌드 설정, 커스텀 도메인 설정 방법을 학습합니다.

## Git 기반 배포

### GitHub 연결

1. **Cloudflare 대시보드 접속**
   - https://dash.cloudflare.com/

2. **Pages → Create a project**

3. **Connect to Git**
   - GitHub 계정 연결
   - 저장소 선택

4. **빌드 설정**
   ```
   Framework preset: None
   Build command: (비워둠)
   Build output directory: /
   Root directory: (비워둠)
   ```

5. **배포 시작**
   - "Save and Deploy" 클릭

### 자동 배포

**main 브랜치 푸시 시:**

```bash
git add .
git commit -m "Update homepage"
git push origin main
```

→ 자동으로 프로덕션 배포

**feature 브랜치 푸시 시:**

```bash
git checkout -b feature/new-page
git add .
git commit -m "Add new page"
git push origin feature/new-page
```

→ 자동으로 프리뷰 배포 (https://xxx.myapp.pages.dev)

## Wrangler CLI 배포

### Wrangler 설치

```bash
npm install -g wrangler

# 로그인
wrangler login
```

### 직접 배포

```bash
# 프로덕션 배포
wrangler pages deploy . --project-name=myapp

# 특정 브랜치로 배포
wrangler pages deploy . --project-name=myapp --branch=develop

# 커밋 메시지 추가
wrangler pages deploy . --project-name=myapp --commit-message="Deploy v1.2.0"
```

## 빌드 설정

### package.json 스크립트

```json
{
  "name": "myapp",
  "version": "1.0.0",
  "scripts": {
    "dev": "npx wrangler pages dev .",
    "build": "echo 'No build required'",
    "deploy": "wrangler pages deploy .",
    "deploy:prod": "wrangler pages deploy . --branch=main"
  },
  "devDependencies": {
    "wrangler": "^3.0.0"
  }
}
```

### 빌드가 필요한 경우 (번들링)

**Vite 사용 예시:**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "deploy": "npm run build && wrangler pages deploy dist"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "wrangler": "^3.0.0"
  }
}
```

**vite.config.js:**

```javascript
import { defineConfig } from 'vite';

export default defineConfig({
    build: {
        outDir: 'dist',
        emptyOutDir: true,
        rollupOptions: {
            input: {
                main: './index.html'
            }
        }
    },
    server: {
        port: 8080
    }
});
```

**Cloudflare Pages 설정:**
```
Build command: npm run build
Build output directory: dist
```

## 환경별 배포

### 프리뷰 배포 (Preview)

**자동 생성:**
- Pull Request 생성 시
- 브랜치 푸시 시

**URL 형식:**
```
https://<commit-hash>.<project-name>.pages.dev
https://feature-new-page.<project-name>.pages.dev
```

### 프로덕션 배포 (Production)

**자동 배포:**
- main 브랜치 푸시 시

**URL:**
```
https://<project-name>.pages.dev
```

## 커스텀 도메인 설정

### 도메인 추가

1. **Pages 프로젝트 → Custom domains**

2. **Add a custom domain**
   - 도메인 입력: `myapp.com`

3. **DNS 레코드 추가**

**CNAME 레코드:**
```
Type: CNAME
Name: @
Target: myapp.pages.dev
```

**www 서브도메인:**
```
Type: CNAME
Name: www
Target: myapp.pages.dev
```

### HTTPS 자동 설정

Cloudflare Pages는 자동으로 SSL 인증서를 발급합니다.

**확인:**
```
https://myapp.com  → ✅ 안전함
```

### 리다이렉트 설정

`_redirects` 파일:

```
# www → non-www
https://www.myapp.com/* https://myapp.com/:splat 301

# HTTP → HTTPS (자동)

# 오래된 경로 리다이렉트
/old-page /new-page 301
/blog/:year/:month/:slug /articles/:slug 301
```

## 배포 롤백

### Cloudflare 대시보드

1. **Pages 프로젝트 → Deployments**

2. **이전 배포 선택**

3. **Rollback to this deployment**

### Wrangler CLI

```bash
# 배포 목록 확인
wrangler pages deployment list --project-name=myapp

# 특정 배포로 롤백
wrangler pages deployment rollback <deployment-id> --project-name=myapp
```

## CI/CD 통합

### GitHub Actions

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to Cloudflare Pages

on:
  push:
    branches:
      - main
      - develop

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Deploy

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Deploy to Cloudflare Pages
        uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: myapp
          directory: .
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}
```

**Secrets 설정:**
1. GitHub 저장소 → Settings → Secrets
2. `CLOUDFLARE_API_TOKEN` 추가
3. `CLOUDFLARE_ACCOUNT_ID` 추가

## 배포 최적화

### 빌드 캐싱

`.github/workflows/deploy.yml`:

```yaml
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- name: Install dependencies
  run: npm ci  # npm install 대신 npm ci 사용 (빠름)
```

### 병렬 빌드

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run build

  deploy:
    needs: [test, build]  # 병렬 실행 후 배포
    runs-on: ubuntu-latest
    steps:
      - uses: cloudflare/pages-action@v1
        # ...
```

## 모니터링

### 배포 로그 확인

**Cloudflare 대시보드:**
1. Pages 프로젝트 → Deployments
2. 배포 선택
3. Deployment log 확인

**Wrangler CLI:**

```bash
# 최근 배포 목록
wrangler pages deployment list --project-name=myapp

# 배포 상세 정보
wrangler pages deployment get <deployment-id> --project-name=myapp
```

### Analytics

**Cloudflare 대시보드:**
1. Pages 프로젝트 → Analytics
2. 확인 가능 지표:
   - 요청 수
   - 대역폭
   - 고유 방문자
   - 응답 시간

## Claude Code와 함께 사용하기

### 배포 자동화

```
배포 자동화 설정을 해줘.

GitHub Actions 워크플로우 생성:
1. main 브랜치 푸시 시 자동 배포
2. PR 생성 시 프리뷰 배포
3. 테스트 실행 (npm test)
4. 빌드 (npm run build)
5. Cloudflare Pages 배포

.github/workflows/deploy.yml 생성.
```

### 롤백 요청

```
배포를 이전 버전으로 롤백해줘.

wrangler pages deployment list로 목록 확인.
가장 최근 안정 버전으로 롤백.
배포 후 URL 접속 확인.
```

### 커스텀 도메인 설정

```
커스텀 도메인을 설정해줘.

도메인: myapp.com
설정:
- CNAME 레코드 (@ → myapp.pages.dev)
- www 리다이렉트 (www.myapp.com → myapp.com)
- HTTPS 자동 설정

_redirects 파일 생성.
```

## 실전 프롬프트 예시

### 스테이징 환경 추가

```
스테이징 환경을 추가해줘.

브랜치: develop
URL: https://staging.myapp.com
환경 변수: 프로덕션과 별도 (API_URL 등)

wrangler.toml에 staging 환경 추가.
GitHub Actions에서 develop 브랜치 배포 추가.
```

### 배포 전 체크리스트

```
배포 전 체크리스트를 실행해줘.

확인 사항:
1. 테스트 통과 (npm test)
2. 빌드 성공 (npm run build)
3. 환경 변수 설정 확인
4. _headers, _redirects 검증
5. 로컬에서 프리뷰 (npm run preview)

문제 없으면 배포.
```

### 성능 최적화 후 배포

```
성능 최적화 후 배포해줘.

최적화:
1. 이미지 압축
2. CSS 미니파이
3. JavaScript 번들링
4. 캐싱 헤더 설정

lighthouse 점수 측정 후 배포.
```

## 체크리스트

배포 전 확인사항:

- [ ] 테스트가 통과하는가?
- [ ] 환경 변수가 설정되어 있는가?
- [ ] D1/KV/R2 바인딩이 올바른가?
- [ ] _headers, _redirects 파일이 있는가?
- [ ] 커스텀 도메인 DNS가 설정되어 있는가?
- [ ] HTTPS가 작동하는가?
- [ ] 프리뷰 배포를 확인했는가?
- [ ] 롤백 계획이 있는가?

## 자주 하는 실수

### 1. 빌드 출력 디렉토리 오류

```yaml
# ❌ 잘못된 설정
Build output directory: .

# ✅ 올바른 설정
Build output directory: dist  # Vite/Webpack 등 사용 시
```

### 2. 환경 변수 미설정

```javascript
// ❌ 배포 후 오류
const apiKey = env.API_KEY;  // undefined!

// ✅ Cloudflare 대시보드에서 환경 변수 설정
```

### 3. DNS 레코드 오류

```
# ❌ 잘못된 CNAME
Type: A
Name: @
Value: 104.21.x.x  # IP 주소

# ✅ 올바른 CNAME
Type: CNAME
Name: @
Target: myapp.pages.dev
```

## 관련 문서

- [프로젝트 시작하기](pages-getting-started.md)
- [환경 변수 및 설정](pages-environment.md)
