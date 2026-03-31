# 과업 2. 추천 시스템 개선

> 담당: 팀원 B
> 참고 서비스: [커리어데이](https://www.careerday.jobs)
> 개발 기간: 4일 MVP — Day 2까지 추천 엔진 + 핵심 UI 완성 목표

---

## 1. 개요 및 목적

### 1.1 과업 목적
지원자와 전문가에게 최적의 직무·프로젝트를 추천하는 시스템을 설계합니다. 단순 키워드 매칭에서 벗어나 개인화된 추천 알고리즘을 기획하고, 이를 React + TypeScript 컴포넌트로 구현할 수 있는 설계안을 도출합니다.

### 1.2 과업 범위
- 현행 추천 방식 분석 및 한계 파악
- 3가지 추천 전략 설계
- 개인 맞춤형 추천 로직 기획
- UI 와이어프레임 및 컴포넌트 설계

---

## 2. 현행 추천 방식 분석

### 2.1 커리어데이 추천 구조 분석 (관찰 기반)

커리어데이의 공개된 UX 흐름을 분석한 결과, 다음과 같은 추천 패턴이 관찰됩니다:
- 프로필 등록 시 스킬 태그 선택 → 관련 공고 노출
- 직무 카테고리 필터링 우선 → 스킬 매칭
- 기업이 인재를 역방향으로 검색하는 구조 (재능마켓 모델)

### 2.2 현행 방식의 한계

| 한계 | 설명 | 개선 방향 |
|------|------|---------|
| 단순 키워드 매칭 | 태그 완전 일치에 의존, 유사 스킬 미인식 | 벡터 유사도 기반으로 전환 |
| Cold Start 문제 | 신규 사용자/공고의 추천 품질 저하 | 콘텐츠 기반 초기 추천 보완 |
| 인기 편향 | 조회수 높은 공고에 집중 노출 | 롱테일 추천 균형 설계 |
| 맥락 미반영 | 현재 구직 의향, 긴급도 미고려 | 세션 컨텍스트 추가 |

### 2.3 유사 서비스 추천 수준 비교

| 항목 | 커리어데이 (현행 추정) | LinkedIn | Wanted |
|------|---------------------|----------|--------|
| 추천 방식 | 규칙 기반 필터 | 딥러닝 하이브리드 | AI 스코어링 |
| 개인화 수준 | 낮음 | 매우 높음 | 높음 |
| 실시간 업데이트 | 제한적 | 실시간 | 실시간 |
| 설명 가능성 | 낮음 | 낮음 | 중간 (합격 예측) |

---

## 3. 개선 방향 — 3가지 추천 전략

### 3.1 전략 1: 콘텐츠 기반 필터링 (Content-Based Filtering)

**원리**: 사용자 프로필 속성과 공고 속성 간의 특징 벡터 유사도 계산

**적합한 상황**: 신규 사용자 (행동 데이터 부족 시 Cold Start 해결)

**핵심 로직**:
```
유사도(user, job) = cosine_similarity(user_vector, job_vector)

user_vector  = [skills_embedding | experience_score | location_score | salary_fit]
job_vector   = [skills_embedding | required_exp    | location_code  | salary_range]
```

**피처 가중치 (초기 설정)**:
| 피처 | 가중치 | 근거 |
|------|-------|------|
| 스킬 매칭 점수 | 0.45 | 직무 적합성의 핵심 요소 |
| 경력 연수 적합도 | 0.25 | 경력 미스매치 방지 |
| 급여 적합도 | 0.15 | 협상 가능성 반영 |
| 근무지 일치 | 0.10 | 원격/재택 선호 반영 |
| 직무 카테고리 | 0.05 | 대분류 일치 보정 |

### 3.2 전략 2: 협업 필터링 (Collaborative Filtering)

**원리**: "나와 비슷한 행동 패턴을 가진 사용자들이 지원한 공고"를 추천

**적합한 상황**: 행동 데이터가 충분히 쌓인 기존 사용자

**유사 사용자 결정 방법**:
```
사용자-공고 상호작용 행렬 (User-Item Matrix)

        공고1  공고2  공고3  공고4
User A   1      1      0      1
User B   1      0      1      0
User C   0      1      1      1
User D   1      1      0      0   ← 타겟 사용자

User D ↔ User A: cosine_similarity = 높음
→ User A가 지원한 공고3 추천
```

**Cold Start 해결**: 행동 데이터 5건 이상 축적 전까지 콘텐츠 기반으로 대체

### 3.3 전략 3: 하이브리드 앙상블 (최종 채택)

**원리**: 두 전략의 점수를 가중 합산하여 최종 추천 순위 결정

```
final_score = α × content_score + β × collaborative_score + γ × popularity_boost

초기 파라미터:
  α = 0.6  (콘텐츠 기반 - 설명 가능성 높음)
  β = 0.3  (협업 필터링 - 개인화)
  γ = 0.1  (인기도 보정 - 다양성 확보)
```

**단계별 전환**:
```
행동 이력 0~5건   → 콘텐츠 기반 100%
행동 이력 6~20건  → 콘텐츠 70% + 협업 30%
행동 이력 21건+   → 콘텐츠 60% + 협업 30% + 인기도 10%
```

---

## 4. 알고리즘 설계

### 4.1 입력 피처 정의

**구직자 피처 벡터**

| 피처 | 데이터 타입 | 변환 방식 | 차원 |
|------|-----------|---------|------|
| skills | string[] | TF-IDF 또는 Word2Vec 임베딩 | 100d |
| experience_years | number | Min-Max 정규화 (0~1) | 1d |
| job_category | string | One-hot 인코딩 | 20d |
| location_preference | string | 원핫 + "원격" 플래그 | 10d |
| salary_range | { min, max } | 중앙값 정규화 | 1d |
| activity_score | number | 그대로 사용 | 1d |

**공고 피처 벡터**

| 피처 | 데이터 타입 | 변환 방식 | 차원 |
|------|-----------|---------|------|
| required_skills | string[] | 동일 임베딩 공간 | 100d |
| preferred_skills | string[] | 보너스 점수로 처리 | - |
| experience_level | string | 서열 인코딩 (0~4) | 1d |
| job_type | string | 원핫 | 3d |
| industry | string | 원핫 | 15d |
| salary_offered | range | 중앙값 정규화 | 1d |

### 4.2 유사도 계산 공식

**코사인 유사도**:
```
cos(θ) = (A · B) / (‖A‖ × ‖B‖)

A = 사용자 피처 벡터
B = 공고 피처 벡터
결과: 0.0 (완전 불일치) ~ 1.0 (완전 일치)
```

**스킬 재킹 유사도 (스킬 태그 특화)**:
```
skill_score = |user_skills ∩ required_skills| / |required_skills|
            + 0.5 × |user_skills ∩ preferred_skills| / |preferred_skills|
```

### 4.3 추천 점수 산출 로직 (의사코드)

```typescript
function calculateRecommendationScore(
  user: UserProfile,
  job: JobPosting,
  userHistory: UserEvent[]
): number {
  // 1. 콘텐츠 기반 점수
  const userVector = buildUserVector(user);
  const jobVector = buildJobVector(job);
  const contentScore = cosineSimilarity(userVector, jobVector);

  // 2. 스킬 특화 점수
  const skillScore = calculateSkillScore(user.skills, job.requiredSkills, job.preferredSkills);

  // 3. 협업 필터링 점수 (이력이 충분할 경우)
  const collaborativeScore = userHistory.length >= 6
    ? calculateCollaborativeScore(user.userId, job.jobId)
    : 0;

  // 4. 가중치 결정 (행동 이력 양에 따라)
  const weights = determineWeights(userHistory.length);

  // 5. 최종 점수 합산
  const finalScore =
    weights.content * contentScore +
    weights.skill * skillScore +
    weights.collaborative * collaborativeScore;

  // 6. 0~100 스케일로 변환
  return Math.round(finalScore * 100);
}
```

---

## 5. 개인 맞춤형 추천 기획

### 5.1 지원자 추천 (구직자 → 공고)

**추천 기준 우선순위**:
1. 스킬 매칭도 (필수 스킬 충족률 > 80%)
2. 경력 수준 적합도 (요구 경력 ±1년 허용)
3. 희망 급여와 제시 급여 교집합 존재
4. 선호 근무지 일치 (원격 여부 포함)

**추천 카드 표시 정보**:
- 직무명, 기업명
- 매칭 스코어 (0~100)
- 스킬 일치 태그 (강조 표시)
- 스킬 부족 태그 (보완 필요 표시)

### 5.2 전문가/멘토 추천 (기업 → 전문가)

**추천 기준**:
- 요구 스킬과 전문가 스킬 코사인 유사도
- 포트폴리오 관련 프로젝트 수
- 업종·직무 카테고리 연관도
- 가용 여부 (현재 프로젝트 참여 수 < 최대 3개)

### 5.3 프로젝트 추천 (프리랜서/전문가 전용)

**추천 기준**:
- 관심 태그와 프로젝트 태그 교집합
- 프로젝트 기간 선호도 (단기/장기)
- 선호 기술 스택 일치
- 이전 유사 프로젝트 완료 이력

### 5.4 추천 다양성 보장 (탐색-활용 균형)

```
추천 목록 구성:
- 상위 70%: 높은 매칭 스코어 공고 (활용, Exploitation)
- 하위 30%: 스코어 약간 낮지만 다양성 높은 공고 (탐색, Exploration)

탐색 공고 선정 기준:
- 사용자가 아직 조회하지 않은 직무 카테고리
- 스킬 확장 가능성이 있는 공고 (부족 스킬 1~2개)
```

---

## 6. UI 와이어프레임

### 6.1 추천 공고 목록 페이지

**모바일 (< 768px)**
```
┌─────────────────────────┐
│  추천 공고              │
│  [필터] [정렬 ▼]        │
├─────────────────────────┤
│ ┌─────────────────────┐ │
│ │ [회사로고]  매칭 92% │ │
│ │ 프론트엔드 개발자    │ │
│ │ (주)커리어데이       │ │
│ │ React TypeScript    │ │
│ │ 서울 강남 · 정규직   │ │
│ │ 연봉 5,000~7,000만  │ │
│ │      [지원하기]      │ │
│ └─────────────────────┘ │
├─────────────────────────┤
│ ┌─────────────────────┐ │
│ │ [회사로고]  매칭 85% │ │
│ │ ...                  │ │
│ └─────────────────────┘ │
└─────────────────────────┘
```

**데스크탑 (≥ 1024px)**
```
┌──────────────┬──────────────────────────────────────────────────┐
│ 필터 패널    │  추천 공고 (32건)              [정렬 ▼] [그리드/리스트] │
│              ├─────────────────────────────────────────────────-┤
│ 직무 카테고리 │ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ ☑ 프론트엔드 │ │ 매칭 92%    │ │ 매칭 87%    │ │ 매칭 81%    │ │
│ ☐ 백엔드     │ │ 프론트엔드   │ │ 풀스택 개발자│ │ React 개발자 │ │
│ ☐ 데이터     │ │ (주)커리어A  │ │ (주)커리어B  │ │ 스타트업C    │ │
│              │ │ React TS    │ │ React Node  │ │ Next.js      │ │
│ 경력         │ │ 서울 · 정규직│ │ 원격 · 정규직│ │ 분당 · 정규직│ │
│ ○ 신입       │ └──────────────┘ └──────────────┘ └──────────────┘ │
│ ● 3~5년      │                                                    │
│ ○ 5년 이상   │ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│              │ │ 매칭 78%    │ │ ...          │ │ ...          │ │
│ 근무 형태    │ └──────────────┘ └──────────────┘ └──────────────┘ │
│ ☑ 정규직     │                                                    │
│ ☐ 프리랜서   │                                        [더 보기]    │
└──────────────┴─────────────────────────────────────────────────--┘
```

### 6.2 매칭 스코어 컴포넌트

```
┌──────────────────────────────┐
│  매칭 점수: 92/100           │
│  ████████████████████░░ 92% │
│                              │
│  일치 스킬 (4/5)             │
│  [React ✓] [TypeScript ✓]   │
│  [Next.js ✓] [Tailwind ✓]   │
│                              │
│  보완 필요 (1개)              │
│  [GraphQL ✗]                │
│                              │
│  경력 적합 ✓ (3년/3~5년 요구) │
│  급여 적합 ✓ (범위 일치)      │
└──────────────────────────────┘
```

---

## 7. 구현 컴포넌트 설계

### 7.1 추천 엔진 인터페이스

```typescript
// lib/recommendation/types.ts
interface RecommendationInput {
  userProfile: UserProfile;
  userHistory: UserEvent[];
  limit?: number;         // 기본: 20
  page?: number;
}

interface RecommendationResult {
  jobId: string;
  score: number;          // 0~100
  matchedSkills: string[];
  missingSkills: string[];
  reasons: string[];      // 설명 가능한 추천 이유
}

interface RecommendationEngine {
  recommend(input: RecommendationInput): Promise<RecommendationResult[]>;
  getScore(userId: string, jobId: string): Promise<number>;
}
```

### 7.2 컴포넌트 스펙

**`JobCard` 컴포넌트**

```typescript
// components/cards/JobCard.tsx
interface JobCardProps {
  job: JobPosting;
  score: number;
  matchedSkills: string[];
  missingSkills: string[];
  onApply: (jobId: string) => void;
  onBookmark: (jobId: string) => void;
}
```

**`MatchScore` 컴포넌트**

```typescript
// components/ui/MatchScore.tsx
interface MatchScoreProps {
  score: number;           // 0~100
  matchedSkills: string[];
  missingSkills: string[];
  variant?: 'compact' | 'detail'; // compact: 숫자만, detail: 전체 분석
}
```

**`RecommendList` 컴포넌트**

```typescript
// components/recommend/RecommendList.tsx
interface RecommendListProps {
  userId: string;
  layout?: 'grid' | 'list';
  filters?: FilterState;
  onFilterChange?: (filters: FilterState) => void;
}
// React Query 기반 무한 스크롤, 페이지당 20개
```

---

## 8. MVP 산출물 (Deliverables)

> 4일 일정 기준 — 협업 필터링·하이브리드는 Day 4 Nice to Have로 이동.

| 산출물 | 형태 | 목표 완성 시점 | 우선순위 |
|-------|------|-------------|---------|
| 추천 엔진 타입 정의 | `lib/recommendation/types.ts` | Day 1 오전 | Must |
| 콘텐츠 기반 필터링 구현 | `lib/recommendation/contentBased.ts` (스킬 코사인 유사도) | Day 2 오전 | Must |
| `JobCard` 컴포넌트 | `components/cards/JobCard.tsx` | Day 2 오전 | Must |
| `MatchScore` 컴포넌트 | `components/ui/MatchScore.tsx` | Day 2 오전 | Must |
| `RecommendList` 컴포넌트 | `components/recommend/RecommendList.tsx` | Day 2 오후 | Must |
| 추천 API 엔드포인트 | `app/api/recommend/route.ts` | Day 2 오후 | Must |
| 추천 목록 페이지 반응형 통합 | `app/page.tsx` + FilterPanel 연결 | Day 3 오전 | Must |
| 하이브리드 앙상블 구현 | `lib/recommendation/hybrid.ts` | Day 4 여유 시 | Nice |
