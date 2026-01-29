# Cloudflare Workers - 테스트 및 디버깅

Workers 테스트 작성과 디버깅 방법을 학습합니다.

## 단위 테스트

### Vitest 설정

```bash
npm install -D vitest @cloudflare/vitest-pool-workers
```

**vitest.config.ts:**

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
    test: {
        poolOptions: {
            workers: {
                wrangler: { configPath: './wrangler.toml' }
            }
        }
    }
});
```

### 테스트 작성

**src/index.test.ts:**

```typescript
import { env, createExecutionContext } from 'cloudflare:test';
import { describe, it, expect } from 'vitest';
import worker from './index';

describe('Worker', () => {
    it('responds with hello', async () => {
        const request = new Request('http://localhost/');
        const ctx = createExecutionContext();
        const response = await worker.fetch(request, env, ctx);

        expect(response.status).toBe(200);
        const text = await response.text();
        expect(text).toContain('Hello');
    });
});
```

### 테스트 실행

```bash
npm test
```

## 로컬 디버깅

### wrangler dev

```bash
# 로컬 서버 시작
npx wrangler dev

# 디버그 모드
npx wrangler dev --local --persist
```

### 로그 출력

```typescript
export default {
    async fetch(request: Request, env: Env): Promise<Response> {
        console.log('Request:', request.url);
        console.log('Method:', request.method);

        const response = new Response('OK');
        console.log('Response status:', response.status);

        return response;
    }
};
```

## 프로덕션 로그

### wrangler tail

```bash
# 실시간 로그 확인
npx wrangler tail

# 특정 Worker
npx wrangler tail my-worker

# 필터링
npx wrangler tail --status error
```

## 관련 문서

- [프로젝트 시작하기](workers-getting-started.md)
- [배포 및 모니터링](workers-deployment.md)
