# 개발 환경 설정

## 개요

Claude Code를 사용하기 위한 **최적의 개발 환경**을 설정하는 가이드입니다.

## 1. VSCode 설치

### Windows

1. [VSCode 공식 사이트](https://code.visualstudio.com/) 접속
2. **Download for Windows** 클릭
3. 설치 프로그램 실행
4. **Add to PATH** 옵션 체크 (중요!)

### macOS

```bash
brew install --cask visual-studio-code
```

## 2. Claude Code 확장 설치

### 방법 1: VSCode Marketplace

1. VSCode 실행
2. 좌측 확장(Extensions) 아이콘 클릭 (`Ctrl+Shift+X`)
3. "Claude Code" 검색
4. **Install** 클릭

### 방법 2: 명령 팔레트

1. `Ctrl+Shift+P` (macOS: `Cmd+Shift+P`)
2. "Extensions: Install Extensions" 입력
3. "Claude Code" 검색 및 설치

### API 키 설정

1. [Anthropic Console](https://console.anthropic.com/) 가입
2. API 키 발급
3. VSCode에서 Claude Code 설정
   - `Ctrl+,` → "Claude Code" 검색
   - API Key 입력

## 3. Git 설정

Claude Code는 Git과 긴밀하게 통합됩니다.

### Git 설치

**Windows:**
- [Git for Windows](https://git-scm.com/download/win) 다운로드 및 설치

**macOS:**
```bash
brew install git
```

### Git 사용자 정보 설정

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### .gitignore 기본 설정

프로젝트 루트에 `.gitignore` 생성:

```gitignore
# Node.js
node_modules/
.env
.env.local

# Logs
*.log
npm-debug.log*

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Build
dist/
build/
.wrangler/

# Claude Code (선택)
# .claude/memory/  # 작업 이력을 Git에 포함하지 않으려면 주석 해제
```

## 4. 프로젝트별 필수 설정

### Node.js 프로젝트 (Workers, Pages)

#### Node.js 설치

**권장 버전:** Node.js 18 이상

```bash
# nvm 사용 (권장)
nvm install 18
nvm use 18

# 또는 직접 설치
# https://nodejs.org/
```

#### package.json 생성

```bash
npm init -y
```

#### 기본 의존성 설치

**Cloudflare Workers:**
```bash
npm install hono
npm install -D wrangler
```

**Cloudflare Pages:**
```bash
npm install vue@3
npm install -D @viewlogic/cli
```

### Java 프로젝트 (맑은프레임워크)

#### JDK 설치

**권장 버전:** JDK 8 이상

```bash
# Windows: Chocolatey 사용
choco install openjdk8

# macOS
brew install openjdk@8
```

#### 환경 변수 설정

**Windows:**
```
JAVA_HOME=C:\Program Files\Java\jdk-8
Path=%JAVA_HOME%\bin;...
```

**macOS/Linux:**
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk
export PATH=$JAVA_HOME/bin:$PATH
```

## 5. VSCode 권장 확장

Claude Code와 함께 사용하면 좋은 확장들:

### 필수 확장

- **GitLens** - Git 히스토리 시각화
- **ESLint** - JavaScript 린팅
- **Prettier** - 코드 포맷팅
- **Error Lens** - 에러 인라인 표시

### 언어별 확장

**JavaScript/TypeScript:**
- Volar (Vue)
- ES7+ React/Redux/React-Native snippets

**Java:**
- Extension Pack for Java

## 6. 데이터베이스 설정

### MySQL 설치

맑은소프트 프로젝트는 **MySQL**을 기본 데이터베이스로 사용합니다.

**Windows:**
- [MySQL Community Server](https://dev.mysql.com/downloads/mysql/) 다운로드 및 설치

**macOS:**
```bash
brew install mysql
brew services start mysql
```

### Cloudflare Hyperdrive 연동 (Workers)

Workers 프로젝트에서는 **Hyperdrive**를 통해 MySQL에 연결합니다.

#### wrangler.toml 설정

```toml
name = "my-worker"
main = "src/index.js"

[[hyperdrive]]
binding = "DB"
id = "your-hyperdrive-id"
```

#### 사용 예시

```javascript
export default {
  async fetch(request, env) {
    const db = env.DB;
    const results = await db.query('SELECT * FROM users');
    return Response.json(results);
  }
}
```

자세한 내용은 [Cloudflare Hyperdrive 문서](https://developers.cloudflare.com/hyperdrive/) 참조.

## 7. 프로젝트 초기 구조

### 권장 폴더 구조

```
your-project/
├── .claude/                # Claude Code 설정
│   ├── rules/             # 개발 규칙
│   ├── templates/         # 코드 템플릿
│   └── memory/            # 작업 이력 (선택)
├── CLAUDE.md              # 프로젝트 컨텍스트
├── .gitignore
├── README.md
└── src/                   # 소스 코드
```

### .claude 폴더 생성

```bash
mkdir -p .claude/rules
mkdir -p .claude/templates
```

### CLAUDE.md 템플릿 복사

프로젝트 타입에 맞는 템플릿 사용:

- [Workers CLAUDE.md 예제](workers-claude-setup.md)
- [Pages CLAUDE.md 예제](pages-claude-setup.md)
- [맑은프레임워크 CLAUDE.md 예제](malgn-claude-setup.md)

## 8. 환경별 설정 파일

### .env 파일 생성

```bash
# .env (Git에 포함하지 말 것!)
DATABASE_URL=mysql://user:password@localhost:3306/mydb
API_KEY=your-secret-key
JWT_SECRET=your-jwt-secret

# Cloudflare (Workers)
HYPERDRIVE_ID=your-hyperdrive-id
```

### .env.example 파일

```bash
# .env.example (Git에 포함)
DATABASE_URL=mysql://user:password@localhost:3306/dbname
API_KEY=
JWT_SECRET=
HYPERDRIVE_ID=
```

## 9. 설정 확인

모든 설정이 완료되었는지 확인:

```bash
# Node.js 버전 확인
node --version  # v18.x.x 이상

# Git 확인
git --version

# Claude Code 확장 확인 (VSCode에서)
# Ctrl+Shift+P → "Claude Code: Open Chat"
```

## 10. 다음 단계

설정이 완료되었다면 첫 프로젝트를 만들어보세요!

- [첫 프로젝트 만들기](first-project.md) - 실전 CRUD 프로젝트
- [.claude/rules 작성 가이드](claude-rules.md) - 프로젝트 규칙 정의
- [CLAUDE.md 작성 가이드](claude-md.md) - 프로젝트 컨텍스트 문서

## 문제 해결

### Claude Code가 보이지 않음

- VSCode 재시작
- 확장 탭에서 "Claude Code" 검색 후 다시 설치

### API 키 오류

- Anthropic Console에서 API 키 확인
- 키가 유효한지, 크레딧이 남아있는지 확인

### Git 명령이 작동하지 않음

- PATH 환경 변수에 Git이 포함되어 있는지 확인
- 터미널 재시작

---

[← 목차로 돌아가기](../_sidebar.md)
