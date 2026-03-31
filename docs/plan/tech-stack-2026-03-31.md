# CareerLog OS — 기술 스택 및 환경 설정 문서

> 작성일: 2026-03-31
> 관련 계획: `mvp-plan-2026-03-31.md`

---

## 1. 선정 기술 스택

### 핵심 스택

| 기술 | 버전 | 선정 이유 |
|-----|------|---------|
| **Next.js** | 14.x (App Router) | SSR/SSG 지원, API Routes 내장, Vercel 배포 최적화, SEO 친화적 |
| **TypeScript** | 5.x | PRD 데이터 스키마가 TS로 정의됨, 컴파일 타임 타입 안전성 |
| **Tailwind CSS** | 3.x | 유틸리티 기반 빠른 반응형 구현, 커스텀 디자인 토큰 지원 |
| **React** | 18.x (Next.js 내장) | 컴포넌트 기반 UI, Server Components 활용 |

### 개발 도구

| 도구 | 목적 |
|-----|------|
| **ESLint** | 코드 품질 및 규칙 강제 (`next/core-web-vitals` 프리셋) |
| **Prettier** | 코드 포맷 일관성 |
| **Prisma** | DB ORM (MVP: schema-only, 실제 연결은 v2) |

### 배포

| 서비스 | 목적 |
|-------|------|
| **Vercel** | Next.js 공식 배포 플랫폼, 무료 티어, 자동 HTTPS, CI/CD |

---

## 2. DB 환경 준비 전략

### MVP 방침: Mock 데이터 파일 전용

MVP 단계에서는 **실제 데이터베이스를 연결하지 않는다.**
대신 Prisma 스키마와 환경변수 파일만 준비하여 v2에서 DB 연결 시 최소한의 수정으로 전환 가능하게 한다.

```
MVP 현재                        v2 전환 시
─────────────────────           ─────────────────────
/src/data/mock/jobs.json  →  Prisma Client + PostgreSQL
API Route reads JSON      →  API Route queries DB
.env.local (URL 준비만)   →  실제 DATABASE_URL 입력
```

### Prisma 설치 및 설정 (schema-only)

```bash
npm install prisma @prisma/client
npx prisma init
```

`prisma/schema.prisma`:
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model UserProfile {
  id              String        @id @default(cuid())
  name            String
  experienceYears Int
  portfolioUrls   String[]
  skills          UserSkill[]
  projects        ProjectCard[]
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt
}

model Skill {
  id               String      @id @default(cuid())
  name             String      @unique
  userSkills       UserSkill[]
  jobRequirements  JobSkill[]
}

model UserSkill {
  id                String      @id @default(cuid())
  userId            String
  skillId           String
  yearsOfExperience Int?
  user              UserProfile @relation(fields: [userId], references: [id])
  skill             Skill       @relation(fields: [skillId], references: [id])
}

model ProjectCard {
  id             String      @id @default(cuid())
  userId         String
  techStack      String[]
  role           ProjectRole
  teamSize       Int
  durationMonths Int
  url            String?
  description    String?
  user           UserProfile @relation(fields: [userId], references: [id])
}

enum ProjectRole {
  LEAD
  MEMBER
  SOLO
}

model JobPosting {
  id                 String       @id @default(cuid())
  title              String
  company            String
  location           String
  employmentType     String
  salaryRange        String
  levelTag           LevelTag
  requiredExperienceMin Int
  requiredExperienceMax Int?
  description        String
  postedAt           DateTime
  requiredSkills     JobSkill[]
  cultureTags        CultureTag[]
  createdAt          DateTime     @default(now())
}

enum LevelTag {
  ENTRY
  JUNIOR
  MIDDLE
  SENIOR
}

model JobSkill {
  id         String     @id @default(cuid())
  jobId      String
  skillId    String
  job        JobPosting @relation(fields: [jobId], references: [id])
  skill      Skill      @relation(fields: [skillId], references: [id])
}

model CultureTag {
  id     String     @id @default(cuid())
  jobId  String
  label  String
  rating Float?
  job    JobPosting @relation(fields: [jobId], references: [id])
}
```

### 환경변수 파일

`.env.example` (커밋):
```env
# Database (v2에서 실제 값으로 교체)
DATABASE_URL="postgresql://user:password@localhost:5432/careerlog_os"

# AI API (v2에서 실제 값으로 교체)
OPENAI_API_KEY=""

# App
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

`.env.local` (gitignore):
```env
DATABASE_URL=""
OPENAI_API_KEY=""
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

---

## 3. Mock 데이터 전략

### 위치 및 구조

```
/src/data/mock/
├── jobs.json          # 공고 30건
├── users.json         # 사용자 5건 (구직자)
└── candidates.json    # 지원자 데이터 20건 (기업 HR용)
```

### API Route에서 Mock 데이터 사용 방식

```typescript
// /src/app/api/recommend/route.ts
import { NextResponse } from 'next/server';
import jobs from '@/data/mock/jobs.json';

export async function POST(request: Request) {
  const body = await request.json();
  // Mock: 스코어 계산 후 반환
  const result = jobs.map(job => ({
    ...job,
    matchScore: calculateScore(job, body.userProfile),
  }));
  return NextResponse.json(result);
}
```

> v2 전환 시: `import jobs from '@/data/mock/jobs.json'`을
> `const jobs = await prisma.jobPosting.findMany()` 로 교체

---

## 4. 성능 목표 및 달성 전략

| 지표 | 목표 | 달성 방법 |
|-----|------|---------|
| Lighthouse Performance | ≥ 90 | next/image 사용, 폰트 최적화, SSR/SSG 활용 |
| LCP | ≤ 2.5s | 서버 컴포넌트로 초기 데이터 로드 |
| FID | ≤ 100ms | 인터랙티브 JS 최소화 |
| CLS | ≤ 0.1 | 이미지 크기 명시, 폰트 fallback |
| Lighthouse Accessibility | ≥ 90 | aria-label, 시맨틱 HTML, 색상 대비 |
| Lighthouse SEO | ≥ 90 | 메타태그, Open Graph |

### 핵심 최적화 포인트

1. **Server Components 우선**: 데이터 페칭은 서버 컴포넌트에서 수행 (클라이언트 JS 최소화)
2. **next/image**: 모든 이미지는 `<Image>` 컴포넌트 사용
3. **폰트**: `next/font` 사용, 로컬 서브셋 폰트
4. **코드 스플리팅**: 동적 임포트로 사이드 패널 컴포넌트 지연 로드

---

## 5. HTML5 시맨틱 마크업 가이드

모든 페이지는 HTML5 시맨틱 태그를 준수한다.

### 기본 페이지 구조

```html
<body>
  <header>
    <nav aria-label="주 네비게이션">
      <!-- 네비게이션 링크 -->
    </nav>
  </header>

  <main id="main-content">
    <!-- 페이지 주요 콘텐츠 -->
  </main>

  <footer>
    <!-- 푸터 정보 -->
  </footer>
</body>
```

### 공고 목록 구조

```html
<main>
  <section aria-labelledby="job-list-heading">
    <h1 id="job-list-heading">추천 공고</h1>
    <aside aria-label="필터">
      <!-- 레벨 필터, 문화 태그 필터 -->
    </aside>
    <ul role="list" aria-label="공고 목록">
      <li>
        <article aria-labelledby="job-title-1">
          <h2 id="job-title-1">프론트엔드 개발자</h2>
          <!-- 매칭 스코어, 스킬 배지 -->
        </article>
      </li>
    </ul>
  </section>
</main>
```

### 접근성 필수 사항

- 게이지 바: `role="progressbar"`, `aria-valuenow`, `aria-valuemin`, `aria-valuemax`
- 스킬 배지: 색상만으로 구분 금지 → 텍스트 + 아이콘 병용
- 사이드 패널: `role="dialog"`, `aria-modal="true"`, `aria-labelledby`
- 모달 열릴 때 포커스 트랩

---

## 6. 코드 규칙

### 파일 네이밍

| 유형 | 규칙 | 예시 |
|-----|------|------|
| 컴포넌트 | PascalCase | `JobCard.tsx` |
| 유틸리티/훅 | camelCase | `scoring.ts`, `useABTest.ts` |
| 페이지 | next.js 규칙 | `page.tsx` |
| 타입 | index.ts 통합 | `/src/types/index.ts` |
| Mock 데이터 | kebab-case.json | `jobs.json` |

### 컴포넌트 규칙

- 서버 컴포넌트 기본, 인터랙션 필요 시 `'use client'` 명시
- Props는 인터페이스로 정의 (`interface JobCardProps`)
- 인라인 스타일 금지 — Tailwind 클래스만 사용

---

## 관련 문서

| 문서 | 경로 |
|-----|------|
| MVP 전체 계획 | `docs/plan/mvp-plan-2026-03-31.md` |
| Phase 0 체크리스트 | `docs/checklist/phase-0-setup.md` |
