# 과업 3. A/B 테스트 및 성능 최적화

> 담당: 팀원 C
> 참고 서비스: [커리어데이](https://www.careerday.jobs)
> 개발 기간: 4일 MVP — Day 3까지 A/B 로직 1개 시나리오 구현 + 성능 목표 달성

---

## 1. 개요 및 목적

### 1.1 과업 목적
추천 시스템의 UI/UX 변경사항이 실제 사용자 행동에 미치는 영향을 데이터로 검증합니다. A/B 테스트 프레임워크를 구축하고, 통계적으로 유의미한 결과를 기반으로 의사결정하는 체계를 설계합니다. 또한 Next.js 기반 반응형 웹의 Core Web Vitals를 목표 수준까지 최적화합니다.

### 1.2 과업 범위
- A/B 테스트 설계 방법론 수립
- 5개 테스트 시나리오 상세 설계
- 결과 분석 프레임워크 구축
- Next.js 성능 최적화 계획 수립

---

## 2. A/B 테스트 설계 방법론

### 2.1 가설 설정 템플릿

모든 실험은 아래 구조로 가설을 수립합니다:

```
현상:    [현재 관찰되는 문제 또는 기회]
원인 가설: [왜 이 현상이 발생하는지]
실험 설계: [무엇을 바꿀 것인지]
기대 효과: [어떤 지표가 얼마나 개선될 것인지]
성공 기준: [어떤 수치가 나와야 배포를 결정하는지]
```

**예시**:
```
현상:    추천 공고 클릭률이 6.5%로 업계 평균(8%) 대비 낮음
원인 가설: 추천 카드의 매칭 스코어 표시 방식이 사용자에게 직관적이지 않음
실험 설계: A(숫자 92%) vs B(게이지 바 + 92%) 표시 방식 비교
기대 효과: CTR을 6.5% → 8.5% 이상으로 향상
성공 기준: p-value < 0.05 AND Δ CTR ≥ +15% (상대적)
```

### 2.2 실험군/대조군 분리 기준

```
사용자 ID → SHA-256 해시 → 마지막 2자리 → 실험 할당

예시:
  user_id = "abc123"
  hash = sha256("abc123") = "a3f9..."
  마지막 2자리 = hex 값 → 0~255
  0~127 (50%) → 대조군 (A)
  128~255 (50%) → 실험군 (B)
```

**세그먼트 보장 조건**:
| 조건 | 이유 |
|------|------|
| 같은 사용자는 항상 같은 그룹 | 일관된 경험 보장 |
| 신규/기존 사용자 균등 분포 | 편향 방지 |
| 기기 유형(모바일/데스크탑) 균형 | 기기 효과 통제 |

### 2.3 최소 샘플 사이즈 계산

```
n = 2 × (z_α/2 + z_β)² × p(1-p) / (MDE)²

변수 정의:
  z_α/2 = 1.96 (유의수준 5%, 양측 검정)
  z_β   = 0.84 (검정력 80%)
  p     = 기준 전환율 (예: CTR 0.065)
  MDE   = 최소 감지 효과 크기 (예: 절대 +0.01 = 1%p)

계산 예시 (CTR 6.5% → 7.5%, 1%p 향상 감지):
  n ≈ 2 × (1.96+0.84)² × 0.065×0.935 / (0.01)²
  n ≈ 3,167명 (그룹당)
  총 필요 샘플: ~6,334명
```

---

## 3. A/B 테스트 시나리오 목록

> **MVP 적용 범위**: 4일 일정상 시나리오 2 (매칭 스코어 시각화)를 **1순위 구현 대상**으로 선정합니다. 나머지 시나리오는 설계 문서로만 보존하며 추후 구현합니다.

### 시나리오 1: 추천 카드 정렬 알고리즘

| 항목 | 내용 |
|------|------|
| **가설** | 매칭 점수 + 최신성 복합 정렬이 순수 점수 정렬보다 CTR 향상 |
| **대조군 A** | 매칭 스코어 내림차순 정렬 |
| **실험군 B** | 스코어 70% + 최신 공고 30% 가중 정렬 |
| **주요 지표** | CTR (클릭률), 지원 전환율 |
| **보조 지표** | 세션 체류 시간, 스크롤 깊이 |
| **실험 기간** | 2주 |
| **최소 샘플** | 그룹당 3,000명 |
| **성공 기준** | CTR +10% 이상 (상대), p < 0.05 |

### 시나리오 2: 매칭 스코어 시각화 방식

| 항목 | 내용 |
|------|------|
| **가설** | 게이지 바로 스코어를 시각화하면 지원 전환율 향상 |
| **대조군 A** | 텍스트 숫자 표시 ("매칭 92%") |
| **실험군 B** | 컬러 게이지 바 + 숫자 + 스킬 분석 요약 |
| **주요 지표** | 지원 전환율 (CTR → Apply) |
| **보조 지표** | 공고 상세 페이지 체류 시간 |
| **실험 기간** | 2주 |
| **최소 샘플** | 그룹당 2,500명 |
| **성공 기준** | 지원 전환율 +15% 이상 (상대), p < 0.05 |

### 시나리오 3: 필터 패널 위치

| 항목 | 내용 |
|------|------|
| **가설** | 상단 가로 필터가 사이드바보다 모바일 사용성 향상 |
| **대조군 A** | 사이드바 필터 (데스크탑 기준) |
| **실험군 B** | 상단 가로 슬라이드 필터 (모바일/데스크탑 통일) |
| **주요 지표** | 필터 사용률, 세션 체류 시간 |
| **보조 지표** | 필터 적용 후 CTR |
| **실험 기간** | 1주 |
| **최소 샘플** | 그룹당 1,500명 (모바일 사용자 한정) |
| **성공 기준** | 필터 사용률 +20% 이상, p < 0.05 |

### 시나리오 4: 맞춤 알림 빈도 최적화

| 항목 | 내용 |
|------|------|
| **가설** | 일 1회 알림보다 주 3회 알림이 스팸 인식 없이 재방문율 향상 |
| **대조군 A** | 새 공고 알림: 일 1회 (오전 9시) |
| **실험군 B** | 새 공고 알림: 주 3회 (월/수/금 오전 9시) |
| **주요 지표** | 알림 클릭률 (Open Rate), 7일 재방문율 |
| **보조 지표** | 알림 수신 거부율 (Unsubscribe Rate) |
| **실험 기간** | 3주 |
| **최소 샘플** | 그룹당 1,000명 (알림 수신 동의 사용자) |
| **성공 기준** | 재방문율 +10% AND 수신거부율 +3% 이내, p < 0.05 |

### 시나리오 5: 온보딩 플로우 단계 수 최적화

| 항목 | 내용 |
|------|------|
| **가설** | 온보딩 5단계를 3단계로 줄이면 완료율 향상 |
| **대조군 A** | 5단계 온보딩 (기본정보 → 스킬 → 경력 → 희망조건 → 포트폴리오) |
| **실험군 B** | 3단계 온보딩 (기본정보+스킬 통합 → 경력+희망조건 통합 → 포트폴리오) |
| **주요 지표** | 온보딩 완료율, 완료 소요 시간 |
| **보조 지표** | 각 단계별 이탈률 |
| **실험 기간** | 1주 |
| **최소 샘플** | 그룹당 500명 (신규 가입자) |
| **성공 기준** | 완료율 +20% 이상, 소요 시간 -25% 이상, p < 0.05 |

---

## 4. 결과 분석 프레임워크

### 4.1 통계적 유의성 검정

```
사용 검정 방법:
- 비율 비교 (CTR, 전환율): Two-proportion Z-test
- 연속 변수 (체류 시간): Mann-Whitney U test (비정규 분포 가정)
- 다중 비교 보정: Bonferroni correction (복수 지표 동시 검정 시)

판정 기준:
  p-value < 0.05 → 통계적으로 유의미
  p-value ≥ 0.05 → 차이 없음 (귀무가설 채택)
```

### 4.2 신뢰구간 계산

```
95% 신뢰구간 = 관측 차이 ± 1.96 × 표준 오차

표준 오차 (SE) = sqrt(p1(1-p1)/n1 + p2(1-p2)/n2)

해석 예시:
  A그룹 CTR: 6.5%
  B그룹 CTR: 7.8%
  차이: +1.3%p
  95% CI: [+0.4%p, +2.2%p]
  → 신뢰구간이 0을 포함하지 않으므로 유의미 (배포 권장)
```

### 4.3 의사결정 트리

```
실험 종료
    │
    ▼
p-value < 0.05?
    ├── NO  → 차이 없음 → 현행 유지
    └── YES
         │
         ▼
    실용적 유의성?
    (MDE 이상의 효과?)
         ├── NO  → 통계 유의하나 실용 효과 미미 → 현행 유지
         └── YES
              │
              ▼
         부작용 지표 악화?
         (이탈률, 불만 증가?)
              ├── YES → 추가 분석 필요 → 설계 재검토
              └── NO  → 배포 결정 ✓
```

### 4.4 실험 로그 데이터 스키마

```typescript
interface ABExperimentEvent {
  experimentId: string;       // "exp_001_card_sort"
  variant: 'A' | 'B';
  userId: string;
  eventType: 'impression' | 'click' | 'apply' | 'session_end';
  timestamp: Date;
  sessionId: string;
  metadata?: {
    durationMs?: number;
    scrollDepth?: number;
    deviceType?: 'mobile' | 'tablet' | 'desktop';
  };
}
```

---

## 5. 성능 최적화 계획

### 5.1 Core Web Vitals 목표

| 지표 | 측정 기준 | 현재 목표 | 최적 목표 |
|------|---------|---------|---------|
| **LCP** (Largest Contentful Paint) | 뷰포트 내 최대 콘텐츠 로딩 | < 2.5초 | < 1.8초 |
| **FID** (First Input Delay) | 첫 상호작용 지연 | < 100ms | < 50ms |
| **CLS** (Cumulative Layout Shift) | 레이아웃 이동 누적 | < 0.1 | < 0.05 |
| **INP** (Interaction to Next Paint) | 상호작용 응답성 | < 200ms | < 100ms |
| **FCP** (First Contentful Paint) | 첫 콘텐츠 표시 | < 1.8초 | < 1.2초 |

### 5.2 Next.js 최적화 전략

#### 이미지 최적화
```typescript
// components/cards/JobCard.tsx
import Image from 'next/image';

// 기업 로고
<Image
  src={company.logoUrl}
  alt={company.name}
  width={48}
  height={48}
  priority={isAboveFold}  // 첫 화면 이미지만 priority
  loading={isAboveFold ? 'eager' : 'lazy'}
/>
```

#### 코드 스플리팅
```typescript
// 차트 컴포넌트 동적 임포트 (Recharts 번들 크기 감소)
const ChartWidget = dynamic(
  () => import('@/components/charts/ChartWidget'),
  { loading: () => <LoadingSkeleton />, ssr: false }
);

// 대시보드 페이지만 차트 번들 로드
```

#### ISR (Incremental Static Regeneration)
```typescript
// app/jobs/[id]/page.tsx
export async function generateStaticParams() {
  // 인기 공고 500개 사전 빌드
  return topJobs.map(job => ({ id: job.jobId }));
}

export const revalidate = 3600; // 1시간마다 재생성
```

#### React Query 캐싱 전략
```typescript
const { data: recommendations } = useQuery({
  queryKey: ['recommendations', userId, filters],
  queryFn: () => fetchRecommendations(userId, filters),
  staleTime: 5 * 60 * 1000,   // 5분 캐시 유지
  cacheTime: 30 * 60 * 1000,  // 30분 보관
  refetchOnWindowFocus: false,
});
```

### 5.3 번들 크기 최적화

| 최적화 항목 | 방법 | 예상 감소 |
|-----------|------|---------|
| Chart.js 트리쉐이킹 | 필요한 차트 타입만 import | -150KB |
| 아이콘 최적화 | `lucide-react` 개별 import | -50KB |
| Date 라이브러리 | `date-fns` (Moment.js 대체) | -200KB |
| 폰트 최적화 | `next/font` 로컬 폰트 서브셋 | -100KB |

### 5.4 반응형 이미지 전략

```
breakpoint 별 이미지 사이즈:
- 모바일: 기업 로고 40×40px
- 태블릿: 48×48px
- 데스크탑: 56×56px

sizes 속성:
  "(max-width: 768px) 40px, (max-width: 1024px) 48px, 56px"
```

---

## 6. 모니터링 대시보드 와이어프레임

```
┌────────────────────────────────────────────────────────────────┐
│              A/B 테스트 모니터링 대시보드                        │
├─────────────────────┬──────────────────────────────────────────┤
│  실험 목록           │  실험 상세: 시나리오 2 (스코어 시각화)     │
│  ┌─────────────────┐│  ┌────────────────────────────────────┐  │
│  │ ● 실험 1 (진행) ││  │  기간: 2025-04-07 ~ 2025-04-21    │  │
│  │ ● 실험 2 (진행) ││  │  상태: 진행 중 (8일 경과, 6일 남음) │  │
│  │ ✓ 실험 3 (완료) ││  │  샘플: A=2,341명 | B=2,389명      │  │
│  │ ○ 실험 4 (대기) ││  └────────────────────────────────────┘  │
│  │ ○ 실험 5 (대기) ││                                          │
│  └─────────────────┘│  지표 추이 (일별)                        │
│                     │  ┌────────────────────────────────────┐  │
│  핵심 지표 요약       │  │ CTR                               │  │
│  ┌─────────────────┐│  │ 10% ─── B (실험군) ─────────▶      │  │
│  │ 실험 2 진행률    ││  │  8%                               │  │
│  │ ████████░░ 57%  ││  │  6% ─── A (대조군) ─────────▶      │  │
│  │                 ││  │     D1  D2  D3  D4  D5  D6  D7  D8  │  │
│  │ 현재 CTR 차이   ││  └────────────────────────────────────┘  │
│  │ A: 6.8%         ││                                          │
│  │ B: 8.2% (+20%)  ││  통계 검정 결과                          │
│  │ 신뢰도: 91%     ││  p-value: 0.031 ✓ (< 0.05)             │
│  └─────────────────┘│  95% CI: [+0.8%p, +2.1%p]              │
└─────────────────────┴──────────────────────────────────────────┘
```

---

## 7. 구현 컴포넌트 설계

### A/B 테스트 훅

```typescript
// hooks/useABTest.ts
interface ABTestConfig {
  experimentId: string;
  variants: ['A', 'B'];
}

function useABTest(config: ABTestConfig): 'A' | 'B' {
  const { userId } = useUserStore();
  // 일관된 사용자 분류 보장
  return assignVariant(userId, config.experimentId);
}

// 사용 예시
const variant = useABTest({ experimentId: 'exp_002_score_viz' });
return variant === 'B' ? <MatchScoreGauge /> : <MatchScoreText />;
```

### 이벤트 트래킹 유틸리티

```typescript
// lib/analytics/abTracking.ts
function trackABEvent(
  experimentId: string,
  variant: 'A' | 'B',
  eventType: string,
  metadata?: Record<string, unknown>
): void {
  // 배치 업로드 (5초마다 flush)
  eventQueue.push({ experimentId, variant, eventType, ...metadata });
}
```

---

## 8. MVP 산출물 (Deliverables)

> 4일 일정 기준 — 시나리오 2 (스코어 시각화) 1개만 Day 3까지 구현. 나머지 시나리오는 설계 문서 보존.

| 산출물 | 형태 | 목표 완성 시점 | 우선순위 |
|-------|------|-------------|---------|
| A/B 변환 유틸리티 | `lib/abtest/assignVariant.ts` (해시 기반 분류) | Day 2 오후 | Must |
| 이벤트 트래킹 모듈 | `lib/analytics/abTracking.ts` (배치 큐) | Day 2 오후 | Must |
| `useABTest` 훅 | `hooks/useABTest.ts` | Day 3 오전 | Must |
| 시나리오 2 적용 | MatchScoreText vs MatchScoreGauge A/B 분기 | Day 3 오전 | Must |
| 반응형 성능 최적화 | Next.js Image, dynamic import, Tailwind purge | Day 3~4 | Must |
| Lighthouse 점수 검증 | 수동 실행 후 결과 캡처 (목표: 90+) | Day 4 오전 | Must |
| A/B 모니터링 대시보드 | `app/dashboard/ab-tests/page.tsx` 간략 버전 | Day 4 여유 시 | Nice |
| 시나리오 3~5 구현 | 추후 구현 (설계 문서만 완성) | - | Nice |
