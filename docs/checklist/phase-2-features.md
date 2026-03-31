# Phase 2 체크리스트 — 차별화 기능 구현

> 작성일: 2026-03-31
> Phase: 2 | Day: 3
> 목표: F-03, F-05, F-06 기능 구현 + 기업 HR 대시보드 완성

---

## 진행 상황

| 항목 | 상태 | 완료일 |
|-----|------|-------|
| F-03 역량 리포트 사이드 패널 | ⬜ 미완료 | - |
| F-03 AI 탈락 시뮬레이션 | ⬜ 미완료 | - |
| F-05 조직문화 핏 태그 | ⬜ 미완료 | - |
| F-05 문화 태그 필터 | ⬜ 미완료 | - |
| F-06 기업 HR 대시보드 | ⬜ 미완료 | - |
| F-06 자격요건 자동 필터링 | ⬜ 미완료 | - |
| F-06 Skill-Fit Score | ⬜ 미완료 | - |
| 이벤트 트래킹 | ⬜ 미완료 | - |

> 상태 표기: ⬜ 미완료 / 🔄 진행 중 / ✅ 완료

---

## 체크리스트

### 1. F-03 역량 리포트 + AI 탈락 사유 시뮬레이션

#### API 라우트

**`src/app/api/jobs/[id]/report/route.ts`**

- [ ] `GET /api/jobs/:id/report` 구현
- [ ] Mock 응답: 합격자 평균 스킬 프로필 vs 현재 사용자 프로필 갭 데이터
  ```typescript
  {
    averageProfile: { skills: [{ name: string, avgYears: number }] },
    userProfile: { skills: [{ name: string, years: number }] },
    gaps: [{ skill: string, userYears: number, avgYears: number }],
    recommendations: string[],
  }
  ```
- [ ] 합격자 평균 데이터 없는 공고: `{ status: 'collecting', message: '데이터 수집 중' }` 반환
- [ ] 응답 시간 1초 이내 (Mock 이므로 딜레이 없이 즉시 반환)

**`src/app/api/jobs/[id]/simulate/route.ts`**

- [ ] `POST /api/jobs/:id/simulate` 구현
- [ ] 요청 바디: `{ userId: string }`
- [ ] Mock 응답: JD 키워드와 이력서 비교 결과 텍스트
  ```typescript
  {
    feedback: string,  // "이 공고는 AWS 인프라 경험을 중시하는데, 현재 이력서에는 인프라 관련 언급이 없습니다."
    missingKeywords: string[],
    confidence: 'high' | 'medium' | 'low',
    disclaimer: 'AI 추정 결과이며 실제 채용 기준과 다를 수 있습니다.',
  }
  ```
- [ ] 공고 ID별 고정 Mock 응답 5~6종 준비

#### `src/components/jobs/CompetencyReport.tsx` — 역량 리포트 사이드 패널

```tsx
interface CompetencyReportProps {
  jobId: string;
  userId: string;
  isOpen: boolean;
  onClose: () => void;
  onApply: () => void;
}
```

- [ ] `role="dialog"` + `aria-modal="true"` + `aria-labelledby` 설정
- [ ] 패널 오픈 시 포커스 트랩 구현 (Tab 키가 패널 내부 순환)
- [ ] ESC 키로 패널 닫기
- [ ] 배경 클릭으로 패널 닫기
- [ ] 패널 닫힐 때 원래 포커스 위치로 복귀

**역량 리포트 섹션**:
- [ ] `<h2>` "이 공고 역량 리포트"
- [ ] 로딩 상태: 스피너 + "리포트 불러오는 중..."
- [ ] 합격자 평균 vs 내 프로필 바 차트 (CSS 기반, SVG 또는 div)
  - 각 스킬 행: 스킬명 | 내 경력 바 | 평균 경력 바
- [ ] 갭 항목: "React — 합격자 평균 3년 / 나 1년 (갭 2년)"
- [ ] 보완 추천 목록 (`<ul>`)
- [ ] 데이터 없음 상태: "데이터 수집 중" 안내

**AI 탈락 시뮬레이션 섹션**:
- [ ] `<h3>` "AI 탈락 예측"
- [ ] "AI 추정 결과" 레이블 명시 (면책 문구)
- [ ] 피드백 텍스트 표시
- [ ] 부족 키워드 배지 목록
- [ ] 로딩 상태 처리

**하단 버튼**:
- [ ] "지원하기" 버튼 (클릭 시 `apply_complete` 이벤트 + `viewed_report=true`)
- [ ] "나중에 지원하기" 버튼 (패널 닫기)
- [ ] 지원 완료 후 확인 상태 표시

### 2. F-05 조직문화 핏 태그 시스템

#### API 라우트

**`src/app/api/jobs/[id]/culture/route.ts`**

- [ ] `GET /api/jobs/:id/culture` 구현
- [ ] Mock 응답: 문화 태그 + 재직자 평점
  ```typescript
  {
    tags: [{ id: string, label: string, rating: number | null }]
  }
  ```

#### `src/components/ui/CultureTag.tsx` — 문화 태그 컴포넌트

```tsx
interface CultureTagProps {
  tag: CultureTag;
  showRating?: boolean;
}
```

- [ ] 태그 배지 표시 (`<span>`)
- [ ] 재직자 평점 있을 경우: 마우스오버 툴팁 표시 (`title` 속성 또는 커스텀 툴팁)
  - 툴팁 내용: "재직자 평점: ★{rating}/5"
- [ ] 평점 없을 경우: 태그만 표시 (평점 섹션 없이)
- [ ] 키보드 접근 가능 (포커스 시 툴팁 표시)
- [ ] `aria-describedby` 또는 `title`로 평점 정보 접근성 제공

#### 공고 목록 필터 업데이트

- [ ] 기존 `JobFilter.tsx`에 문화 태그 필터 섹션 추가
  - `<fieldset>` + `<legend>조직문화</legend>`
  - 20개 문화 태그 풀에서 다중 선택 가능
  - Mock 데이터에서 사용된 태그만 필터 옵션으로 노출
- [ ] 문화 태그 필터 선택 시 해당 태그 보유 공고만 노출
- [ ] `culture_tag_filter` 이벤트 발송 (`tag_list` 속성)

#### 공고 상세 페이지 업데이트

- [ ] 조직문화 섹션 (`<section aria-labelledby="culture-heading">`)
- [ ] 태그 없는 공고: 조직문화 섹션 비표시 (레이아웃 깨짐 없음)
- [ ] 문화 태그 클릭 시 해당 태그로 필터된 공고 목록으로 이동

### 3. F-06 자격요건 자동 필터링 + Skill-Fit Score (기업용)

#### API 라우트

**`src/app/api/employer/filter/route.ts`**

- [ ] `POST /api/employer/filter` 구현
- [ ] 요청: `{ jobId: string, conditions: FilterCondition }`
- [ ] Mock 응답: 지원자를 3단계로 분류한 결과
  ```typescript
  {
    suitable: CandidateApplication[],
    review: CandidateApplication[],
    unqualified: CandidateApplication[],
  }
  ```
- [ ] 분류 로직:
  - `suitable`: 필수 조건 100% 충족
  - `review`: 필수 조건 70~99% 충족
  - `unqualified`: 필수 조건 70% 미만 충족

**`src/app/api/employer/skill-fit/route.ts`**

- [ ] `POST /api/employer/skill-fit` 구현
- [ ] 요청: `{ candidateId: string, jobId: string }`
- [ ] Mock 응답:
  ```typescript
  {
    skillFitScore: number,       // 0~100
    summary: string,             // "프로젝트 A에서 React 3년 사용 확인됨"
    evidence: string[],          // 근거 항목들
  }
  ```

#### 기업 HR 대시보드 (`src/app/employer/page.tsx`)

- [ ] `<h1>` "채용 대시보드"
- [ ] 공고 선택 드롭다운 (`<select>`)
- [ ] 지원자 집계 현황 (전체/적합/검토필요/미충족 수)

#### 지원자 탭 UI (`src/components/employer/CandidateList.tsx`)

```tsx
interface CandidateListProps {
  candidates: CandidateApplication[];
  category: FilterCategory;
}
```

- [ ] 탭 네비게이션 (`<nav role="tablist">`)
  - 탭 1: "적합 후보 (8)" — 기본 활성
  - 탭 2: "검토 필요 (7)"
  - 탭 3: "조건 미충족 (5)" — 기본 접힘
- [ ] 각 탭 패널 `role="tabpanel"` + `aria-labelledby`
- [ ] 조건 미충족 탭: 기본 접힘, "수동 열람" 버튼으로 토글
- [ ] 지원자 카드: 이름, 경력, 스킬, Skill-Fit Score 표시

#### `src/components/employer/SkillFitScore.tsx`

```tsx
interface SkillFitScoreProps {
  score: number;
  summary: string;
  evidence: string[];
}
```

- [ ] 0~100 점수 표시
- [ ] 점수 근거 요약 텍스트 표시
- [ ] "근거 보기" 토글로 세부 evidence 목록 표시
- [ ] 수동 재분류 버튼 ("적합으로 변경") — `filterCategory` 업데이트

### 4. 이벤트 트래킹

`src/lib/abTest.ts`의 `trackEvent` 함수 업데이트:

- [ ] `job_card_view` — 카드 뷰포트 진입 시 (`job_id`, `score`, `ab_variant`)
- [ ] `job_card_click` — 카드 클릭 시 (`job_id`, `score`, `ab_variant`)
- [ ] `apply_start` — "지원하기" 클릭 시 (`job_id`, `viewed_report`)
- [ ] `apply_complete` — 지원 완료 시 (`job_id`, `time_spent`)
- [ ] `report_viewed` — 역량 리포트 열람 시 (`job_id`, `gap_count`)
- [ ] `ai_simulation_viewed` — AI 시뮬레이션 조회 시 (`job_id`, `gap_keywords`)
- [ ] `culture_tag_filter` — 문화 태그 필터 적용 시 (`tag_list`)
- [ ] `skill_fit_score_viewed` — HR이 Skill-Fit Score 조회 시 (`candidate_id`, `score`)
- [ ] MVP: 모든 이벤트는 `console.log` + `localStorage` 저장 (실제 분석 서버 없음)

---

## 완료 기준

- [ ] F-03 역량 리포트 패널 — "지원하기" 클릭 시 1초 이내 로드
- [ ] F-03 데이터 없음 공고 — "데이터 수집 중" 정상 표시
- [ ] F-03 AI 시뮬레이션 — "AI 추정 결과" 레이블 포함 출력
- [ ] F-03 AC 체크: `docs/checklist/feature-acceptance.md` → F-03 섹션
- [ ] F-05 문화 태그 없는 공고 — 공고 상세 정상 표시
- [ ] F-05 태그 클릭 툴팁 — 재직자 평점 요약 표시
- [ ] F-05 AC 체크: `docs/checklist/feature-acceptance.md` → F-05 섹션
- [ ] F-06 3단계 분류 탭 — 각 탭 지원자 수 실시간 표시
- [ ] F-06 조건 미충족 탭 — 기본 접힘 확인
- [ ] F-06 수동 재분류 — 지원자 카테고리 변경 가능
- [ ] F-06 AC 체크: `docs/checklist/feature-acceptance.md` → F-06 섹션
- [ ] 이벤트 트래킹 9개 — 브라우저 콘솔에서 발송 확인

---

## 에러 발생 시

에러 발생 시 즉시 `docs/errors/error-YYYYMMDD-{번호}.md` 파일을 생성하여 기록한다.
