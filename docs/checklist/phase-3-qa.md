# Phase 3 체크리스트 — QA / 최적화 / 배포

> 작성일: 2026-03-31
> Phase: 3 | Day: 4
> 목표: Lighthouse ≥ 90, 반응형/접근성 검증, Vercel 배포 완료

---

## 진행 상황

| 항목 | 상태 | 완료일 |
|-----|------|-------|
| Lighthouse Performance ≥ 90 | ⬜ 미완료 | - |
| Lighthouse Accessibility ≥ 90 | ⬜ 미완료 | - |
| Lighthouse Best Practices ≥ 90 | ⬜ 미완료 | - |
| Lighthouse SEO ≥ 90 | ⬜ 미완료 | - |
| 시맨틱 HTML 검증 | ⬜ 미완료 | - |
| 반응형 전 구간 확인 | ⬜ 미완료 | - |
| WCAG 2.1 AA 접근성 | ⬜ 미완료 | - |
| 에러 상태 처리 | ⬜ 미완료 | - |
| Vercel 배포 | ⬜ 미완료 | - |

> 상태 표기: ⬜ 미완료 / 🔄 진행 중 / ✅ 완료

---

## 체크리스트

### 1. 빌드 검증

- [ ] `npm run build` 에러 없음
- [ ] `npm run lint` ESLint 경고/에러 없음
- [ ] `npx tsc --noEmit` TypeScript 타입 에러 없음
- [ ] `npm run start` 프로덕션 빌드 정상 실행

### 2. 성능 최적화 (Lighthouse Performance ≥ 90)

#### 이미지 최적화

- [ ] 모든 이미지 `next/image` (`<Image>`) 컴포넌트 사용
- [ ] 이미지에 `width`, `height` 속성 명시 (CLS 방지)
- [ ] 외부 이미지 도메인 `next.config.ts`에 등록
- [ ] 히어로 이미지 `priority` 속성 설정

#### 폰트 최적화

- [ ] `next/font` 사용 (Google Fonts 또는 로컬 폰트)
- [ ] 폰트 `display: swap` 설정

#### JavaScript 번들 최적화

- [ ] 불필요한 클라이언트 컴포넌트 최소화 (`'use client'` 남용 제거)
- [ ] 역량 리포트 사이드 패널: `dynamic import` 적용 (지연 로드)
- [ ] `next/bundle-analyzer` 로 번들 사이즈 확인 (선택)

#### Core Web Vitals 목표값

| 지표 | 목표 | 측정 방법 |
|-----|------|---------|
| LCP | ≤ 2.5s | Lighthouse, Chrome DevTools Performance |
| FID (INP) | ≤ 100ms | Lighthouse |
| CLS | ≤ 0.1 | Lighthouse |

### 3. 접근성 검증 (Lighthouse Accessibility ≥ 90 + WCAG 2.1 AA)

#### 색상 대비

- [ ] 일반 텍스트: 4.5:1 이상
- [ ] 큰 텍스트 (18px+): 3:1 이상
- [ ] 버튼 텍스트: 4.5:1 이상
- [ ] 게이지 바 색상 (초록/노랑/빨강) vs 배경 대비 확인

#### 키보드 네비게이션

- [ ] Tab 키로 모든 인터랙티브 요소 접근 가능
- [ ] Tab 순서가 시각적 레이아웃과 일치
- [ ] 포커스 링 표시 (outline 제거 금지 또는 커스텀 포커스 스타일)
- [ ] 역량 리포트 패널 — 포커스 트랩 확인
- [ ] 역량 리포트 패널 — ESC 키로 닫기 확인

#### 스크린 리더 호환

- [ ] 게이지 바: `role="progressbar"`, `aria-valuenow` 설정 확인
- [ ] 스킬 배지: `aria-label="React - 일치"` 형식 확인
- [ ] 레벨 태그: `aria-label` 설정 확인
- [ ] 역량 리포트 패널: `role="dialog"`, `aria-modal`, `aria-labelledby` 확인
- [ ] 탭 UI: `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected` 확인
- [ ] 버튼에 aria-label (아이콘 전용 버튼)
- [ ] 이미지에 alt 텍스트

#### 기타 접근성

- [ ] 색상만으로 정보를 전달하지 않음 (텍스트/아이콘 병용 확인)
- [ ] `<label>`과 `<input>` 연결 확인 (`htmlFor`)
- [ ] 오류 메시지 `aria-live="polite"` 또는 `role="alert"`
- [ ] 스킵 네비게이션 링크 동작 확인

### 4. 반응형 레이아웃 확인

다음 뷰포트에서 모든 페이지 확인:

| 뷰포트 | 기기 유형 | 확인 항목 |
|-------|---------|---------|
| 360px | 소형 모바일 | 카드 1열, 게이지 바 정상 렌더링 |
| 414px | 대형 모바일 | 카드 1열, 필터 패널 |
| 768px | 태블릿 | 카드 2열, 사이드바 레이아웃 |
| 1280px | 데스크탑 | 카드 3열, 역량 리포트 패널 |
| 1920px | 대형 모니터 | 최대 너비 제한, 레이아웃 깨짐 없음 |

체크:
- [ ] 360px — 랜딩 페이지 정상
- [ ] 360px — 공고 목록 카드 1열
- [ ] 360px — 게이지 바 정상 렌더링
- [ ] 360px — 필터 패널 접힘/펼침
- [ ] 768px — 카드 2열 레이아웃
- [ ] 1280px — 카드 3열 레이아웃
- [ ] 1280px — 역량 리포트 사이드 패널 오버플로 없음
- [ ] 1920px — 레이아웃 깨짐 없음

### 5. HTML5 시맨틱 마크업 검증

- [ ] W3C HTML Validator (validator.w3.org) 또는 `npx html-validate` 실행
- [ ] 모든 페이지 HTML 유효성 검사 통과
- [ ] `<h1>` ~ `<h6>` 계층 구조 논리적 순서 확인 (h1 건너뜀 없음)
- [ ] `<main>` 요소 페이지당 1개
- [ ] `<nav>` 요소 `aria-label` 구분
- [ ] `<table>` 요소 사용 시 `<caption>`, `<th scope>` 설정

### 6. 에러 상태 처리

- [ ] 공고 목록 빈 상태 — "조건에 맞는 공고가 없습니다" 메시지 + 아이콘
- [ ] API 에러 상태 — "데이터를 불러올 수 없습니다. 잠시 후 다시 시도해주세요." 메시지
- [ ] 역량 리포트 로딩 상태 — 스피너/스켈레톤 UI
- [ ] 존재하지 않는 공고 ID — Next.js `notFound()` 처리 → 404 페이지
- [ ] 프로젝트 카드 최대 5개 초과 시 — 추가 버튼 비활성화 + 안내 문구
- [ ] `error.tsx` (Next.js Error Boundary) 글로벌 에러 페이지 존재

### 7. SEO 최적화 (Lighthouse SEO ≥ 90)

- [ ] 각 페이지 `<title>` 태그 고유하게 설정 (Next.js Metadata API)
- [ ] `<meta name="description">` 각 페이지 설정
- [ ] Open Graph 태그 (`og:title`, `og:description`, `og:image`)
- [ ] `robots.txt` 파일 생성
- [ ] `sitemap.xml` 생성 (선택)
- [ ] 모든 링크 `<a>` 태그에 의미있는 텍스트

### 8. Vercel 배포

- [ ] Vercel 계정 연결 및 GitHub 레포지토리 연동
- [ ] 환경변수 Vercel 대시보드에 설정
  - `NEXT_PUBLIC_APP_URL` (Vercel 배포 URL)
  - `DATABASE_URL` (빈 값으로 설정, 에러 없음 확인)
- [ ] `npm run build` Vercel에서 정상 완료
- [ ] 배포된 URL에서 모든 페이지 접근 확인
- [ ] HTTPS 자동 적용 확인
- [ ] 배포 URL 기록: ___________________

---

## Lighthouse 점수 기록

> 측정 도구: Chrome DevTools Lighthouse (Incognito 모드) 또는 `npx lighthouse`
> 측정 일시: ___________________

| 페이지 | Performance | Accessibility | Best Practices | SEO |
|-------|------------|--------------|----------------|-----|
| `/` 랜딩 | - | - | - | - |
| `/jobs` 목록 | - | - | - | - |
| `/jobs/:id` 상세 | - | - | - | - |
| `/profile` | - | - | - | - |
| `/employer` | - | - | - | - |

**목표: 모든 항목 ≥ 90**

---

## 배포 후 확인

- [ ] 배포 URL에서 공고 30건 정상 로드
- [ ] 배포 URL에서 A/B 테스트 변형 정상 분기
- [ ] 배포 URL에서 역량 리포트 패널 정상 동작
- [ ] 배포 URL에서 기업 대시보드 정상 동작
- [ ] 모바일 기기(실제 기기)에서 레이아웃 확인

---

## 에러 발생 시

에러 발생 시 즉시 `docs/errors/error-YYYYMMDD-{번호}.md` 파일을 생성하여 기록한다.

### QA 과정에서 발견된 에러 목록

| 에러 파일 | 에러 내용 요약 | 상태 |
|---------|------------|------|
| - | - | - |
