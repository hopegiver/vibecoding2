# 인프라 및 배포

바이브코딩에서 **인프라는 신경 쓰지 않는 것이 목표**입니다. Cloudflare 플랫폼을 사용하면 서버 관리, 배포, 확장을 모두 자동화할 수 있어 개발에만 집중할 수 있습니다.

## 기존 인프라 구성

전통적인 웹 서비스 운영에 필요한 인프라:

```
┌─────────┐    ┌──────────────┐    ┌──────────┐    ┌──────────┐
│   CDN   │ →  │ 로드밸런서    │ →  │ 웹서버    │ →  │ WAS 서버  │
└─────────┘    └──────────────┘    └──────────┘    └──────────┘
                                                        ↓
                                   ┌──────────┐    ┌──────────┐
                                   │스토리지서버│    │ DB 서버   │
                                   └──────────┘    └──────────┘
```

**6개의 서버/서비스를 각각 설정하고 관리해야 합니다:**

| 구성 요소 | AWS 서비스 예시 | 해야 할 일 |
|-----------|---------------|-----------|
| CDN | CloudFront | 캐시 정책, 도메인, SSL 인증서 설정 |
| 로드밸런서 | ALB/NLB | 타겟 그룹, 헬스체크, 라우팅 규칙 설정 |
| 웹서버 | EC2 + Nginx | OS 패치, Nginx 설정, 보안 그룹 관리 |
| WAS 서버 | EC2 + Node/Java | 런타임 설치, 프로세스 관리, 스케일링 설정 |
| DB 서버 | RDS | 인스턴스 타입 선택, 백업 정책, 복제 설정 |
| 스토리지 | S3 | 버킷 정책, CORS, 라이프사이클 규칙 |

**문제점:**
- **시스템 엔지니어(DevOps)가 별도로 필요** - 개발자와 다른 전문 영역
- **초기 구축에 1~2주 소요** - 개발 시작 전에 인프라부터 준비
- **확장 시 복잡도 급증** - 오토스케일링, 멀티 AZ, 컨테이너 오케스트레이션
- **각 서비스별 보안 설정** - 보안 그룹, IAM, VPC, SSL 인증서 개별 관리
- **장애 대응이 어려움** - 어느 레이어에서 문제인지 파악하는 것부터 난관

---

## Cloudflare 통합 플랫폼

Cloudflare는 위의 **6가지 역할을 하나의 플랫폼에서 통합** 제공합니다:

```
┌─────────────────────────────────────────────────────────┐
│                    Cloudflare 플랫폼                      │
│                                                          │
│   Pages (프론트엔드)          Workers (백엔드 API)         │
│   ├── 정적 호스팅 + CDN       ├── 서버리스 컴퓨팅           │
│   ├── 자동 SSL               ├── 글로벌 에지 실행          │
│   └── git push 배포          └── git push 배포            │
│                                                          │
│   D1 (SQL DB)    KV (키-값)    R2 (파일 저장소)            │
│   ├── SQLite     ├── 초저지연   ├── S3 호환 API            │
│   └── 서버리스    └── 캐시 최적  └── 이그레스 무료           │
│                                                          │
│   ✅ SSL/DDoS 자동 보호  ✅ 글로벌 CDN  ✅ 로드밸런싱 자동   │
└─────────────────────────────────────────────────────────┘
```

| 기존 구성 요소 | Cloudflare 대체 | 별도 설정 |
|--------------|----------------|----------|
| CDN | **자동 제공** (전 세계 300+개 PoP) | 없음 |
| 로드밸런서 | **자동 제공** (에지 네트워크) | 없음 |
| 웹서버 | **Pages** | 없음 (정적 파일 서빙) |
| WAS 서버 | **Workers** | 코드만 작성 |
| DB 서버 | **D1** / PlanetScale + Hyperdrive | 바인딩 설정만 |
| 스토리지 | **R2** | 바인딩 설정만 |

**시스템 엔지니어가 필요 없습니다.** 서버 프로비저닝, OS 패치, 스케일링 설정이 모두 자동입니다.

---

## 바이브코딩에 적합한 이유

Cloudflare Pages + Workers 조합이 바이브코딩에 특히 적합한 이유:

**1. 빌드/배포가 없는 프론트엔드**
- Pages는 정적 파일을 그대로 서빙합니다
- ViewLogic은 빌드 단계가 없으므로 HTML/JS를 그대로 배포
- Webpack, Vite 같은 빌드 도구 설정이 불필요

**2. AI가 작성한 코드를 바로 배포**
- 로컬에서 동작하면 프로덕션에서도 동일하게 동작
- 환경 차이로 인한 "내 PC에서는 되는데" 문제 없음

**3. 프론트/백엔드 독립 배포**
- Pages(프론트)와 Workers(백엔드)가 각각 독립 배포
- 프론트엔드만 수정해도 백엔드 재배포 불필요
- 바이브코딩 프로세스의 "프론트 먼저 → 백엔드 나중에" 순서와 일치

**4. 개발 환경 = 운영 환경**
- Wrangler CLI로 로컬에서 Workers, D1, KV, R2를 모두 에뮬레이션
- 개발 환경과 운영 환경이 거의 동일하므로 환경 차이 버그가 없음
- `wrangler dev`로 로컬 테스트, `wrangler deploy`로 운영 배포

---

## 배포

### Pages 배포 (프론트엔드)

GitHub/GitLab 저장소를 연결하면 **git push만으로 자동 배포**됩니다:

```bash
git add .
git commit -m "게시판 UI 수정"
git push origin main
# → Cloudflare Pages가 자동으로 감지하여 배포
# → https://myapp.pages.dev 에 반영 (약 30초)
```

- 브랜치별 프리뷰 배포 자동 생성 (PR 단위 확인 가능)
- 롤백은 Cloudflare 대시보드에서 이전 배포 선택만 하면 됨

### Workers 배포 (백엔드)

Workers도 Git 저장소를 연결하면 **git push만으로 자동 배포**됩니다:

```bash
git add .
git commit -m "게시판 API 추가"
git push origin main
# → Cloudflare Workers가 자동으로 감지하여 배포
```

Workers 생성 시 Git 저장소를 지정하면 Pages와 동일하게 push 기반 자동 배포가 설정됩니다. 별도의 CI/CD 파이프라인 구성이 필요 없습니다.

### 환경 분리

```toml
# wrangler.toml
[env.dev]
name = "myapp-api-dev"
vars = { ENVIRONMENT = "development" }

[env.production]
name = "myapp-api"
vars = { ENVIRONMENT = "production" }
```

```bash
# 개발 환경 배포
wrangler deploy --env dev

# 운영 환경 배포 (git push 시 자동)
wrangler deploy --env production
```

---

## 데이터베이스 선택

프로젝트 규모에 따라 적합한 데이터베이스를 선택합니다:

### 경량 데이터베이스: D1 + KV

소규모~중규모 프로젝트에 적합합니다:

| 서비스 | 용도 | 특징 |
|--------|------|------|
| **D1** | 관계형 데이터 (사용자, 게시글 등) | SQLite 기반, SQL 사용, 서버리스 |
| **KV** | 캐시, 설정값, 세션 | 키-값 저장소, 초저지연, 전역 분산 |

```javascript
// D1 사용 예시 (Workers 서비스 코드)
const users = await this.env.DB.prepare(
    'SELECT * FROM users WHERE role = ?'
).bind('admin').all();

// KV 사용 예시
await this.env.KV.put('config:site-name', '내 서비스', { expirationTtl: 3600 });
const siteName = await this.env.KV.get('config:site-name');
```

**D1이 적합한 경우:**
- 게시판, 회원관리, 간단한 SaaS
- 데이터 규모 수 GB 이내
- 단순한 관계형 쿼리

### 확장형 데이터베이스: PlanetScale + Hyperdrive

대규모 서비스나 복잡한 데이터 요구사항이 있는 경우:

| 서비스 | 역할 |
|--------|------|
| **PlanetScale** | 관리형 MySQL 또는 PostgreSQL 데이터베이스 |
| **Hyperdrive** | Workers와 외부 DB 간 연결 최적화 (캐싱, 커넥션 풀링) |

**PlanetScale이 적합한 경우:**
- SaaS 서비스, 대형 서비스
- 복잡한 트랜잭션, JOIN이 많은 쿼리
- 수평 확장(샤딩)이 필요한 규모
- 비용 대비 성능과 확장성이 우수

**PlanetScale 데이터베이스 옵션:**
- **MySQL (Vitess 기반)**: 수평 샤딩 지원, 대규모 트래픽에 강함
- **PostgreSQL**: 완전 관리형, 고가용성 (3개 AZ 자동 복제)

**Hyperdrive의 역할:**

```
Workers (에지) ──── Hyperdrive ──── PlanetScale (DB)
                     ├── 커넥션 풀링 (매 요청마다 새 연결 X)
                     ├── 쿼리 캐싱 (반복 쿼리 가속)
                     └── 보안 연결 (Workers ↔ DB 간 암호화)
```

- **보안**: Workers에서 외부 DB로의 연결을 안전하게 관리
- **성능**: 커넥션 풀링과 쿼리 캐싱으로 응답 속도 향상
- **개발 환경에서도 사용 가능**: 개발용 MySQL/PostgreSQL에도 Hyperdrive로 연결

```javascript
// Hyperdrive를 통한 PlanetScale MySQL 연결 (Workers 코드)
import mysql from 'mysql2/promise';

export default {
    async fetch(request, env) {
        const connection = await mysql.createConnection(env.HYPERDRIVE.connectionString);

        const [rows] = await connection.execute('SELECT * FROM users WHERE active = ?', [1]);
        return Response.json(rows);
    }
};
```

```toml
# wrangler.toml
[[hyperdrive]]
binding = "HYPERDRIVE"
id = "your-hyperdrive-config-id"
```

### 데이터베이스 선택 가이드

| 기준 | D1 | PlanetScale + Hyperdrive |
|------|-----|-------------------------|
| 프로젝트 규모 | 소~중규모 | 중~대규모, SaaS |
| 데이터 크기 | 수 GB 이내 | 수십 GB 이상 |
| 쿼리 복잡도 | 단순 CRUD | 복잡한 JOIN, 트랜잭션 |
| 확장성 | 제한적 | 수평 샤딩 (Vitess) |
| 비용 | 무료~매우 저렴 | 월 $5~ (규모에 따라) |
| 설정 난이도 | 바인딩만 추가 | Hyperdrive 설정 필요 |
| 추천 시나리오 | 게시판, 사내 도구 | SaaS, 커머스, 대형 서비스 |

---

## 파일 저장소: R2

이미지 업로드, 첨부파일 등 파일 저장에는 **R2**를 사용합니다:

```javascript
// 파일 업로드 (Workers)
await this.env.BUCKET.put(`uploads/${filename}`, fileBuffer, {
    httpMetadata: { contentType: 'image/png' }
});

// 파일 다운로드 URL 생성
const object = await this.env.BUCKET.get(`uploads/${filename}`);
```

**R2의 장점:**
- S3 호환 API (기존 S3 코드/도구 그대로 사용)
- **이그레스(전송) 비용 무료** - AWS S3 대비 가장 큰 차이점
- 10GB까지 무료

---

## 보안

Cloudflare 플랫폼은 **별도 설정 없이** 다음 보안 기능을 자동 제공합니다:

| 보안 기능 | 설명 | AWS 대비 |
|----------|------|---------|
| **SSL/TLS** | 모든 도메인에 자동 HTTPS 적용 | ACM 인증서 발급/갱신 불필요 |
| **DDoS 방어** | L3/L4/L7 DDoS 자동 차단 | AWS Shield 별도 구매 불필요 |
| **WAF** | 기본 웹 방화벽 규칙 적용 | AWS WAF 별도 설정 불필요 |
| **Bot 관리** | 악성 봇 자동 차단 | 별도 솔루션 필요 |

**시스템 엔지니어 없이** 엔터프라이즈급 보안이 기본 적용됩니다.

---

## 에지 컴퓨팅

Cloudflare Workers는 전 세계 **300개 이상의 PoP(Point of Presence)**에서 실행됩니다:

```
기존 (AWS Lambda):
  한국 사용자 → 서울 리전 (빠름)
  미국 사용자 → 서울 리전 (느림, ~200ms 네트워크 지연)

Cloudflare Workers:
  한국 사용자 → 서울 PoP (빠름)
  미국 사용자 → 미국 PoP (빠름)
  유럽 사용자 → 유럽 PoP (빠름)
```

- 리전 선택이 불필요합니다 (자동으로 가장 가까운 곳에서 실행)
- 글로벌 서비스를 별도의 멀티 리전 설정 없이 제공
- 콜드 스타트가 거의 없음 (Lambda 대비 큰 장점)

---

## 운영 비용 비교

### 시나리오: 중규모 SaaS 서비스

**조건:**
- 일 10만 사용자, 사용자당 50 API 요청
- 월 약 1.5억 API 요청
- 데이터베이스 10GB, 파일 저장소 50GB

### AWS 구성

```
CloudFront (CDN)           ~$50/월
ALB (로드밸런서)            ~$25/월
EC2 t3.medium x 2 (WAS)   ~$150/월
RDS db.t3.medium (MySQL)   ~$70/월
S3 50GB + 전송비용          ~$50/월
Route 53 (DNS)             ~$5/월
ACM (SSL)                  무료
기타 (CloudWatch, VPC 등)   ~$30/월
─────────────────────────────────
합계                       ~$380/월 이상
```

추가로 고려해야 할 사항:
- 시스템 엔지니어 인건비 (서버 관리, 장애 대응)
- 오토스케일링 설정 시 비용 증가
- 트래픽 급증 시 예측 불가능한 비용

### Cloudflare + PlanetScale 구성

```
Pages (프론트엔드)          무료
Workers Paid ($5 기본)      ~$47/월  (1.5억 요청)
D1 또는 PlanetScale         ~$15~39/월
R2 50GB                    ~$0.75/월
Hyperdrive                 Workers Paid에 포함
DNS + SSL + DDoS           무료
─────────────────────────────────
합계                       ~$65~90/월
```

| 항목 | AWS | Cloudflare |
|------|-----|-----------|
| 월 비용 | ~$380 이상 | ~$65~90 |
| 시스템 엔지니어 | 필요 | **불필요** |
| 스케일링 설정 | 수동 (ASG, ECS 등) | **자동** |
| 글로벌 배포 | 멀티 리전 추가 비용 | **기본 포함** |
| SSL/보안 | 개별 설정 | **자동 제공** |
| 예측 가능성 | 트래픽에 따라 변동 | **사용량 비례, 예측 가능** |

> **핵심:** 인프라 비용만 비교해도 약 1/4~1/6 수준이며, 시스템 엔지니어 인건비까지 포함하면 실질적으로 **1/10 이하**로 운영이 가능합니다.

---

## 무료 티어

Cloudflare의 무료 티어는 프로토타입이나 소규모 서비스에 충분합니다:

| 서비스 | 무료 범위 | 비고 |
|--------|----------|------|
| **Pages** | 무제한 | 사이트 수, 배포 횟수 무제한 |
| **Workers** | 일 10만 요청 | 소규모 서비스에 충분 |
| **D1** | 5GB 저장소 | 일 500만 읽기, 10만 쓰기 |
| **KV** | 1GB 저장소 | 일 10만 읽기 |
| **R2** | 10GB 저장소 | 월 1,000만 읽기 |

> **바이브코딩의 이점:** 프로토타입을 만들고 고객에게 보여주는 단계까지 **비용이 0원**입니다. 고객 확인 후 실제 서비스로 전환할 때만 유료 플랜($5/월~)으로 업그레이드하면 됩니다.

---

## 알아둘 제약사항

Cloudflare 플랫폼은 만능이 아닙니다. 다음 제약사항을 이해하고 선택하세요:

| 제약 | 내용 | 대응 방안 |
|------|------|----------|
| **Workers CPU 시간** | 무료 10ms, 유료 30초 | 무거운 연산은 외부 서비스 활용 |
| **D1 규모 한계** | 대용량/복잡한 쿼리에 부적합 | PlanetScale + Hyperdrive로 전환 |
| **WebSocket 제한** | Durable Objects 필요 (추가 비용) | 실시간 기능이 핵심이면 검토 필요 |
| **벤더 종속** | Cloudflare 전용 API 사용 | Workers 코드는 표준 Web API 기반이라 이식성 양호 |
| **복잡한 트랜잭션** | D1은 트랜잭션 지원 제한적 | 필요시 PlanetScale 사용 |

### Cloudflare가 적합하지 않은 경우

- 실시간 영상/음성 처리 (미디어 서버 필요)
- 대규모 배치 처리 (수 분 이상 소요되는 작업)
- GPU 연산이 필요한 ML/AI 추론
- 레거시 시스템과의 VPN 연동이 필수인 경우

이러한 경우에도 **프론트엔드는 Pages**, **일반 API는 Workers**에 두고, 특수한 요구사항만 AWS/GCP의 해당 서비스를 사용하는 **하이브리드 구성**이 가능합니다.

---

## 요약

| 항목 | 기존 (AWS 등) | Cloudflare |
|------|-------------|-----------|
| 서버 관리 | 직접 관리 (6종) | **자동 (관리할 것 없음)** |
| 시스템 엔지니어 | 필요 | **불필요** |
| 배포 | CI/CD 파이프라인 구축 | **git push** |
| 확장 | 수동 설정 (ASG, 컨테이너) | **자동 확장** |
| 보안 | SSL, WAF, DDoS 개별 설정 | **전부 자동** |
| 글로벌 서비스 | 멀티 리전 추가 구축 | **기본 제공 (300+ PoP)** |
| 개발↔운영 환경 | 차이 있음 (Docker 등으로 극복) | **거의 동일** |
| 비용 (중규모 SaaS) | ~$380/월 + 인건비 | **~$65~90/월** |
| 바이브코딩 적합성 | 인프라 준비에 시간 소요 | **코드 작성 즉시 배포 가능** |

---

## 관련 문서

- [개발 프로세스](dev-process.md) - 바이브코딩 10단계 개발 프로세스
- [Cloudflare Pages 시작하기](pages-getting-started.md) - 프론트엔드 프로젝트 생성
- [Cloudflare Workers 시작하기](workers-getting-started.md) - 백엔드 프로젝트 생성
- [Pages 배포](pages-deployment.md) - Pages 배포 상세
- [Workers 배포](workers-deployment.md) - Workers 배포 상세

---

[← 목차로 돌아가기](../_sidebar.md)
