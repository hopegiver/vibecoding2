# Cloudflare Workers - Durable Objects 활용

Durable Objects를 활용한 상태 관리와 실시간 기능 구현 방법을 학습합니다.

## Durable Objects란?

Durable Objects는 전역적으로 고유하고 일관된 상태를 가진 객체입니다.

**사용 사례:**
- 실시간 협업 (채팅, 문서 편집)
- 카운터
- 세션 관리
- 게임 상태

## 기본 구현

### Counter 예제

```typescript
export class Counter {
    private state: DurableObjectState;
    private count: number = 0;

    constructor(state: DurableObjectState, env: Env) {
        this.state = state;
    }

    async fetch(request: Request) {
        const url = new URL(request.url);

        if (url.pathname === '/increment') {
            this.count++;
            await this.state.storage.put('count', this.count);
            return Response.json({ count: this.count });
        }

        if (url.pathname === '/get') {
            this.count = await this.state.storage.get('count') || 0;
            return Response.json({ count: this.count });
        }

        return new Response('Not Found', { status: 404 });
    }
}
```

### wrangler.toml 설정

```toml
[[durable_objects.bindings]]
name = "COUNTER"
class_name = "Counter"
script_name = "my-worker"
```

### Worker에서 사용

```typescript
export default {
    async fetch(request: Request, env: Env): Promise<Response> {
        const id = env.COUNTER.idFromName('global-counter');
        const stub = env.COUNTER.get(id);
        return stub.fetch(request);
    }
};

export { Counter };
```

## 실시간 채팅

```typescript
export class ChatRoom {
    private state: DurableObjectState;
    private sessions: Set<WebSocket> = new Set();

    constructor(state: DurableObjectState, env: Env) {
        this.state = state;
    }

    async fetch(request: Request) {
        if (request.headers.get('Upgrade') === 'websocket') {
            const pair = new WebSocketPair();
            await this.handleSession(pair[1]);
            return new Response(null, { status: 101, webSocket: pair[0] });
        }

        return new Response('Expected WebSocket', { status: 400 });
    }

    async handleSession(websocket: WebSocket) {
        this.sessions.add(websocket);

        websocket.addEventListener('message', (event) => {
            // 모든 클라이언트에게 브로드캐스트
            for (const session of this.sessions) {
                session.send(event.data);
            }
        });

        websocket.addEventListener('close', () => {
            this.sessions.delete(websocket);
        });

        websocket.accept();
    }
}
```

## 관련 문서

- [프로젝트 시작하기](workers-getting-started.md)
- [API 개발](workers-api.md)
