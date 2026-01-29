# 다음 할 일

## 우선순위 높음

### 섹션 5: 맑은프레임워크 실전 가이드 (7개 문서 남음)

- [ ] **프로젝트 시작하기** (malgn-getting-started.md)
  - 프로젝트 초기 설정
  - 디렉토리 구조 생성
  - init.jsp 설정
  - 첫 페이지 만들기

- [ ] **페이지 및 라우팅 개발** (malgn-pages-routing.md)
  - JSP 페이지 생성
  - HTML 템플릿 작성
  - 페이지 라우팅
  - 레이아웃 설정

- [ ] **컴포넌트 개발** (malgn-components.md)
  - Page 클래스 활용
  - 반복 템플릿 (loop)
  - 조건부 표시 (if)
  - 변수 전달 (setVar)

- [ ] **API 개발 및 연동** (malgn-api.md)
  - DAO 클래스 작성
  - CRUD 패턴
  - 트랜잭션 처리
  - 에러 처리

- [ ] **데이터베이스 작업** (malgn-database.md)
  - DataSet 활용
  - ListManager (페이징)
  - 검색 조건 추가
  - 정렬 및 필터링

- [ ] **상태 관리** (malgn-state.md)
  - 세션 관리
  - 쿠키 활용
  - 파라미터 처리 (m.rs, f.get)
  - Postback 패턴

- [ ] **테스트 작성** (malgn-testing.md)
  - 단위 테스트 (Java)
  - 통합 테스트
  - 테스트 데이터 준비

- [ ] **배포 및 운영** (malgn-deployment.md)
  - WAR 파일 생성
  - 서버 배포
  - 로그 모니터링
  - 성능 최적화

### 섹션 6: Pages 실전 가이드 (7개 문서 남음)

- [ ] **프로젝트 시작하기** (pages-getting-started.md)
  - ViewLogic Router 설정
  - 프로젝트 초기화
  - 첫 페이지 만들기

- [ ] **Pages 프로젝트 구조** (pages-structure.md)
  - views/ 폴더 구조
  - logic/ 폴더 구조
  - css/ 구조
  - 파일 명명 규칙

- [ ] **라우팅 및 페이지 개발** (pages-routing.md)
  - 라우트 정의
  - 페이지 네비게이션
  - 파라미터 전달
  - 레이아웃 설정

- [ ] **Functions 개발** (pages-functions.md)
  - Pages Functions 생성
  - API 엔드포인트
  - 데이터 연동

- [ ] **정적 에셋 관리** (pages-static-assets.md)
  - 이미지 최적화
  - CSS 구조
  - JavaScript 번들링

- [ ] **API 엔드포인트 개발** (pages-api.md)
  - Functions API
  - 데이터 페칭
  - 에러 처리

- [ ] **환경 변수 및 설정** (pages-environment.md)
  - 환경 변수 설정
  - 빌드 설정
  - 배포 설정

- [ ] **빌드 및 배포** (pages-deployment.md)
  - Cloudflare Pages 배포
  - 커스텀 도메인
  - 프리뷰 배포

## 우선순위 중간

### 섹션 7: Workers 실전 가이드 (8개 문서 남음)

- [ ] **프로젝트 시작하기** (workers-getting-started.md)
- [ ] **라우팅 및 요청 처리** (workers-routing.md)
- [ ] **API 개발** (workers-api.md)
- [ ] **KV 스토리지 활용** (workers-kv.md)
- [ ] **D1 데이터베이스 활용** (workers-d1.md)
- [ ] **R2 객체 스토리지 활용** (workers-r2.md)
- [ ] **Durable Objects 활용** (workers-durable-objects.md)
- [ ] **테스트 및 디버깅** (workers-testing.md)
- [ ] **배포 및 모니터링** (workers-deployment.md)

## 우선순위 낮음

### 섹션 8: Claude Code 도구 활용 (4개 문서)

- [ ] **파일 읽기/쓰기 작업** (file-operations.md)
  - Read, Write, Edit 도구
  - 파일 탐색
  - 코드 수정 패턴

- [ ] **코드 검색 및 탐색** (code-navigation.md)
  - Grep, Glob 활용
  - Task 에이전트 사용
  - 코드베이스 탐색

- [ ] **Git 작업 자동화** (git-automation.md)
  - 커밋 자동화
  - PR 생성
  - 브랜치 관리

- [ ] **터미널 명령 실행** (terminal-commands.md)
  - Bash 도구 활용
  - 스크립트 실행
  - 빌드 자동화

### 섹션 9: 문제 해결 (3개 문서)

- [ ] **일반적인 오류 해결** (troubleshooting.md)
  - 자주 발생하는 에러
  - 해결 방법
  - 디버깅 팁

- [ ] **성능 최적화** (performance.md)
  - 병목 지점 찾기
  - 쿼리 최적화
  - 캐싱 전략

- [ ] **디버깅 전략** (debugging.md)
  - Claude Code 디버깅
  - 로그 분석
  - 문제 격리

## 보류

### 추가 섹션 (선택)

- [ ] 고급 패턴 가이드
- [ ] 마이그레이션 가이드 (기존 프로젝트 → 바이브코딩)
- [ ] 팀 협업 가이드
- [ ] CI/CD 통합

## 작업 전략

### Phase 1: 플랫폼별 실전 가이드 (현재)
- 섹션 5, 6, 7을 순차적으로 완성
- 각 플랫폼별로 실제 사용 가능한 예제 포함
- 샘플 프로젝트 참조

### Phase 2: 도구 및 문제 해결
- 섹션 8, 9 완성
- 실전 경험 기반 팁 제공

### Phase 3: 최종 검토
- 전체 문서 일관성 확인
- 링크 검증
- 예제 코드 테스트

## 예상 소요 시간

- 섹션 5: 3-4시간
- 섹션 6: 3-4시간
- 섹션 7: 4-5시간
- 섹션 8: 2-3시간
- 섹션 9: 1-2시간

**전체 예상 소요:** 13-18시간

## 참고사항

- 각 섹션 완료 후 Git 커밋
- 토큰 크기 관리 지속
- 사용자 피드백 반영
- 실전 활용 가능한 내용 우선
