# Cloudflare Workers - 배포 및 모니터링

Workers 배포, 버전 관리, 모니터링 방법을 학습합니다.

## 배포

### Wrangler로 배포

```bash
# 프로덕션 배포
npx wrangler deploy

# 특정 환경
npx wrangler deploy --env production
```

### 버전 관리

```bash
# 현재 배포된 버전 확인
npx wrangler deployments list

# 특정 버전으로 롤백
npx wrangler rollback <deployment-id>
```

## 환경별 설정

### wrangler.toml

```toml
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

# 개발 환경 (기본)
[env.development]
vars = { ENVIRONMENT = "development" }

# 프로덕션 환경
[env.production]
vars = { ENVIRONMENT = "production" }
route = { pattern = "api.example.com/*", zone_name = "example.com" }
```

## 커스텀 도메인

### 라우트 설정

```toml
[[routes]]
pattern = "api.example.com/*"
zone_name = "example.com"
```

또는 대시보드에서:
1. Workers → 프로젝트 선택
2. Triggers → Custom Domains
3. 도메인 추가

## 모니터링

### Analytics

**Cloudflare 대시보드:**
- 요청 수
- 에러율
- CPU 시간
- 대역폭

### wrangler tail

```bash
# 실시간 로그
npx wrangler tail

# 필터링
npx wrangler tail --status error
npx wrangler tail --method POST
```

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
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
      - run: npx wrangler deploy
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

## 관련 문서

- [프로젝트 시작하기](workers-getting-started.md)
- [테스트 및 디버깅](workers-testing.md)
