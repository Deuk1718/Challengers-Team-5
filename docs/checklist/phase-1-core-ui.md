# Phase 1 체크리스트 — 핵심 공고 목록 UI

> 작성일: 2026-03-31
> Phase: 1 | Day: 1 후반 ~ Day 2
> 목표: 구직자 공고 탐색 핵심 기능 (F-01, F-02, F-04) + 기본 페이지 구조 완성

---

## 진행 상황

| 항목 | 상태 | 완료일 |
|-----|------|-------|
| 공통 레이아웃 | ⬜ 미완료 | - |
| 랜딩 페이지 | ⬜ 미완료 | - |
| 공고 목록 페이지 | ⬜ 미완료 | - |
| F-01 매칭 스코어 카드 | ⬜ 미완료 | - |
| F-02 경력 레벨 자동 태그 | ⬜ 미완료 | - |
| F-04 포트폴리오 프로젝트 태그 | ⬜ 미완료 | - |
| A/B 테스트 E1 로직 | ⬜ 미완료 | - |

> 상태 표기: ⬜ 미완료 / 🔄 진행 중 / ✅ 완료

---

## 체크리스트

### 1. 공통 레이아웃 (`src/app/layout.tsx`)

HTML5 시맨틱 마크업 필수:

- [ ] `<html lang="ko">` 언어 속성 설정
- [ ] `<head>` 메타태그 (charset, viewport, description, og:title)
- [ ] `<body>` 내 `<header>`, `<main>`, `<footer>` 구조
- [ ] `<header>` 내 `<nav aria-label="주 네비게이션">` 구현
- [ ] 스킵 네비게이션 링크 (`<a href="#main-content">본문으로 이동</a>`)
- [ ] `<main id="main-content">` 속성 설정
- [ ] 반응형 네비게이션 (모바일 햄버거 메뉴 또는 단순화)
- [ ] Tailwind 글로벌 스타일 적용

### 2. 랜딩 페이지 (`src/app/page.tsx`)

- [ ] `<main>` 내 서비스 소개 섹션 (`<section>`)
- [ ] 슬로건 "나의 경험을 자산으로, 기업의 채용을 확신으로" 표시
- [ ] 구직자용 CTA 버튼 ("공고 탐색하기" → `/jobs`)
- [ ] 기업용 CTA 버튼 ("채용 관리하기" → `/employer`)
- [ ] 3가지 핵심 가치 카드 섹션

### 3. 공고 목록 페이지 (`src/app/jobs/page.tsx`)

- [ ] `<h1>` 페이지 제목 ("추천 공고")
- [ ] 필터 영역 `<aside aria-label="공고 필터">` 또는 `<section>`
- [ ] 공고 목록 `<ul role="list" aria-label="공고 목록">`
- [ ] 각 공고 카드 `<li>` > `<article aria-labelledby="job-{id}">` 구조
- [ ] Mock 데이터 30건 목록 정상 렌더링
- [ ] 360px 모바일에서 카드 1열 레이아웃
- [ ] 768px 태블릿에서 카드 2열 레이아웃
- [ ] 1280px+ 데스크탑에서 카드 3열 레이아웃

### 4. 공고 상세 페이지 (`src/app/jobs/[id]/page.tsx`)

- [ ] `<article>` 내 공고 상세 정보 구조
- [ ] `<h1>` 공고 제목
- [ ] 회사명, 위치, 고용형태, 연봉 범위 표시
- [ ] 요구 스킬 목록 (`<ul>`)
- [ ] 공고 설명 (`<section>`)
- [ ] 지원하기 버튼 (F-03 역량 리포트 트리거, Phase 2에서 구현)
- [ ] 존재하지 않는 ID 접근 시 404 처리

### 5. F-01 매칭 스코어 카드

#### `src/lib/scoring.ts` — 스코어 계산 로직

- [ ] `calculateMatchScore(userProfile, jobPosting): number` 함수 작성
  - 스킬 일치 가중치: 일치 스킬 수 / 전체 요구 스킬 수 × 100
  - 경력 연수 보정: 요구 경력 범위 내 ±10% 가산
  - 프로젝트 카드 보정: 카드 1개당 +5점 (최대 +25)
- [ ] `getMatchedSkills(userProfile, jobPosting): Skill[]` 함수
- [ ] `getMissingSkills(userProfile, jobPosting): Skill[]` 함수
- [ ] 반환 범위 0~100 클램핑 처리

#### `src/components/ui/GaugeBar.tsx` — 게이지 바 컴포넌트

```tsx
interface GaugeBarProps {
  score: number;        // 0~100
  showLabel?: boolean;
}
```

- [ ] `role="progressbar"` 접근성 속성 설정
- [ ] `aria-valuenow`, `aria-valuemin="0"`, `aria-valuemax="100"` 설정
- [ ] `aria-label="매칭 스코어 {score}%"` 설정
- [ ] 점수에 따른 색상 변화 (0~59: 빨강, 60~79: 노랑, 80~100: 초록)
- [ ] CSS transition으로 부드러운 애니메이션
- [ ] 모바일에서 정상 렌더링 확인

#### `src/components/ui/SkillBadge.tsx` — 스킬 배지 컴포넌트

```tsx
interface SkillBadgeProps {
  skill: string;
  matched: boolean;    // true: 일치(초록), false: 부족(빨강)
}
```

- [ ] 일치 스킬: 초록 배지 + 체크 아이콘 (`✓`)
- [ ] 부족 스킬: 빨간 배지 + X 아이콘 (`✗`)
- [ ] 색상만으로 구분하지 않음 — 텍스트 레이블 또는 아이콘 병용 (접근성)
- [ ] `aria-label="{skill} - {일치/부족}"` 설정

#### `src/components/jobs/JobCard.tsx` — 공고 카드

```tsx
interface JobCardProps {
  job: JobPosting;
  candidateScore: CandidateScore;
  abVariant: ABVariant;
}
```

- [ ] `<article>` 태그 사용
- [ ] `<h2>` 공고 제목 (`id="job-{job.id}"`)
- [ ] 회사명, 위치, 고용형태, 연봉 표시
- [ ] A 변형 (abVariant === 'A'): 숫자만 표시 `"82%"`
- [ ] B 변형 (abVariant === 'B'): `GaugeBar` + `SkillBadge` 목록 표시
- [ ] "이 공고는 당신에게 {score}% 맞습니다." 1줄 요약 (B 변형)
- [ ] "레벨 불일치" 경고 배지 표시 (F-02 연동)
- [ ] 저장 버튼 (아이콘 버튼, `aria-label="공고 저장"`)
- [ ] 카드 클릭 시 공고 상세로 이동

#### A/B 테스트 E1 (`src/lib/abTest.ts`)

- [ ] `getABVariant(userId: string): ABVariant` 함수 작성
  - `userId` 문자열 해시 기반 50:50 분배
  - `userId % 2 === 0 ? 'A' : 'B'` 방식 또는 해시 함수
- [ ] `trackEvent(eventName: string, properties: object)` 함수 (console.log로 MVP 구현)
  - 이벤트: `job_card_view`, `job_card_click`, `apply_start`
  - 속성: `job_id`, `score`, `ab_variant` 포함
- [ ] A 변형 vs B 변형 분기 테스트 확인

### 6. F-02 경력 레벨 자동 태그

#### `src/lib/levelTag.ts` — 레벨 태그 로직

- [ ] `getLevelTag(requiredExperienceMin: number): LevelTag` 함수
  - 0년 = `'entry'` (신입)
  - 1~2년 = `'junior'` (주니어)
  - 3~6년 = `'middle'` (미들)
  - 7년+ = `'senior'` (시니어)
- [ ] `getLevelTagLabel(levelTag: LevelTag): string` 함수
  - `'entry'` → `'신입'`
  - `'junior'` → `'주니어 1~3년'`
  - `'middle'` → `'미들 3~7년'`
  - `'senior'` → `'시니어 7+'`
- [ ] `isLevelMismatch(userLevel: LevelTag, jobLevel: LevelTag): boolean` 함수
  - 1단계 이상 차이 시 `true` 반환

#### `src/components/ui/LevelTag.tsx` — 레벨 태그 컴포넌트

- [ ] 레벨별 색상 구분 (신입: 파랑, 주니어: 초록, 미들: 노랑, 시니어: 빨강)
- [ ] `aria-label="경력 레벨: {levelLabel}"` 설정

#### 필터 패널 (`src/components/jobs/JobFilter.tsx`)

- [ ] 레벨 필터 체크박스 (중복 선택 가능)
  - `<fieldset>` + `<legend>경력 레벨</legend>` 구조
  - 신입, 주니어, 미들, 시니어 옵션
- [ ] 필터 선택 시 해당 레벨 공고만 노출

### 7. F-04 포트폴리오 프로젝트 태그 입력

#### 프로필 페이지 (`src/app/profile/page.tsx`)

- [ ] 사용자 기본 정보 섹션 (`<section>`)
- [ ] "프로젝트 카드" 섹션 (`<section aria-labelledby="projects-heading">`)
- [ ] `<h2 id="projects-heading">포트폴리오 프로젝트</h2>`

#### `src/components/profile/ProjectCardForm.tsx`

```tsx
interface ProjectCardFormProps {
  onSubmit: (card: Omit<ProjectCard, 'id'>) => void;
  initialData?: ProjectCard;
}
```

- [ ] 기술 스택 입력 (태그 형식, 콤마 구분)
- [ ] 역할 선택 (`<select>`: 리드/팀원/단독)
- [ ] 팀 규모 입력 (`<input type="number">`)
- [ ] 기간 입력 (개월 수, `<input type="number">`)
- [ ] URL 입력 (옵션, `<input type="url">`)
- [ ] 설명 입력 (옵션, `<textarea>`)
- [ ] 최대 5개 제한 (5개 초과 시 추가 버튼 비활성화 + 안내 문구)
- [ ] 카드 수정 기능 (편집 버튼)
- [ ] 카드 삭제 기능 (삭제 버튼, 확인 없이 즉시 삭제)
- [ ] 모든 필드에 `<label>` + `htmlFor` 연결
- [ ] 필수 필드 `required` 속성 + 유효성 검사 메시지

---

## 완료 기준

- [ ] `npm run dev` 에러 없음
- [ ] `/` 랜딩 페이지 정상 렌더링
- [ ] `/jobs` 공고 목록 30건 정상 렌더링
- [ ] `/jobs/:id` 공고 상세 정상 렌더링
- [ ] `/profile` 프로젝트 카드 입력/수정/삭제 동작
- [ ] F-01 AC 체크: `docs/checklist/feature-acceptance.md` → F-01 섹션
- [ ] F-02 AC 체크: `docs/checklist/feature-acceptance.md` → F-02 섹션
- [ ] F-04 AC 체크: `docs/checklist/feature-acceptance.md` → F-04 섹션
- [ ] 360px 모바일 레이아웃 정상 확인
- [ ] 키보드 네비게이션 (Tab 키) 모든 인터랙티브 요소 접근 가능
- [ ] A/B 테스트 E1 — A 변형(숫자)과 B 변형(게이지+배지) 분기 확인

---

## 에러 발생 시

에러 발생 시 즉시 `docs/errors/error-YYYYMMDD-{번호}.md` 파일을 생성하여 기록한다.
