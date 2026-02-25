# 바이브코딩 개발 프로세스

## 개요

바이브코딩은 단순히 AI로 코드를 작성하는 것이 아닙니다. **개발 순서 자체가 바뀝니다.** 기존의 "백엔드 먼저" 방식에서 "프론트엔드 먼저" 방식으로 전환하여, 고객과 함께 눈에 보이는 결과물을 먼저 만들고 그로부터 백엔드를 역설계합니다.

## 기존 개발 프로세스

전통적인 웹 개발 순서:

```
요구사항 분석 → DB 설계 → API 개발 → 프론트엔드 개발
```

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ 요구사항  │ →  │ DB 설계   │ →  │ API 개발  │ →  │ 프론트엔드│
│ 분석     │    │          │    │          │    │ 개발     │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
     1주             1주            2~3주           2~3주
```

### 기존 방식의 문제점

- **고객 피드백이 너무 늦음**: 화면이 나오기까지 4~6주 이상 소요
- **DB부터 설계하면 화면 요구사항과 괴리** 발생
- **API 스펙 변경이 비쌈**: DB 설계 후 API 개발까지 완료된 상태에서 화면 요구사항이 바뀌면 전부 수정
- **고객은 화면을 봐야 판단 가능**: 문서만으로는 정확한 요구사항을 전달하기 어려움
- **개발 후반에 대규모 수정** 발생 → 일정 지연

---

## 바이브코딩 개발 프로세스

```
요구사항 → 프론트엔드 → Mock API → 데이터 통합 → API 스펙 → 비즈니스 로직 → DB 설계 → 백엔드 → 연동 → 테스트
```

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ 1. 요구사항│ →  │ 2. 프론트 │ →  │ 3. Mock  │ →  │ 4. 데이터 │ →  │ 5. API   │
│    분석   │    │   엔드    │    │   API    │    │   통합   │    │   스펙   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
     1주          1~2주(AI)        0.5주           0.5주           0.5주
                     ↓
              고객/기획자 참여
              즉시 피드백

┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ 6. 비즈니스│ →  │ 7. DB    │ →  │ 8. 백엔드 │ →  │ 9. 실제  │ →  │10. 통합  │
│   로직    │    │   설계   │    │   개발    │    │   연동   │    │   테스트  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
     0.5주         0.5주          1~2주           0.5주           0.5주
```

---

## 단계별 상세

### 1단계: 요구사항 분석

기존과 동일하지만, **화면 중심으로 요구사항을 정리**합니다.

```
프롬프트 예시:
"게시판 시스템을 만들어야 해.
필요한 화면 목록:
- 게시글 목록 (페이징, 검색)
- 게시글 상세 (댓글 포함)
- 게시글 작성/수정
- 관리자 대시보드

각 화면에 필요한 데이터와 기능을 정리해줘."
```

**산출물:**
- 화면 목록 및 화면별 기능 정의
- 사용자 역할 정의
- 화면 흐름도 (어디에서 어디로 이동하는지)

> **주의:** 화면 중심으로 정리하되, "화면에 보이지 않는 요구사항"도 반드시 별도 항목으로 기록하세요. 인증/인가 정책, 알림 발송, 배치 처리, 외부 시스템 연동 같은 항목은 화면 목록만으로는 누락되기 쉽습니다.

### 2단계: 프론트엔드 개발 (고객 참여)

**바이브코딩의 핵심 단계입니다.** 고객이나 기획자와 함께 모든 화면을 Claude Code로 빠르게 개발합니다.

```
프롬프트 예시:
"게시판 목록 페이지를 만들어줘.

경로: src/views/board/list.html + src/logic/board/list.js
레이아웃: default

화면 구성:
- 상단: 검색바 (제목+내용, 카테고리 필터)
- 중앙: 게시글 테이블 (번호, 제목, 작성자, 날짜, 조회수)
- 하단: 페이지네이션 (10개씩)
- 우측 상단: 글쓰기 버튼

샘플 데이터 5건을 data()에 직접 넣어서 화면이 바로 보이도록 해줘."
```

**이 단계의 핵심:**

- **실제 동작하는 화면**을 빠르게 만들어 고객에게 보여줌
- 고객 피드백을 **즉시 반영** (바이브코딩으로 수분 내 수정)
- 데이터는 `data()` 안에 **하드코딩된 샘플 데이터** 사용
- 모든 화면이 완성될 때까지 반복

```javascript
// src/logic/board/list.js
export default {
    name: 'BoardList',
    layout: 'default',
    data() {
        return {
            keyword: '',
            category: 'all',
            posts: [
                { id: 5, title: '공지사항입니다', author: '관리자', date: '2025-01-20', views: 152, category: '공지' },
                { id: 4, title: '신규 기능 안내', author: '김개발', date: '2025-01-19', views: 89, category: '안내' },
                { id: 3, title: '점검 일정 공유', author: '이운영', date: '2025-01-18', views: 45, category: '공지' },
                { id: 2, title: '개발팀 회의록', author: '박팀장', date: '2025-01-17', views: 23, category: '일반' },
                { id: 1, title: '프로젝트 킥오프', author: '최대리', date: '2025-01-16', views: 67, category: '일반' }
            ],
            currentPage: 1,
            totalPages: 3
        }
    },
    methods: {
        search() {
            console.log('검색:', this.keyword, this.category);
        },
        goToDetail(id) {
            this.navigateTo('/board/detail', { id });
        },
        goToWrite() {
            this.navigateTo('/board/write');
        }
    }
}
```

**산출물:**
- 완성된 모든 화면 (ViewLogic 페이지)
- 고객 확인 완료된 UI/UX
- 각 화면의 샘플 데이터가 포함된 JS 파일

> **주의 1 - 샘플 데이터 설계를 신중하게:**
> 이 단계에서 넣는 샘플 데이터가 곧 DB 스키마의 기초가 됩니다. 대충 넣으면 나중에 테이블 설계가 비효율적이 됩니다.
> - 작성자를 `author: '관리자'`(문자열)로 넣을지, `userId: 1`(외래키)로 넣을지 의식하세요
> - 날짜 형식을 통일하세요 (`2025-01-20` vs `2025/01/20` vs 타임스탬프)
> - 상태값은 미리 정의하세요 (`status: '진행중'` → 어떤 상태값들이 존재하는지)

> **주의 2 - 고객 피드백 루프 관리:**
> 화면을 바로 고칠 수 있다는 장점이 오히려 "끝없는 수정 요청"으로 이어질 수 있습니다.
> - 화면 확정 기준을 사전에 합의하세요 (예: 2회 리뷰 후 확정)
> - 확정된 화면은 스크린샷 + 기능 명세로 기록해두세요
> - "나중에 바꿀 수 있다"고 해도, 확정 후 변경은 별도 요청으로 처리하세요

### 3단계: Mock API 추출

모든 화면이 확정되면, 각 페이지의 `data()`에 하드코딩된 샘플 데이터를 **JSON 파일로 분리**합니다.

```
프롬프트 예시:
"모든 페이지의 data()에 있는 샘플 데이터를 mock-api/ 폴더에
JSON 파일로 분리해줘.

규칙:
- 폴더 구조: mock-api/[기능]/[액션].json
- API 응답 형태로 래핑: { success: true, data: [...] }
- 페이지의 data()는 빈 배열로 변경
- mounted()에서 fetch로 JSON을 불러오도록 수정"
```

**변환 전:**
```javascript
// src/logic/board/list.js
data() {
    return {
        posts: [
            { id: 5, title: '공지사항입니다', ... },
            { id: 4, title: '신규 기능 안내', ... }
        ]
    }
}
```

**변환 후:**
```json
// mock-api/board/list.json
{
    "success": true,
    "data": [
        { "id": 5, "title": "공지사항입니다", "author": "관리자", "date": "2025-01-20", "views": 152, "category": "공지" },
        { "id": 4, "title": "신규 기능 안내", "author": "김개발", "date": "2025-01-19", "views": 89, "category": "안내" }
    ],
    "pagination": {
        "page": 1,
        "limit": 10,
        "total": 25,
        "totalPages": 3
    }
}
```

```javascript
// src/logic/board/list.js (수정 후)
data() {
    return {
        posts: [],
        currentPage: 1,
        totalPages: 1
    }
},
async mounted() {
    await this.loadPosts();
},
methods: {
    async loadPosts() {
        const response = await this.$api.get('/mock-api/board/list.json');
        this.posts = response.data;
        this.totalPages = response.pagination.totalPages;
    }
}
```

**Mock API 폴더 구조:**
```
mock-api/
├── board/
│   ├── list.json           # 목록 데이터
│   ├── detail-1.json       # 상세 데이터 (id=1)
│   ├── detail-2.json       # 상세 데이터 (id=2)
│   └── categories.json     # 카테고리 목록
├── user/
│   ├── profile.json        # 사용자 프로필
│   └── list.json           # 사용자 목록
└── dashboard/
    └── stats.json           # 대시보드 통계
```

**산출물:**
- `mock-api/` 폴더에 모든 JSON 데이터
- Mock API를 호출하도록 수정된 프론트엔드 코드
- 화면이 이전과 동일하게 동작하는지 확인

> **주의 - Mock의 한계를 인식하세요:**
> 정적 JSON 파일은 "항상 같은 데이터를 성공적으로 반환"합니다. 실제 API에서 발생하는 다음 상황은 Mock에서 테스트할 수 없습니다:
> - 빈 데이터 (`data: []`) 일 때 화면이 어떻게 보이는지
> - 에러 응답 (400, 401, 404, 500) 일 때 사용자에게 뭘 보여줄지
> - 데이터가 100건, 1000건일 때 페이징이 정상인지
> - POST/PUT/DELETE 후 목록이 갱신되는 흐름
>
> **대응:** Mock 단계에서 `mock-api/board/list-empty.json`, `mock-api/board/error-404.json` 같은 예외 케이스 JSON도 함께 만들어두면 연동 시 에러 UI 누락을 방지할 수 있습니다.

### 4단계: Mock 데이터 통합/정제

화면 단위로 만든 Mock JSON에는 **같은 엔티티가 여러 파일에 중복**되어 있습니다. DB 설계 전에 이를 엔티티 단위로 통합하고 정제합니다.

```
프롬프트 예시:
"mock-api/ 폴더의 모든 JSON 파일을 분석해서
중복되는 데이터를 엔티티 단위로 통합해줘.

작업:
1. 모든 JSON에서 등장하는 엔티티(사용자, 게시글, 카테고리 등) 식별
2. 각 엔티티의 전체 필드 목록 통합 (어떤 화면에서는 name만, 어떤 화면에서는 name+email)
3. 엔티티 간 관계 정리 (1:N, N:M)
4. mock-api/_entities/ 폴더에 통합된 마스터 데이터 생성
5. DATA-MODEL.md에 엔티티 관계도 문서 작성"
```

**왜 필요한가?**

화면별 JSON은 같은 데이터를 다른 형태로 표현합니다:

```javascript
// mock-api/board/list.json → 게시글 목록에서의 사용자
{ "author": "관리자" }                    // 이름만

// mock-api/board/detail-1.json → 게시글 상세에서의 사용자
{ "author": { "name": "관리자", "avatar": "/img/admin.png" } }  // 이름+아바타

// mock-api/user/list.json → 사용자 관리에서의 사용자
{ "id": 1, "name": "관리자", "email": "admin@company.com", "role": "admin" }  // 전체 정보
```

이 세 곳에서 표현된 "관리자"는 같은 엔티티입니다. 통합하지 않으면 DB 설계 시 **어떤 필드가 실제로 필요한지 파악하기 어렵습니다.**

**통합 결과 예시:**

```
mock-api/
├── _entities/                  # 통합된 마스터 데이터
│   ├── users.json             # 사용자 전체 (5명)
│   ├── posts.json             # 게시글 전체 (25건)
│   ├── categories.json        # 카테고리 전체 (4개)
│   └── comments.json          # 댓글 전체 (30건)
├── board/                      # 화면별 Mock (기존 유지)
│   ├── list.json
│   └── detail-1.json
└── user/
    └── list.json
```

```json
// mock-api/_entities/users.json (통합된 마스터 데이터)
[
    { "id": 1, "name": "관리자", "email": "admin@company.com", "role": "admin", "avatar": "/img/admin.png" },
    { "id": 2, "name": "김개발", "email": "kim@company.com", "role": "user", "avatar": null },
    { "id": 3, "name": "이운영", "email": "lee@company.com", "role": "user", "avatar": null },
    { "id": 4, "name": "박팀장", "email": "park@company.com", "role": "manager", "avatar": "/img/park.png" },
    { "id": 5, "name": "최대리", "email": "choi@company.com", "role": "user", "avatar": null }
]
```

**산출물 (DATA-MODEL.md) 예시:**

```markdown
## 엔티티 관계

### 식별된 엔티티
| 엔티티 | 출처 (Mock 파일) | 필드 수 |
|--------|-----------------|---------|
| User | board/list, board/detail, user/list, user/profile | 7개 |
| Post | board/list, board/detail | 9개 |
| Comment | board/detail | 5개 |
| Category | board/categories | 3개 |

### 관계
- User → Post: 1:N (작성자)
- User → Comment: 1:N (댓글 작성자)
- Category → Post: 1:N (게시글 분류)
- Post → Comment: 1:N (게시글의 댓글)

### 필드 통합 결과
#### User
| 필드 | 타입 | 출처 | 비고 |
|------|------|------|------|
| id | number | user/list | PK |
| name | string | 모든 곳 | 필수 |
| email | string | user/list, user/profile | 유니크 |
| role | string | user/list | admin, manager, user |
| avatar | string | board/detail | nullable |
| created_at | string | user/profile | |
| updated_at | string | user/profile | |
```

**산출물:**
- `mock-api/_entities/` 폴더에 통합된 마스터 데이터
- `DATA-MODEL.md` - 엔티티 목록, 필드 통합 결과, 관계도
- 각 화면별 Mock JSON은 그대로 유지 (프론트엔드 동작에 영향 없음)

### 5단계: API 스펙 정의

Mock API의 JSON 구조와 프론트엔드 로직을 분석하여 **실제 API 스펙 문서**를 작성합니다. 4단계의 통합된 데이터 모델을 기반으로, API 응답 형식도 정규화된 구조로 설계합니다.

```
프롬프트 예시:
"mock-api/ 폴더의 모든 JSON과
src/logic/ 폴더의 모든 JS 파일을 분석해서
API 스펙 문서를 작성해줘.

포함 항목:
- 엔드포인트 목록 (메서드, 경로)
- 요청 파라미터 (Query, Body)
- 응답 형식 (JSON 구조)
- 인증 필요 여부
- 에러 응답"
```

**산출물 예시 (API-SPEC.md):**
```markdown
## 게시판 API

### GET /api/board/posts
게시글 목록 조회

**Query Parameters:**
| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| page | number | N | 페이지 번호 (기본: 1) |
| limit | number | N | 페이지당 개수 (기본: 10) |
| keyword | string | N | 검색어 (제목+내용) |
| category | string | N | 카테고리 필터 |

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": 5,
            "title": "공지사항입니다",
            "author": "관리자",
            "date": "2025-01-20",
            "views": 152,
            "category": "공지"
        }
    ],
    "pagination": {
        "page": 1,
        "limit": 10,
        "total": 25,
        "totalPages": 3
    }
}
```

**인증:** 불필요

### POST /api/board/posts
게시글 작성 (인증 필요)
...
```

**산출물:**
- `API-SPEC.md` - 전체 API 스펙 문서
- 엔드포인트 목록, 요청/응답 형식, 인증 요구사항

> **주의 - 화면에 없는 API를 잊지 마세요:**
> Mock API는 "화면에서 호출하는 API"만 추출합니다. 하지만 실제 서비스에는 화면과 무관한 백엔드 전용 로직이 필요합니다:
> - **인증 API**: 로그인, 로그아웃, 토큰 갱신, 비밀번호 변경
> - **유효성 검증**: 이메일 중복 체크, 입력값 검증 규칙
> - **권한 체크**: 어떤 역할이 어떤 API를 호출할 수 있는지
> - **파일 업로드**: 화면에서는 `<input type="file">`이지만, 실제로는 R2 업로드 API 필요
> - **배치/스케줄**: 자동 알림 발송, 데이터 정리 등
>
> API 스펙 작성 시 반드시 "화면 기반 API"와 "시스템 API"를 구분하여 빠짐없이 정의하세요.

### 6단계: 비즈니스 로직 정의

API 스펙은 "어떤 데이터가 오가는가"를 정의합니다. 이 단계에서는 **"데이터가 어떤 조건에서 어떻게 변하는가"**를 정의합니다. 화면과 Mock API만으로는 드러나지 않는 **백엔드의 처리 규칙**을 문서화합니다.

```
프롬프트 예시:
"다음 자료를 분석해서 비즈니스 로직 문서를 작성해줘:
- 1단계 요구사항 분석 결과
- 모든 화면의 UI/UX (src/views/, src/logic/)
- API-SPEC.md
- DATA-MODEL.md

포함 항목:
1. 각 기능별 처리 규칙 (조건, 제약, 예외)
2. 상태 전이 다이어그램 (상태값이 있는 경우)
3. 권한 매트릭스 (누가 무엇을 할 수 있는지)
4. 단계별 프로세스 (승인 흐름, 결제 흐름 등)
5. 데이터 유효성 규칙

BUSINESS-LOGIC.md로 저장해줘."
```

**왜 필요한가?**

화면에서 "삭제" 버튼이 있으면 Mock에서는 단순히 데이터가 사라집니다. 하지만 실제로는:

```
❓ 화면만 봐서는 모르는 것들:
- 본인 글만 삭제 가능? 관리자도 삭제 가능?
- 댓글이 있는 글도 삭제 가능?
- 첨부파일이 있으면 R2에서도 삭제?
- 물리 삭제? soft delete (is_deleted 플래그)?
- 삭제 후 목록으로 이동? 이전 페이지 유지?
```

**산출물 예시 (BUSINESS-LOGIC.md):**

```markdown
## 게시판

### 게시글 작성
- **권한**: 로그인한 사용자만 가능
- **필수 입력**: 제목 (2~100자), 내용 (1자 이상), 카테고리
- **자동 설정**: 작성자(JWT에서 추출), 작성일시, 조회수 0
- **첨부파일**: 최대 3개, 파일당 10MB, 허용 확장자(jpg,png,pdf,docx)

### 게시글 수정
- **권한**: 본인만 수정 가능 (관리자 포함 불가)
- **제약**: 작성 후 30일 이내만 수정 가능
- **이력**: 수정 시 updated_at 갱신, 수정 횟수 기록

### 게시글 삭제
- **권한**: 본인 또는 관리자
- **방식**: soft delete (is_deleted = true, deleted_at 기록)
- **연쇄**: 댓글은 유지 (게시글 복구 시 함께 노출)
- **첨부파일**: 삭제 시 R2에서도 제거 (비동기)

### 상태 전이 (게시글에 상태가 있는 경우)
```
작성 → [제출] → 검토중 → [승인] → 게시됨
                       → [반려] → 반려됨 → [재작성] → 작성
게시됨 → [비공개] → 비공개 → [공개] → 게시됨
```

### 권한 매트릭스
| 기능 | 비로그인 | 일반 사용자 | 관리자 |
|------|---------|-----------|--------|
| 목록 조회 | ✅ | ✅ | ✅ |
| 상세 조회 | ✅ | ✅ | ✅ |
| 글 작성 | ❌ | ✅ | ✅ |
| 글 수정 | ❌ | 본인만 | 본인만 |
| 글 삭제 | ❌ | 본인만 | ✅ 전체 |
| 댓글 삭제 | ❌ | 본인만 | ✅ 전체 |
```

**산출물:**
- `BUSINESS-LOGIC.md` - 기능별 처리 규칙, 권한, 상태 전이, 유효성 규칙
- 이 문서가 백엔드 개발 시 서비스 레이어의 **구현 명세**가 됨

> **주의:** 이 단계의 문서는 개발자가 작성해야 합니다. AI가 화면에서 자동으로 추출할 수 있는 내용이 아닙니다. 고객/기획자와의 협의, 도메인 지식, 보안 정책 등을 반영해야 합니다. Claude Code에게는 "포맷 작성"을 맡기고, **규칙의 내용은 개발자가 직접 채우세요.**

### 7단계: DB 설계

API 스펙, 통합된 데이터 모델, 비즈니스 로직 문서를 기반으로 **데이터베이스 테이블을 설계**합니다.

```
프롬프트 예시:
"다음 문서들을 분석해서 D1 데이터베이스 스키마를 설계해줘:
- DATA-MODEL.md (엔티티, 관계, 필드)
- API-SPEC.md (API 응답 구조)
- BUSINESS-LOGIC.md (상태값, 권한, soft delete 여부)

규칙:
- CREATE TABLE 문으로 작성
- 외래키 관계 설정
- 인덱스 설정 (검색, 정렬에 사용되는 컬럼)
- 비즈니스 로직에서 필요한 컬럼 포함 (is_deleted, deleted_at 등)
- schema.sql 파일로 저장"
```

**산출물 (schema.sql):**
```sql
-- 사용자 테이블
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    name TEXT NOT NULL,
    role TEXT DEFAULT 'user',
    created_at TEXT DEFAULT (datetime('now')),
    updated_at TEXT DEFAULT (datetime('now'))
);

-- 게시글 테이블
CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    category TEXT DEFAULT '일반',
    user_id INTEGER NOT NULL,
    views INTEGER DEFAULT 0,
    created_at TEXT DEFAULT (datetime('now')),
    updated_at TEXT DEFAULT (datetime('now')),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- 인덱스
CREATE INDEX idx_posts_category ON posts(category);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_posts_user_id ON posts(user_id);
```

**이 순서의 장점:**
- JSON 데이터에 실제 필요한 필드가 이미 정의되어 있으므로 **불필요한 컬럼이 생기지 않음**
- API 응답 구조가 확정되어 있으므로 **JOIN, 인덱스 설계가 정확**
- 화면에서 실제로 사용하는 데이터만 테이블에 포함

> **주의 - JSON은 평면적, DB는 관계적:**
> Mock JSON은 보통 화면 표시에 최적화된 평면적(denormalized) 구조입니다. 이를 그대로 DB 테이블로 만들면 정규화가 빠집니다.
>
> ```javascript
> // Mock JSON (평면적)
> { "id": 1, "title": "글제목", "author": "김개발", "categoryName": "공지" }
>
> // ❌ 그대로 테이블로 만들면
> CREATE TABLE posts (id, title, author TEXT, category_name TEXT);
>
> // ✅ 정규화 필요
> CREATE TABLE posts (id, title, user_id INTEGER, category_id INTEGER);
> CREATE TABLE users (id, name);
> CREATE TABLE categories (id, name);
> ```
>
> JSON에서 DB 스키마를 추출할 때, 다대다 관계 테이블(태그-게시글, 역할-권한 등)은 화면에 직접 보이지 않으므로 의식적으로 설계해야 합니다.

### 8단계: 백엔드 개발

API 스펙, DB 스키마, 비즈니스 로직 문서, Mock JSON을 모두 갖춘 상태에서 **Workers API 서버를 개발**합니다.

```
프롬프트 예시:
"API-SPEC.md를 읽고 schema.sql의 테이블 구조를 참고해서
게시판 CRUD API를 Workers에 구현해줘.

참조 파일:
- API-SPEC.md (엔드포인트, 요청/응답 형식)
- BUSINESS-LOGIC.md (처리 규칙, 권한, 유효성)
- schema.sql (테이블 구조)
- mock-api/_entities/ (시드 데이터)

규칙:
- 서비스 클래스 패턴 사용
- D1 데이터베이스 사용
- 응답 형식은 mock-api JSON과 동일하게"
```

**핵심: API 응답 형식을 Mock JSON과 동일하게** 구현하면, 프론트엔드 코드 수정이 최소화됩니다.

**산출물:**
- `src/routes/board.js` - 라우트 핸들러
- `src/services/boardService.js` - 비즈니스 로직
- `schema.sql` 적용된 D1 데이터베이스
- Mock 데이터 시드 스크립트

> **주의 - 응답 형식 일치 여부를 꼼꼼히 확인하세요:**
> 실제 API 응답이 Mock JSON과 **필드명, 중첩 구조, 타입**까지 정확히 일치해야 프론트엔드 전환이 매끄럽습니다. 흔한 불일치 사례:
> - Mock에서는 `date: "2025-01-20"` → 실제 DB에서는 `created_at: "2025-01-20T09:30:00Z"` (필드명 + 형식 변경)
> - Mock에서는 `author: "관리자"` → 실제에서는 `user: { id: 1, name: "관리자" }` (구조 변경)
> - Mock에서는 숫자 `views: 152` → 실제에서는 문자열 `"152"` (타입 변경)
>
> **대응:** API 개발 완료 후 Mock JSON과 실제 응답을 diff 비교하는 검증 단계를 추가하세요.

### 9단계: 실제 API 연동

프론트엔드의 Mock API 경로를 **실제 API 엔드포인트로 교체**합니다.

```
프롬프트 예시:
"프론트엔드의 모든 mock-api 호출을
실제 Workers API 엔드포인트로 변경해줘.

변경 규칙:
- /mock-api/board/list.json → /api/board/posts
- /mock-api/board/detail-{id}.json → /api/board/posts/{id}
- POST/PUT/DELETE는 실제 API 호출로 변경
- 에러 처리 추가"
```

**변경 전:**
```javascript
async loadPosts() {
    const response = await this.$api.get('/mock-api/board/list.json');
    this.posts = response.data;
}
```

**변경 후:**
```javascript
async loadPosts() {
    const response = await this.$api.get('/api/board/posts', {
        params: {
            page: this.currentPage,
            keyword: this.keyword,
            category: this.category
        }
    });
    this.posts = response.data;
    this.totalPages = response.pagination.totalPages;
}
```

> **주의 - Mock에는 없었던 것들을 보강하세요:**
> Mock 단계에서는 "항상 성공"이었습니다. 실제 API 전환 시 다음을 반드시 추가하세요:
> - **로딩 상태 UI**: API 호출 중 스피너 또는 스켈레톤 표시
> - **에러 처리 UI**: 네트워크 오류, 서버 오류 시 사용자에게 보여줄 메시지
> - **인증 만료 처리**: 401 응답 시 로그인 페이지로 이동
> - **빈 데이터 상태**: 검색 결과 0건, 목록이 비어있을 때의 화면
> - **재시도 로직**: 일시적 오류 시 재시도 버튼 또는 자동 재시도
>
> ```javascript
> // ❌ Mock 시절 코드
> async mounted() {
>     const response = await this.$api.get('/api/board/posts');
>     this.posts = response.data;
> }
>
> // ✅ 실제 연동 시 보강
> async mounted() {
>     this.loading = true;
>     try {
>         const response = await this.$api.get('/api/board/posts');
>         this.posts = response.data;
>     } catch (error) {
>         if (error.response?.status === 401) {
>             this.navigateTo('/login');
>         } else {
>             this.errorMessage = '데이터를 불러오지 못했습니다.';
>         }
>     } finally {
>         this.loading = false;
>     }
> }
> ```

### 10단계: 통합 테스트

프론트엔드와 백엔드가 실제로 잘 연동되는지 **전체 흐름을 테스트**합니다.

```
프롬프트 예시:
"전체 게시판 기능을 테스트해줘.

테스트 시나리오:
1. 목록 조회 → 페이징 → 검색
2. 글 작성 → 목록에 반영 확인
3. 글 수정 → 상세에 반영 확인
4. 글 삭제 → 목록에서 제거 확인
5. 비로그인 시 작성/수정/삭제 차단 확인"
```

---

## 이 프로세스가 적합한 프로젝트 / 적합하지 않은 프로젝트

### 적합한 경우

- **CRUD 중심 웹 애플리케이션**: 게시판, 관리자 페이지, 대시보드, 회원관리 등
- **고객사 프로젝트**: 고객 피드백을 빨리 받아야 하는 SI/SM 프로젝트
- **프로토타이핑**: MVP를 빠르게 만들어 시장 반응을 확인해야 하는 경우
- **화면 비중이 큰 프로젝트**: 화면 수가 많고 백엔드 로직이 비교적 단순한 경우

### 적합하지 않은 경우 (보완 필요)

- **복잡한 비즈니스 로직 중심**: 결제/정산, 복잡한 상태머신, 워크플로우 엔진 등은 화면으로 드러나지 않으므로 별도 설계 단계가 필요합니다
- **데이터 모델이 핵심인 프로젝트**: 데이터 간 관계가 복잡하고 정규화가 중요한 경우, 화면보다 데이터 모델을 먼저 설계하는 것이 맞을 수 있습니다
- **외부 시스템 연동이 많은 경우**: 결제 PG, 외부 API, 메시지 큐 등은 Mock JSON으로 시뮬레이션하기 어렵습니다
- **실시간 처리**: WebSocket, SSE 기반 실시간 기능은 정적 Mock으로 테스트할 수 없습니다

> **보완 방법:** 위 경우에는 2단계(프론트엔드)와 병행하여 **별도의 백엔드 설계 문서**를 작성하세요. 화면에 안 보이는 로직은 프론트엔드 먼저 접근법으로 커버되지 않습니다.

---

## 프로세스 비교

| 항목 | 기존 프로세스 | 바이브코딩 프로세스 |
|------|-------------|-------------------|
| 첫 화면 확인 | 4~6주 후 | **1~2주 내** |
| 고객 참여 시점 | 개발 완료 후 | **프론트엔드 개발 중** |
| 요구사항 변경 비용 | 높음 (DB, API, 화면 모두 수정) | **낮음 (화면 단계에서 확정)** |
| DB 설계 정확도 | 추측 기반 | **실제 데이터 기반** |
| API 스펙 정확도 | 문서 기반 추측 | **실제 화면 데이터에서 추출** |
| 프론트-백엔드 괴리 | 자주 발생 | **Mock → 실제 전환으로 최소화** |
| AI 활용 효과 | 부분적 | **전 단계 활용** |

## 단계별 산출물 요약

| 단계 | 산출물 | 비고 |
|------|--------|------|
| 1. 요구사항 분석 | 화면 목록, 기능 정의서 | 화면 중심으로 정리 |
| 2. 프론트엔드 개발 | 모든 화면 (ViewLogic 페이지) | 고객 확인 완료 |
| 3. Mock API 추출 | `mock-api/` JSON 파일들 | 화면별 데이터 |
| 4. 데이터 통합/정제 | `_entities/`, `DATA-MODEL.md` | 엔티티 통합, 관계 정의 |
| 5. API 스펙 정의 | `API-SPEC.md` | 엔드포인트, 요청/응답 형식 |
| 6. 비즈니스 로직 정의 | `BUSINESS-LOGIC.md` | 처리 규칙, 권한, 상태 전이 |
| 7. DB 설계 | `schema.sql` | 테이블, 인덱스, 관계 |
| 8. 백엔드 개발 | Workers API 서버 코드 | 라우트, 서비스, 미들웨어 |
| 9. 실제 연동 | 수정된 프론트엔드 코드 | Mock → 실제 API 전환 |
| 10. 통합 테스트 | 테스트 결과 보고서 | 전체 흐름 검증 |

## 실전 타임라인 예시

**게시판 + 회원관리 시스템 (중규모 프로젝트)**

| 단계 | 기존 방식 | 바이브코딩 |
|------|----------|-----------|
| 요구사항 분석 | 1주 | 1주 |
| DB 설계 / 프론트엔드 | 1주 | 1~2주 (프론트 먼저) |
| API 개발 / Mock 추출 + 데이터 통합 | 2~3주 | 0.5~1주 |
| 프론트엔드 / API 스펙 + 비즈니스 로직 | 2~3주 | 0.5~1주 |
| - / DB 설계 | - | 0.5주 |
| - / 백엔드 개발 | - | 1~2주 |
| 통합 테스트 | 1주 | 0.5~1주 |
| **합계** | **7~9주** | **5~7주** |
| **고객 첫 확인** | **6주차** | **2주차** |

---

## 다음 단계

- [빠른 시작](quick-start.md) - Claude Code 시작하기
- [Cloudflare Pages 시작하기](pages-getting-started.md) - 프론트엔드 프로젝트 생성
- [Cloudflare Workers 시작하기](workers-getting-started.md) - 백엔드 프로젝트 생성
- [컨텍스트 관리](context-management.md) - 장기 프로젝트 관리법

---

[← 목차로 돌아가기](../_sidebar.md)
