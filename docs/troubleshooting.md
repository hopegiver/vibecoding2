# 문제 해결 - 일반적인 오류 해결

자주 발생하는 오류와 해결 방법을 학습합니다.

## Claude Code 오류

### "Tool execution failed"

**원인:** 권한 부족 또는 잘못된 명령

**해결:**
```
.claude/settings.local.json에서 autoApproveTools 확인.
명령어 경로가 정확한지 확인.
```

### 메모리 부족

**원인:** 큰 파일 읽기, 복잡한 작업

**해결:**
```
파일을 부분적으로 읽기 (offset, limit 사용).
Task 도구로 작업 분할.
```

## 맑은프레임워크 오류

### "Cannot find class malgnsoft.*"

**원인:** malgn.jar 누락

**해결:**
```
WEB-INF/lib/malgn.jar 파일 확인.
빌드 경로 설정 확인.
```

### 템플릿 렌더링 오류

**원인:** 경로 불일치, 변수명 오류

**해결:**
```
p.setLayout("main") → html/layout/layout_main.html 확인.
p.setVar() 변수명과 HTML {변수명} 일치 확인.
```

### DataSet.next() 누락

**원인:** next() 호출 전 데이터 접근

**해결:**
```java
// ❌ 잘못된 코드
DataSet info = dao.findById(id);
String title = info.s("title");  // 오류!

// ✅ 올바른 코드
DataSet info = dao.findById(id);
if (info.next()) {
    String title = info.s("title");
}
```

## Cloudflare Pages 오류

### "Build failed"

**원인:** 빌드 명령 오류, 의존성 누락

**해결:**
```
package.json의 scripts 확인.
npm install로 의존성 설치.
로컬에서 빌드 테스트.
```

### Functions 오류

**원인:** 환경 변수 미설정, 바인딩 오류

**해결:**
```
Cloudflare 대시보드에서 환경 변수 확인.
wrangler.toml의 바인딩 확인.
```

## Cloudflare Workers 오류

### "Uncaught ReferenceError"

**원인:** 모듈 import 오류

**해결:**
```typescript
// wrangler.toml에서 compatibility_date 확인
compatibility_date = "2024-01-01"

// import 경로 확인
import { Router } from 'itty-router';
```

### D1 바인딩 오류

**원인:** wrangler.toml 설정 오류

**해결:**
```toml
[[d1_databases]]
binding = "DB"  # 코드에서 env.DB로 사용
database_id = "xxx-xxx-xxx"
```

## Git 오류

### "Permission denied"

**원인:** SSH 키 미설정

**해결:**
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
cat ~/.ssh/id_ed25519.pub  # GitHub에 추가
```

### "Merge conflict"

**원인:** 여러 브랜치에서 같은 파일 수정

**해결:**
```
충돌 파일 열기.
<<<<<<< HEAD와 ======= 사이 코드 확인.
필요한 코드 선택.
git add . && git commit.
```

## 실전 프롬프트 예시

### 오류 진단

```
다음 오류를 해결해줘:

Error: Cannot find module 'itty-router'

확인 사항:
- package.json에 의존성 있는지
- node_modules 설치되었는지
- import 경로 올바른지
```

### 배포 실패 해결

```
Cloudflare Pages 배포가 실패했어.

오류 로그:
[Error] Build failed: Command not found

확인:
1. 빌드 명령 확인
2. package.json scripts 확인
3. 로컬에서 빌드 테스트
```

## 체크리스트

문제 해결 시 확인사항:

- [ ] 오류 메시지를 정확히 읽었는가?
- [ ] 로그를 확인했는가?
- [ ] 최근 변경 사항을 확인했는가?
- [ ] 환경 변수가 설정되어 있는가?
- [ ] 의존성이 설치되어 있는가?

## 관련 문서

- [성능 최적화](performance.md)
- [디버깅 전략](debugging.md)
