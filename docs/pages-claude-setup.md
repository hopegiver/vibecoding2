# Pages .claude 설정 예제

## 개요

이 문서는 Cloudflare Pages (ViewLogic 기반) 프로젝트를 위한 `.claude` 폴더 설정 예제를 제공합니다. 실제 운영 중인 성과운영 시스템 프로젝트의 설정을 기반으로 작성되었습니다.

## 프로젝트 구조

```
your-pages-project/
├── .claude/
│   └── rules/
│       ├── style-guide.md          # CSS/HTML 스타일 규칙
│       └── viewlogic-guide.md      # ViewLogic 개발 규칙
├── CLAUDE.md                       # 프로젝트 컨텍스트
├── src/
│   ├── views/                      # HTML 템플릿
│   └── logic/                      # JavaScript 로직
└── css/
    └── base.css                    # 전역 CSS
```

## 1. `.claude/rules/style-guide.md`

CSS와 HTML 작성 규칙을 정의합니다.

```markdown
# CSS 스타일 핵심 규칙

## ⚡ 최우선 원칙

**Bootstrap 5를 최대한 활용, Custom CSS는 최소화**
- Layout: `d-flex`, `row`, `col-*`, `gap-*`
- Spacing: `p-*`, `m-*`, `mb-3`, `gap-2`
- Text: `fw-bold`, `text-center`, `text-secondary`
- Border: `rounded-3`, `border`, `shadow-sm`

## 🚫 절대 금지

**HTML 파일에 `<style>` 태그 사용 금지**
\`\`\`html
<!-- ❌ 금지 -->
<style>.sidebar { width: 250px; }</style>

<!-- ✅ 올바름 -->
모든 CSS는 css/base.css 에 작성
\`\`\`

## 필수 규칙

1. **CSS 변수 사용**
   \`\`\`css
   /* 항상 CSS 변수 사용 */
   color: var(--primary-color);
   background: var(--gray-100);
   \`\`\`

2. **주요 색상**
   - `--primary-color: #6366f1` - 주요 버튼, 활성 메뉴
   - `--success-color: #10b981` - 완료, 달성
   - `--danger-color: #ef4444` - 삭제, Red Flag
   - `--warning-color: #f59e0b` - 경고
   - `--growth-color: #8b5cf6` - 성장 관련

3. **신호등 시스템**
   - `--signal-red: #ef4444` - 40% 미만
   - `--signal-yellow: #f59e0b` - 40-70%
   - `--signal-green: #10b981` - 70% 이상

4. **주요 클래스**
   \`\`\`css
   .stat-card          /* 통계 카드 */
   .stat-icon.primary  /* 아이콘 색상 */
   .signal-light.green /* 신호등 */
   .progress-fill.red  /* 진행바 */
   .red-flag-alert     /* 위험 알림 */
   .badge.success      /* 배지 */
   \`\`\`

5. **반응형**
   - 모바일: `@media (max-width: 768px)`
   - 태블릿: `@media (max-width: 1024px)` - 사이드바 아이콘 모드
   - Bootstrap grid 사용: `col-12 col-md-6 col-xl-3`
```

## 2. `.claude/rules/viewlogic-guide.md`

ViewLogic 프레임워크의 개발 패턴과 규칙을 정의합니다.

```markdown
# ViewLogic 핵심 개발 규칙

## 파일 구조

\`\`\`
src/
├── views/         # HTML만 (CSS 금지)
│   └── goals/my-goals.html
└── logic/         # JavaScript만
    └── goals/my-goals.js
\`\`\`

**규칙:**
- 파일명 동일: `my-goals.html` ↔ `my-goals.js`
- 폴더명 = 라우트: `goals/my-goals` → `#/goals/my-goals`

## 기본 구조

\`\`\`javascript
export default {
    layout: 'default',  // 또는 null (로그인 등)

    data() {
        return {
            items: [],
            loading: false
        }
    },

    async mounted() {
        await this.loadData();
    },

    methods: {
        async loadData() {
            const response = await this.$api.get('/api/data');
            this.items = response.data;
        }
    }
}
\`\`\`

## 필수 패턴

1. **페이지 이동**
   \`\`\`javascript
   this.navigateTo('/goals/my-goals');
   this.navigateTo('/goals', { id: 123 });
   \`\`\`

2. **상세보기 라우팅** ⚠️ 중요
   \`\`\`javascript
   // ✅ 올바름: navigateTo + 쿼리 파라미터
   this.navigateTo('/goals/detail', { id: 123 });
   this.navigateTo('/team/member-goals', { id: 5 });

   // ❌ 금지: window.location 직접 조작
   window.location.hash = '#/goals/detail?id=123';

   // ❌ 금지: 라우트 경로 파라미터
   this.navigateTo('/goals/detail/123');
   \`\`\`

3. **파라미터 받기**
   \`\`\`javascript
   data() {
       return {
           id: this.getParam('id'),
           allParams: this.getParams()
       }
   }
   \`\`\`

4. **API 호출**
   \`\`\`javascript
   await this.$api.get('/api/goals');
   await this.$api.post('/api/goals', data);
   await this.$api.put('/api/goals/123', data);
   await this.$api.delete('/api/goals/123');
   \`\`\`

5. **모달 처리**
   \`\`\`javascript
   mounted() {
       this.$nextTick(() => {
           this.modalInstance = new bootstrap.Modal(
               document.getElementById('myModal')
           );
       });
   }
   \`\`\`

6. **폼 제출**
   \`\`\`html
   <form @submit.prevent="handleSubmit">
   \`\`\`

## ViewLogic 내장 메서드

**라우팅**
\`\`\`javascript
this.navigateTo('/goals/my-goals');           // 페이지 이동
this.navigateTo('/goals', { id: 123 });       // 파라미터와 함께 이동
this.getCurrentRoute();                        // 현재 라우트 정보
this.getParam('id');                          // 단일 파라미터
this.getParams();                             // 모든 파라미터
\`\`\`

**인증**
\`\`\`javascript
this.isAuth();                                // 로그인 여부
this.getToken();                              // 인증 토큰
this.logout();                                // 로그아웃
\`\`\`

**데이터**
\`\`\`javascript
await this.fetchData('/api/goals');           // 간편 API 호출
this.$state.user;                             // 전역 상태 접근
\`\`\`

**다국어**
\`\`\`javascript
this.$t('common.save');                       // 번역 텍스트
this.getLanguage();                           // 현재 언어
this.setLanguage('ko');                       // 언어 변경
\`\`\`

## 금지 사항

- ❌ HTML 파일에 `<style>` 태그
- ❌ `layout: false` (null 사용)
- ❌ index를 key로 사용 (`:key="index"`)
- ❌ Promise then/catch (async/await 사용)
- ❌ `window.location.hash`, `window.location.href` (navigateTo 사용)

## computed vs methods

- **computed**: 자주 변경되지 않는 계산값
- **methods**: 매번 새로 계산해야 하는 값
```

## 3. `CLAUDE.md`

프로젝트 전체 컨텍스트를 제공합니다.

```markdown
# 성과운영 시스템 (GPM - Goal Performance Management)

## 프로젝트 개요
목표 중심의 성과 관리 및 개인 성장 플랫폼. OKR/MBO 기반 목표 설정, 실행 관리, 성장 추적, 평가를 통합 지원.

**기술 스택:** Vue 3 + ViewLogic Router + Bootstrap 5 + Chart.js

## 핵심 아키텍처

### 1. ViewLogic Router 패턴
\`\`\`
src/
├── views/         # HTML 템플릿 (CSS 금지)
│   ├── goals/my-goals.html
│   └── team/tasks.html
└── logic/         # JavaScript 로직
    ├── goals/my-goals.js
    └── team/tasks.js
\`\`\`
- **파일명 = 라우트**: `goals/my-goals.html` → `#/goals/my-goals`
- **분리 원칙**: HTML과 JS 완전 분리

### 2. 역할 기반 메뉴 (Role-Based Menu)

**일반 직원 (EMPLOYEE)**
- 목표: 나의 목표, 팀 목표 보기, 회사 목표 보기
- 실행: 오늘의 업무, 이번 주, 주간 보고서, 월간 통계
- 성장: 나의 성장 맵, 학습 & 개발, 역량 진단
- 평가: 자기평가, 평가 이력

**팀장 (TEAM_LEADER, DEPT_HEAD)** - 직원 메뉴 +
- 목표: 팀 목표 관리 (팀 목표 보기 대신)
- 팀 관리:
  - 팀원 업무 현황 (`/team/tasks`)
  - 팀 실행 현황 (`/team/status`)
  - 팀원 주간보고서 (`/team/weekly-reports`)
  - 팀원 성과
- 성장: 팀원 성장 지원
- 평가: 팀원 평가

## BSC (Balanced Scorecard) 관점

회사 목표는 4개 관점으로 분류:
- **재무** (primary) - 매출, 수익성
- **고객** (success) - 만족도, 유지율
- **프로세스** (warning) - 효율성, 품질
- **학습과성장** (info) - 교육, 혁신

팀/개인 목표는 반드시 회사 KPI와 연계 (`companyKPIId`)

## 주요 데이터 흐름

### Mock API 패턴 (개발 중)
\`\`\`javascript
// 현재: Mock JSON 파일 사용
const response = await fetch('/mock-api/company-goals.json');
const data = await response.json();

// 향후: 실제 API로 전환
// const data = await this.$api.get('/api/goals/company');
\`\`\`

### 목표 데이터 구조
\`\`\`javascript
{
  id: 1,
  companyKPIId: 1,          // 회사 목표 연계 (필수)
  category: "재무",          // BSC 관점
  title: "신규 프로덕트 출시 3개 이상",
  description: "...",
  targetValue: 3,           // 목표 수치
  currentValue: 2,          // 현재 수치
  unit: "개",               // 단위
  achievement: 70,          // 달성률 (%)
  dueDate: "2024-12-31",
  owner: "김민수",
  status: "진행중"          // 진행중|완료|지연|보류
}
\`\`\`

## CSS 규칙

### 절대 원칙
❌ **HTML 파일에 `<style>` 태그 절대 금지**
✅ 모든 CSS는 `css/base.css`에 작성

### Bootstrap 우선
\`\`\`html
<!-- ✅ 올바름 -->
<div class="d-flex gap-3 mb-4">
  <div class="col-12 col-md-6">

<!-- ❌ 금지 -->
<div style="display: flex; gap: 12px;">
\`\`\`

## 핵심 페이지 설명

### 팀 목표 관리 (`/goals/team-goals`)
- 팀장만 접근 가능
- 회사 KPI와 연계된 팀 목표 생성/수정/삭제
- BSC 관점별 회사 KPI 선택 (optgroup)
- 목표 수치, 단위, 담당자, 마감일 필수 입력

### 팀원 업무 현황 (`/team/tasks`)
- 팀원들의 오늘 업무를 카드 형식으로 표시 (3열)
- 실시간 완료/미완료 체크리스트
- 팀 전체 통계: 총 업무, 완료, 진행중, 평균 완료율

## 글로벌 함수

\`\`\`javascript
window.getCurrentUser()  // 현재 사용자 정보
window.hasRole(role)     // 역할 확인
window.isManager()       // 팀장 이상 여부
window.isExecutive()     // 임원 여부
\`\`\`

## 주의사항

- ✅ `layout: 'default'` 사용 (null 사용 금지)
- ✅ `:key`는 고유 ID 사용 (index 금지)
- ✅ async/await 사용 (Promise then/catch 금지)
- ✅ `@submit.prevent` 폼 제출
- ✅ Bootstrap Modal은 `$nextTick`에서 초기화
- ✅ 모든 경로는 hash 모드 (`#/...`)

## 파일 찾기 팁

\`\`\`bash
# 목표 관련
src/views/goals/           # 목표 화면
src/logic/goals/           # 목표 로직

# 팀 관리
src/views/team/            # 팀 관리 화면
src/logic/team/            # 팀 관리 로직

# 실행 관리
src/views/execution/       # 실행 관리 화면
src/logic/execution/       # 실행 관리 로직

# 레이아웃
src/views/layout/default.html    # 메인 레이아웃
src/logic/layout/default.js      # 레이아웃 로직
\`\`\`

---
**마지막 업데이트:** 2024-01-19
**개발 규칙:** `.claude/rules/` 폴더 참조
```

## 4. 실전 활용 예시

### 새로운 페이지 추가 요청

```
프롬프트: "팀원 성과 페이지를 만들어줘. /team/performance 경로로 추가하고,
팀원별 목표 달성률과 주간 업무 완료율을 카드로 표시해줘."
```

**Claude Code의 동작:**
1. `.claude/rules/viewlogic-guide.md` 읽고 파일 구조 파악
2. `.claude/rules/style-guide.md` 읽고 Bootstrap 클래스 사용
3. `CLAUDE.md`에서 데이터 구조와 글로벌 함수 확인
4. `src/views/team/performance.html` 생성 (CSS 없이 Bootstrap만 사용)
5. `src/logic/team/performance.js` 생성 (ViewLogic 패턴 준수)

### API 연동 요청

```
프롬프트: "Mock JSON 대신 실제 API로 회사 목표를 불러오도록 수정해줘."
```

**Claude Code의 동작:**
1. `CLAUDE.md`에서 현재 Mock API 패턴 확인
2. `.claude/rules/viewlogic-guide.md`에서 `this.$api.get()` 패턴 확인
3. 기존 코드 수정: `fetch('/mock-api/...')` → `this.$api.get('/api/goals/company')`
4. 에러 처리 추가

## 효과

이 `.claude` 설정으로:
- ✅ HTML 파일에 CSS가 들어가는 실수 방지
- ✅ ViewLogic 패턴을 일관되게 준수
- ✅ Bootstrap 클래스를 최대한 활용
- ✅ 프로젝트 구조와 데이터 흐름을 정확히 이해
- ✅ 역할 기반 권한 체크를 자동으로 추가

## 다음 단계

- [Workers .claude 설정 예제](workers-claude-setup.md)
- [맑은프레임워크 .claude 설정 예제](malgn-claude-setup.md)
- [MCP 서버 설정 및 활용](mcp-setup.md)

---

[← 목차로 돌아가기](../_sidebar.md)
