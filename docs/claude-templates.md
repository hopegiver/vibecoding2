# .claude/templates 활용법

## 개요

`.claude/templates` 폴더는 자주 사용하는 코드 패턴을 **재사용 가능한 템플릿**으로 저장하는 곳입니다. 신규 페이지나 기능을 추가할 때 Claude Code가 이 템플릿을 참고하여 일관된 코드를 생성합니다.

## 왜 필요한가?

프로젝트에서 반복적으로 작성하는 코드 패턴이 있습니다:
- CRUD 페이지 (목록, 등록, 수정, 상세)
- API 엔드포인트
- 인증이 필요한 페이지
- 폼 유효성 검증
- 권한 체크

이런 패턴을 템플릿으로 만들면:
- ✅ 일관된 코드 구조 유지
- ✅ 개발 시간 단축
- ✅ 실수 방지
- ✅ 베스트 프랙티스 공유

## 템플릿 작성 원칙

### 기본 구조

템플릿 파일은 **Markdown 형식**으로 작성하며 다음 구조를 권장합니다:

```markdown
# 템플릿 제목

## 사용 방법
- 이 템플릿의 목적과 사용 시기
- 플레이스홀더 설명

## 파일 1: 경로/파일명
\`\`\`언어
실제 코드 내용
\`\`\`

## 파일 2: 경로/파일명
\`\`\`언어
실제 코드 내용
\`\`\`
```

### 포함할 내용

각 템플릿은 다음 요소를 포함해야 합니다:

**1. 명확한 설명**
- 템플릿의 목적
- 언제 사용하는지
- 플레이스홀더 설명

**2. 완전한 코드**
- 실제 작동하는 코드
- 필수 import/include
- 기본 에러 처리

**3. 필수 보안 요소**
- 입력 검증
- XSS 방지
- SQL Injection 방지
- 권한 체크

**4. 주석**
- 핵심 로직 설명
- 플레이스홀더 위치 표시

## 플레이스홀더 규칙

템플릿에서 변경 가능한 부분은 `{{플레이스홀더}}` 형식으로 표시합니다.

### 네이밍 컨벤션

**엔티티/리소스명:**
- `{{entity}}`: camelCase (예: userProfile, productItem)
- `{{Entity}}`: PascalCase (예: UserProfile, ProductItem)
- `{{ENTITY}}`: UPPER_CASE (예: USER_PROFILE, PRODUCT_ITEM)

**데이터베이스:**
- `{{table}}`: snake_case (예: tb_user, tb_product)
- `{{fields}}`: 필드 목록 (예: name,email,phone)

**경로/폴더:**
- `{{folder}}`: 폴더명 (예: user, product, admin)
- `{{path}}`: 전체 경로 (예: /admin/user)

### 사용 예시

```javascript
// {{Entity}}: PascalCase 엔티티명 (예: User, Product)
export class {{Entity}}Service {
  constructor(env) {
    this.env = env;
    this.table = '{{table}}'; // 테이블명 (예: users, products)
  }
}
```

## 템플릿 유형별 구성

### CRUD 템플릿

**포함할 파일:**
- 목록 페이지 (list)
- 등록 페이지 (insert/create)
- 수정 페이지 (update)
- 상세 페이지 (view/detail)
- DAO/Service 클래스

**필수 기능:**
- 페이징
- 검색
- 정렬
- 유효성 검증
- 에러 처리

### API 템플릿

**포함할 파일:**
- Service 클래스
- Route 핸들러
- 에러 핸들러
- 응답 포맷터

**필수 기능:**
- RESTful 엔드포인트 (GET, POST, PUT, DELETE)
- 인증/권한 체크
- 입력 검증
- 캐싱 (선택)

### 페이지 템플릿

**포함할 파일:**
- Logic/Controller
- View/Template
- CSS (선택)

**필수 기능:**
- 레이아웃 설정
- 데이터 바인딩
- 이벤트 핸들러

**맑은프레임워크 예시:**
```
.claude/templates/
├── malgn-crud-list.md      # 목록 페이지
├── malgn-crud-insert.md    # 등록 페이지
├── malgn-crud-update.md    # 수정 페이지
└── malgn-dao.md            # DAO 클래스
```

**Cloudflare Workers 예시:**
```
.claude/templates/
├── workers-api-service.md  # Service 클래스
├── workers-api-route.md    # Route 핸들러
└── workers-middleware.md   # 미들웨어
```

**Cloudflare Pages 예시:**
```
.claude/templates/
├── pages-viewlogic.md      # ViewLogic 페이지
├── pages-component.md      # 재사용 컴포넌트
└── pages-api-call.md       # API 호출 패턴
```

## Claude Code 활용 방법

### 템플릿 참조 프롬프트

**기본 패턴:**
```
프롬프트: ".claude/templates/{템플릿명}.md를 참고해서 {기능}을 만들어줘."
```

**구체적 예시:**
```
프롬프트: ".claude/templates/workers-api-service.md를 참고해서
Product API를 만들어줘. 테이블은 products이고 KV 캐시를 사용해줘."
```

**Claude Code 동작:**
1. 템플릿 파일 읽기
2. 플레이스홀더 치환
3. 프로젝트 구조에 맞게 파일 생성

### 템플릿 활용 효과

**템플릿 없이:**
- 매번 구조 설명 필요
- 일관성 유지 어려움
- 필수 요소 누락 가능

**템플릿 사용:**
- 즉시 코드 생성 가능
- 검증된 패턴 적용
- 팀 전체 일관성 유지

## 템플릿 관리 팁

### 1. 실제 운영 코드 기반
- 검증된 코드를 템플릿화
- 프로덕션 환경에서 작동하는 코드 사용

### 2. 명확한 설명과 주석
- 각 플레이스홀더 설명 추가
- 핵심 로직에 주석 포함

### 3. 일관된 네이밍
- 팀 내 플레이스홀더 규칙 통일
- 문서화하여 공유

### 4. 필수 요소 포함
- 유효성 검증
- 에러 처리
- 권한 체크
- 보안 패턴

### 5. 정기적 업데이트
- 베스트 프랙티스 변경 시 반영
- 팀 피드백 반영

### 6. Git 버전 관리
- 템플릿을 Git에 커밋
- 팀 전체가 최신 템플릿 사용

## 다음 단계

- [.claude/rules 작성 가이드](claude-rules.md)
- [CLAUDE.md 작성 가이드](claude-md.md)
- [MCP 서버 설정 및 활용](mcp-setup.md)

---

[← 목차로 돌아가기](../_sidebar.md)
