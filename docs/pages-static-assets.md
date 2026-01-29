# Cloudflare Pages - 정적 에셋 관리

CSS, 이미지, 폰트 등 정적 파일의 최적화와 관리 방법을 학습합니다.

## CSS 구조

### 모듈화된 CSS

```
css/
├── base.css              # 기본 스타일, CSS 리셋
├── variables.css         # CSS 변수
├── layout.css            # 레이아웃 (헤더, 푸터, 그리드)
├── typography.css        # 타이포그래피
└── components/           # 컴포넌트별 스타일
    ├── header.css
    ├── card.css
    ├── button.css
    └── form.css
```

### base.css

```css
/* CSS 리셋 */
*,
*::before,
*::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

html {
    font-size: 16px;
    line-height: 1.6;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
                 'Helvetica Neue', Arial, sans-serif;
    color: var(--text-color);
    background-color: var(--bg-color);
}

/* 유틸리티 클래스 */
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border-width: 0;
}
```

### variables.css

```css
:root {
    /* 색상 */
    --primary: #3b82f6;
    --secondary: #64748b;
    --success: #10b981;
    --danger: #ef4444;
    --warning: #f59e0b;

    --text-color: #1f2937;
    --bg-color: #ffffff;
    --border-color: #e5e7eb;

    /* 간격 */
    --spacing-xs: 0.25rem;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 1.5rem;
    --spacing-xl: 2rem;

    /* 폰트 크기 */
    --text-xs: 0.75rem;
    --text-sm: 0.875rem;
    --text-base: 1rem;
    --text-lg: 1.125rem;
    --text-xl: 1.25rem;
    --text-2xl: 1.5rem;
    --text-3xl: 1.875rem;

    /* 반응형 중단점 */
    --breakpoint-sm: 640px;
    --breakpoint-md: 768px;
    --breakpoint-lg: 1024px;
    --breakpoint-xl: 1280px;
}

/* 다크 모드 */
@media (prefers-color-scheme: dark) {
    :root {
        --text-color: #f9fafb;
        --bg-color: #111827;
        --border-color: #374151;
    }
}
```

## 이미지 최적화

### 이미지 포맷 선택

**권장 포맷:**
- **JPEG:** 사진 (압축률 높음)
- **PNG:** 투명 배경, 아이콘
- **SVG:** 로고, 아이콘 (확대 가능)
- **WebP:** 최신 브라우저 (파일 크기 작음)

### 반응형 이미지

```html
<picture>
    <source srcset="/assets/images/hero.webp" type="image/webp">
    <source srcset="/assets/images/hero.jpg" type="image/jpeg">
    <img src="/assets/images/hero.jpg" alt="히어로 이미지" loading="lazy">
</picture>
```

### 이미지 지연 로딩

```html
<img src="/assets/images/thumbnail.jpg"
     alt="썸네일"
     loading="lazy"
     width="300"
     height="200">
```

**loading="lazy":**
- 뷰포트에 들어올 때 로드
- 초기 페이지 속도 향상

### Cloudflare Images 사용

```html
<!-- 원본 -->
<img src="https://imagedelivery.net/xxx/hero.jpg" alt="히어로">

<!-- 리사이즈 (300x200) -->
<img src="https://imagedelivery.net/xxx/hero.jpg/w=300,h=200" alt="히어로">

<!-- 포맷 변환 (WebP) -->
<img src="https://imagedelivery.net/xxx/hero.jpg/format=webp" alt="히어로">
```

## 폰트 최적화

### 웹 폰트 로드

```css
@font-face {
    font-family: 'Custom Font';
    src: url('/assets/fonts/custom-font.woff2') format('woff2'),
         url('/assets/fonts/custom-font.woff') format('woff');
    font-weight: 400;
    font-style: normal;
    font-display: swap;  /* FOUT 방지 */
}

body {
    font-family: 'Custom Font', sans-serif;
}
```

**font-display 옵션:**
- `swap`: 폴백 폰트 표시 → 웹 폰트 교체
- `block`: 웹 폰트 로드까지 대기 (3초)
- `fallback`: 100ms 대기 → 폴백 폰트 표시
- `optional`: 네트워크 상태에 따라 결정

### Google Fonts 사용

```html
<!-- index.html -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
```

```css
body {
    font-family: 'Inter', sans-serif;
}
```

## 아이콘 관리

### SVG 스프라이트

`assets/images/icons.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
    <symbol id="icon-home" viewBox="0 0 24 24">
        <path d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"/>
    </symbol>

    <symbol id="icon-user" viewBox="0 0 24 24">
        <path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/>
        <circle cx="12" cy="7" r="4"/>
    </symbol>
</svg>
```

**HTML에서 사용:**

```html
<svg class="icon">
    <use xlink:href="/assets/images/icons.svg#icon-home"></use>
</svg>
```

```css
.icon {
    width: 24px;
    height: 24px;
    fill: currentColor;
}
```

### Font Awesome 사용

```html
<!-- CDN -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

<!-- HTML -->
<i class="fas fa-home"></i>
<i class="fas fa-user"></i>
```

## 캐싱 전략

### _headers 파일

```
# 이미지 - 1년 캐싱
/assets/images/*
  Cache-Control: public, max-age=31536000, immutable

# CSS/JS - 1년 캐싱 (해시 포함 가정)
/css/*
  Cache-Control: public, max-age=31536000

/src/*
  Cache-Control: public, max-age=31536000

# HTML - 캐싱 안 함
/*.html
  Cache-Control: no-cache
```

### 파일 해싱

**빌드 시 해시 추가:**

```
css/base.css         → css/base.abc123.css
src/logic/router.js  → src/logic/router.def456.js
```

**index.html 업데이트:**

```html
<link rel="stylesheet" href="/css/base.abc123.css">
<script type="module" src="/src/logic/router.def456.js"></script>
```

## 성능 최적화

### 리소스 힌트

```html
<!DOCTYPE html>
<html>
<head>
    <!-- DNS Prefetch -->
    <link rel="dns-prefetch" href="https://fonts.googleapis.com">

    <!-- Preconnect -->
    <link rel="preconnect" href="https://api.example.com">

    <!-- Preload -->
    <link rel="preload" href="/css/base.css" as="style">
    <link rel="preload" href="/assets/fonts/custom-font.woff2" as="font" type="font/woff2" crossorigin>

    <!-- CSS -->
    <link rel="stylesheet" href="/css/base.css">
</head>
<body>
    <!-- 콘텐츠 -->
</body>
</html>
```

### 중요 CSS 인라인

```html
<head>
    <style>
        /* Critical CSS */
        body { margin: 0; font-family: sans-serif; }
        .container { max-width: 1200px; margin: 0 auto; }
        /* ... */
    </style>

    <!-- 나머지 CSS는 비동기 로드 -->
    <link rel="preload" href="/css/base.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
    <noscript><link rel="stylesheet" href="/css/base.css"></noscript>
</head>
```

## 압축 및 최적화

### Gzip/Brotli 압축

Cloudflare Pages는 자동으로 압축합니다.

**지원 파일:**
- HTML
- CSS
- JavaScript
- JSON
- XML

### 이미지 압축

**도구:**
- ImageOptim (macOS)
- TinyPNG (온라인)
- Squoosh (온라인)

**CLI:**

```bash
# JPEG
jpegoptim --max=85 *.jpg

# PNG
optipng -o7 *.png
pngquant --quality=65-80 *.png
```

## 실전 프롬프트 예시

### 반응형 이미지 갤러리

```
반응형 이미지 갤러리를 만들어줘.

경로: /gallery
레이아웃: 그리드 (3열 → 2열 → 1열)

기능:
- 썸네일 지연 로딩
- 클릭 시 라이트박스
- WebP 지원 (폴백 JPEG)
- 이미지 캡션

assets/images/gallery/ 폴더 사용.
```

### 다크 모드 토글

```
다크 모드 토글 기능을 추가해줘.

위치: 헤더 우측
저장: localStorage

기능:
- 아이콘 토글 버튼
- CSS 변수 전환 (--text-color, --bg-color 등)
- 시스템 설정 감지 (prefers-color-scheme)
- 부드러운 전환 효과

variables.css 업데이트.
```

### 로딩 애니메이션

```
페이지 로딩 애니메이션을 만들어줘.

위치: #app 영역
표시 타이밍: 라우터 네비게이션 중

디자인:
- 스피너 애니메이션
- 반투명 오버레이
- 중앙 정렬

CSS 애니메이션 사용.
```

## 체크리스트

정적 에셋 관리 확인사항:

- [ ] CSS가 모듈화되어 있는가?
- [ ] 이미지에 width/height 속성이 있는가?
- [ ] loading="lazy"를 사용했는가?
- [ ] 웹 폰트가 최적화되어 있는가? (woff2, font-display)
- [ ] 캐싱 헤더가 설정되어 있는가?
- [ ] 리소스 힌트를 사용했는가? (preconnect, preload)
- [ ] 반응형 디자인이 적용되어 있는가?
- [ ] 다크 모드를 지원하는가?

## 자주 하는 실수

### 1. 이미지 크기 지정 누락

```html
<!-- ❌ 잘못된 코드 (CLS 발생) -->
<img src="/assets/images/hero.jpg" alt="히어로">

<!-- ✅ 올바른 코드 -->
<img src="/assets/images/hero.jpg" alt="히어로" width="1200" height="600">
```

### 2. 폰트 플래시 (FOUT)

```css
/* ❌ 잘못된 코드 */
@font-face {
    font-family: 'Custom Font';
    src: url('/fonts/custom.woff2');
}

/* ✅ 올바른 코드 */
@font-face {
    font-family: 'Custom Font';
    src: url('/fonts/custom.woff2');
    font-display: swap;  /* FOUT 방지 */
}
```

### 3. 캐싱 미적용

```
# ❌ _headers 파일 없음

# ✅ _headers 파일 작성
/assets/*
  Cache-Control: public, max-age=31536000
```

## 관련 문서

- [프로젝트 구조](pages-structure.md)
- [라우팅 및 페이지 개발](pages-routing.md)
- [빌드 및 배포](pages-deployment.md)
- [베스트 프랙티스](best-practices.md)
