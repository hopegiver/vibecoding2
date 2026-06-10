# Vue Zero - 배포

Vue Zero + Cloudflare Workers 풀스택 앱의 배포, 환경 설정, 모니터링 방법을 안내합니다.

## 배포

### Wrangler로 배포

```bash
# 프로덕션 배포
wrangler deploy

# 특정 환경
wrangler deploy --env production
```

### 버전 관리

```bash
# 배포 이력 확인
wrangler deployments list

# 롤백
wrangler rollback <deployment-id>
```

## 환경 설정

### wrangler.toml

```toml
name = "my-app"
main = "server/index.js"
compatibility_date = "2024-01-01"

[assets]
directory = "./app"
not_found_handling = "single-page-application"
run_worker_first = ["/api/*"]

[vars]
ENVIRONMENT = "development"

[env.production]
name = "my-app-prod"
vars = { ENVIRONMENT = "production" }
```

> `[assets]` 섹션이 vue-zero의 핵심 설정입니다. `app/` 폴더를 정적 파일로 서빙하고, `/api/*` 요청만 Worker가 처리합니다.

### 환경 변수

**로컬 개발:** `.dev.vars` 파일 (Wrangler 자동 로드)

```
JWT_SECRET=your-dev-secret-key
```

**프로덕션:** Wrangler secrets

```bash
# 시크릿 등록
wrangler secret put JWT_SECRET --env production

# JWT 시크릿 생성
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

### 바인딩 설정 (필요 시)

```toml
# D1 데이터베이스
[[d1_databases]]
binding = "DB"
database_name = "my-database"
database_id = "your-database-id"

# KV 네임스페이스
[[kv_namespaces]]
binding = "KV"
id = "your-kv-namespace-id"

# R2 버킷
[[r2_buckets]]
binding = "BUCKET"
bucket_name = "your-bucket"
```

## 커스텀 도메인

### wrangler.toml

```toml
[env.production]
route = { pattern = "app.example.com/*", zone_name = "example.com" }
```

### Cloudflare 대시보드

1. Workers → 프로젝트 선택
2. Triggers → Custom Domains
3. 도메인 추가

## 모니터링

### 실시간 로그

```bash
# 전체 로그
wrangler tail

# 에러만
wrangler tail --status error

# 특정 메서드
wrangler tail --method POST
```

### Cloudflare 대시보드

- 요청 수
- 에러율
- CPU 시간
- 대역폭

## CI/CD

### GitHub Actions

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm install
      - run: npm run test
      - run: wrangler deploy --env production
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

## 배포 전 체크리스트

- [ ] `npm run scan`으로 pages.json / components.json 최신화
- [ ] 로컬에서 `wrangler dev`로 동작 확인
- [ ] `.dev.vars`의 시크릿을 프로덕션에 등록 (`wrangler secret put`)
- [ ] `wrangler.toml`의 `name`, 바인딩 ID 프로덕션 값 확인
- [ ] 배포 실행 (`wrangler deploy`)
- [ ] 프로덕션 URL에서 동작 확인

## 문제 해결

### 배포 실패

**증상:** `Error: Authentication error`

**해결:**
- `wrangler login` 재실행
- Cloudflare 계정 권한 확인

### 정적 파일이 서빙되지 않음

**해결:**
- `wrangler.toml`의 `[assets]` → `directory` 경로 확인
- `not_found_handling = "single-page-application"` 설정 확인

### API 호출이 프론트엔드 HTML을 반환

**해결:**
- `wrangler.toml`의 `run_worker_first = ["/api/*"]` 설정 확인
- API 경로가 `/api/`로 시작하는지 확인

## 관련 문서

- [시작하기](workers-getting-started.md)
- [프로젝트 구조](workers-structure.md)

[← 목차로 돌아가기](../_sidebar.md)
