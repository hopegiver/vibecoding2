# Cloudflare Workers - 배포

Workers 배포, 환경 설정, 모니터링 방법을 안내합니다.

## 배포

### Wrangler로 배포

```bash
# 프로덕션 배포
npm run deploy

# 또는
npx wrangler deploy

# 특정 환경
npx wrangler deploy --env production
```

### 버전 관리

```bash
# 배포 이력 확인
npx wrangler deployments list

# 롤백
npx wrangler rollback <deployment-id>
```

## 환경 설정

### 환경 변수

**로컬 개발:** `.dev.vars` 파일 (Wrangler 자동 로드)

```
JWT_SECRET=your-dev-secret-key
OPENAI_API_KEY=sk-xxx
```

**프로덕션:** Wrangler secrets

```bash
# 시크릿 등록
wrangler secret put JWT_SECRET --env production

# JWT 시크릿 생성
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

### wrangler.toml 환경 분리

```toml
name = "workers-template"
main = "src/index.js"
compatibility_date = "2024-01-01"

[vars]
ENVIRONMENT = "development"

[env.dev]
name = "workers-template-dev"
vars = { ENVIRONMENT = "development" }

[env.production]
name = "workers-template-prod"
vars = { ENVIRONMENT = "production" }
```

### 바인딩 설정 (필요 시)

```toml
# KV 네임스페이스
[[kv_namespaces]]
binding = "KV"
id = "your-kv-namespace-id"
preview_id = "your-preview-kv-namespace-id"

# D1 데이터베이스
[[d1_databases]]
binding = "DB"
database_name = "your-database"
database_id = "your-database-id"

# R2 버킷
[[r2_buckets]]
binding = "BUCKET"
bucket_name = "your-bucket"
preview_bucket_name = "your-preview-bucket"
```

## 커스텀 도메인

### wrangler.toml

```toml
[env.production]
route = { pattern = "api.example.com/*", zone_name = "example.com" }
```

### 대시보드

1. Workers → 프로젝트 선택
2. Triggers → Custom Domains
3. 도메인 추가

## 모니터링

### 실시간 로그

```bash
# 전체 로그
npx wrangler tail

# 에러만
npx wrangler tail --status error

# 특정 메서드
npx wrangler tail --method POST
```

### Cloudflare 대시보드

- 요청 수
- 에러율
- CPU 시간
- 대역폭

## CI/CD

### GitHub Actions

```yaml
name: Deploy Worker

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
      - run: npx wrangler deploy --env production
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

## 관련 문서

- [시작하기](workers-getting-started.md)
- [라우팅](workers-routing.md)
