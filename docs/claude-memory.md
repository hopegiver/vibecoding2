# .claude/memory 활용 가이드

## 개요

`.claude/memory/`는 Claude Code가 **자동으로 작업 이력을 기록**하는 폴더입니다. 프로젝트의 진행 상황, 작업 히스토리, 다음 할 일을 추적하여 일관된 개발을 지원합니다.

## 왜 필요한가?

### Memory 없이 작업할 때

```
개발자: "어제 작업하던 거 이어서 해줘"
Claude: "죄송하지만 이전 작업 내용을 모릅니다. 다시 설명해주세요."

개발자: (10분간 이전 작업 설명...)
```

### Memory 사용 시

```
개발자: "어제 작업하던 거 이어서 해줘"
Claude: ".claude/memory를 확인했습니다.
어제 Product DAO 작성 중이었고, search 메소드만 남았네요.
바로 완성하겠습니다."

(즉시 작업 재개)
```

## 기본 구조

```
.claude/memory/
├── status.md       # 현재 프로젝트 상태
├── history.md      # 작업 이력 (최근 10개)
└── next-tasks.md   # 다음 할 일
```

### status.md

**현재 진행 상황 요약**

```markdown
# 프로젝트 상태

## 진행률
- 전체: 70% 완료
- 섹션 1-4: 완료 ✅
- 섹션 5-9: 진행 중 🚧

## 현재 작업
- Product CRUD API 구현 중
- DAO 레이어: 완료
- Service 레이어: 70%
- Controller: 대기 중

## 블로커
- D1 데이터베이스 마이그레이션 대기
```

### history.md

**최근 작업 기록**

```markdown
# 작업 이력

## 2024-01-30 16:30
- Product DAO 작성 완료
- CRUD 메소드 구현
- 페이징, 검색 기능 추가

## 2024-01-30 14:00
- Product 테이블 스키마 설계
- D1 마이그레이션 파일 작성

## 2024-01-29 15:00
- User API 완성 및 테스트
```

### next-tasks.md

**다음 할 일 목록**

```markdown
# 다음 작업

## 우선순위 높음
- [ ] Product Service 레이어 완성
- [ ] Product Controller 구현
- [ ] 통합 테스트 작성

## 우선순위 중간
- [ ] API 문서 자동 생성
- [ ] 에러 핸들링 개선

## 나중에
- [ ] 캐싱 레이어 추가
- [ ] 성능 최적화
```

## 실전 활용 패턴

### 패턴 1: 작업 중단 및 재개

**중단 시:**
```
개발자: "오늘 작업 내용을 memory에 기록해줘"

Claude: ".claude/memory/status.md와 history.md를 업데이트했습니다.

현재 상태:
- Product Service의 create, update 완료
- delete, search 메소드 미완성
- 내일 이 부분부터 재개하면 됩니다."
```

**재개 시:**
```
개발자: "어제 작업 이어서 해줘"

Claude: "memory 확인했습니다.
Product Service의 delete, search 메소드를 완성하겠습니다."

(즉시 작업 계속)
```

### 패턴 2: 장기 프로젝트 관리

**매주 진행 상황 확인:**
```
개발자: "이번 주 작업 내용 요약해줘"

Claude: "memory/history.md를 확인했습니다.

이번 주 완료:
✅ User 모듈 완성 (월~화)
✅ Product 모듈 70% (수~금)

다음 주 계획:
🚧 Product 모듈 완성 (월)
🚧 Order 모듈 시작 (화~)"
```

### 패턴 3: 팀 인수인계

**인수인계 문서 자동 생성:**
```
개발자: "memory를 기반으로 인수인계 문서 만들어줘"

Claude: "HANDOVER.md 파일을 생성했습니다.

포함 내용:
- 프로젝트 개요
- 완료된 작업 (history 기반)
- 진행 중 작업 (status 기반)
- 향후 계획 (next-tasks 기반)
- 주의사항 및 블로커
- 주요 파일 위치"
```

### 패턴 4: 버그 추적

**버그 발견 시:**
```
개발자: "Product 검색에서 한글 검색이 안 돼. memory에 버그로 기록해줘"

Claude: ".claude/memory/issues.md에 기록했습니다.

BUG-001: Product 검색 한글 미지원
- 발견일: 2024-01-30
- 위치: src/services/productService.js:45
- 증상: 한글 검색 시 결과 없음
- 우선순위: 높음"
```

**버그 수정 시:**
```
개발자: "BUG-001 수정했어. memory 업데이트해줘"

Claude: "issues.md 업데이트 및 history.md에 기록했습니다.

BUG-001: ✅ 해결
- 해결일: 2024-01-30 17:00
- 원인: URL 인코딩 누락
- 수정: encodeURIComponent 추가"
```

### 패턴 5: 성과 리뷰

**월말 리뷰:**
```
개발자: "이번 달 작업 성과를 정리해줘"

Claude: "history.md 기반 월간 리포트:

📊 통계:
- 완료된 기능: 15개
- 수정된 버그: 8개
- 작성된 테스트: 42개
- 커밋 횟수: 87회

🎯 주요 성과:
1. User 인증 시스템 완성
2. Product CRUD API 완성
3. Order 처리 플로우 50%

💡 배운 점:
- D1 데이터베이스 트랜잭션 처리
- JWT 토큰 관리 패턴
- 테스트 자동화 구축"
```

## Memory 관리 팁

### 1. 정기적 정리

**매주 금요일:**
```
개발자: "memory를 정리해줘. 오래된 항목은 아카이브하고"

Claude: "정리 완료:
- history.md: 최근 10개만 유지, 나머지는 archive/2024-01.md로 이동
- next-tasks.md: 완료된 항목 제거
- status.md: 진행률 업데이트"
```

### 2. 컨텍스트 크기 관리

Memory 파일이 너무 크면 Claude의 컨텍스트를 과다하게 사용합니다.

**권장 크기:**
- status.md: 50줄 이하
- history.md: 최근 10개 항목
- next-tasks.md: 20개 항목 이하

**초과 시:**
```
개발자: "memory가 너무 커. 요약해줘"

Claude: "각 파일을 압축했습니다:
- history: 30개 → 10개 (나머지는 archive로)
- next-tasks: 완료 항목 제거
- status: 핵심만 유지"
```

### 3. 자동 업데이트 요청

**작업 완료 시마다:**
```
"작업 완료했어. memory 업데이트해줘"

Claude가 자동으로:
- history.md에 작업 기록
- next-tasks.md에서 완료 항목 제거
- status.md 진행률 갱신
```

### 4. 구조화된 형식 유지

**일관된 포맷:**
```markdown
# history.md 형식

## YYYY-MM-DD HH:MM
- 작업 제목
- 상세 내용 (간략하게)
- 관련 파일
- 참고사항
```

## MCP와 연동

Memory를 MCP 서버로 활용하면 더 강력합니다.

**설정 예시:**
```json
{
  "mcpServers": {
    "project-memory": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        ".claude/memory"
      ]
    }
  }
}
```

**사용:**
```
"@project-memory를 확인하고 어제 작업 이어서 해줘"

Claude가 자동으로:
1. memory 폴더 전체 읽기
2. 작업 맥락 파악
3. 중단된 지점부터 재개
```

## 자동화 패턴

### 슬래시 커맨드 연동

`.claude/commands/save-progress.md`:
```markdown
# 진행 상황 저장

.claude/memory/를 업데이트합니다.

1. 현재 작업 내용을 history.md에 기록
2. status.md 진행률 업데이트
3. 완료된 next-tasks 항목 제거
4. 새로운 블로커나 이슈 기록
```

**사용:**
```
/save-progress
```

자동으로 memory 전체 업데이트

### Git 커밋 시 자동 기록

`.claude/rules/git-workflow.md`:
```markdown
# Git 워크플로우

커밋 시 자동으로:
1. .claude/memory/history.md에 커밋 내용 기록
2. status.md 진행률 갱신
3. 관련 next-tasks 항목 업데이트
```

## 실전 템플릿

### status.md 템플릿

```markdown
# 프로젝트 상태

## 마지막 업데이트
2024-01-30 17:00

## 전체 진행률
60% 완료

## 모듈별 상태
| 모듈 | 상태 | 진행률 |
|------|------|--------|
| User | ✅ 완료 | 100% |
| Product | 🚧 진행중 | 70% |
| Order | ⏳ 대기 | 0% |

## 현재 작업
- Product Service 레이어 구현 중
- 다음: Product Controller

## 블로커
- 없음
```

### history.md 템플릿

```markdown
# 작업 이력

## 2024-01-30 17:00
**작업:** Product Service search 메소드 구현
- 파일: src/services/productService.js
- 내용: 제품명, 카테고리별 검색 기능
- 테스트: ✅ 통과

## 2024-01-30 14:30
**작업:** Product DAO 완성
- 파일: src/dao/productDao.java
- 내용: CRUD + 페이징 + 검색
- 커밋: abc123f
```

### next-tasks.md 템플릿

```markdown
# 다음 작업

## 🔴 긴급 (오늘)
- [ ] Product Controller 구현
- [ ] 통합 테스트 작성

## 🟡 중요 (이번 주)
- [ ] API 문서 생성
- [ ] 에러 핸들링 개선

## 🟢 일반 (다음 주)
- [ ] 캐싱 레이어 추가
- [ ] 성능 최적화

## 💡 아이디어
- Elasticsearch 검색 엔진 도입 검토
- GraphQL API 추가 고려
```

## Memory 활용 체크리스트

- [ ] 매일 작업 종료 시 memory 업데이트
- [ ] 매주 금요일 memory 정리 및 아카이브
- [ ] 작업 중단 시 명확한 중단 지점 기록
- [ ] 블로커 발생 시 즉시 status.md에 기록
- [ ] 주요 결정 사항 history에 기록
- [ ] Memory 크기가 너무 커지지 않도록 관리

## 관련 문서

- [.claude/commands 활용](claude-commands.md) - 슬래시 커맨드
- [CLAUDE.md 작성](claude-md.md) - 프로젝트 컨텍스트
- [MCP 서버 설정](mcp-setup.md) - Memory를 MCP로 활용

---

[← 목차로 돌아가기](../_sidebar.md)
