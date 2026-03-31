# Phase 0 체크리스트 — 환경 설정 및 문서화

> 작성일: 2026-03-31
> Phase: 0 | Day: 1 전반
> 목표: 개발 환경 완성 + 모든 문서 초안 작성 완료

---

## 진행 상황

| 항목 | 상태 | 완료일 |
|-----|------|-------|
| 문서 구조 생성 | ⬜ 미완료 | - |
| Next.js 초기화 | ⬜ 미완료 | - |
| Tailwind CSS 설정 | ⬜ 미완료 | - |
| ESLint + Prettier 설정 | ⬜ 미완료 | - |
| Prisma 설치 및 스키마 작성 | ⬜ 미완료 | - |
| TypeScript 타입 정의 | ⬜ 미완료 | - |
| Mock 데이터 작성 | ⬜ 미완료 | - |
| 환경변수 파일 준비 | ⬜ 미완료 | - |

> 상태 표기: ⬜ 미완료 / 🔄 진행 중 / ✅ 완료

---

## 체크리스트

### 1. 문서 구조 생성

- [ ] `/docs/plan/` 디렉토리 생성
- [ ] `/docs/checklist/` 디렉토리 생성
- [ ] `/docs/dev-log/` 디렉토리 생성
- [ ] `/docs/errors/` 디렉토리 생성
- [ ] `docs/plan/mvp-plan-2026-03-31.md` 작성 완료
- [ ] `docs/plan/tech-stack-2026-03-31.md` 작성 완료
- [ ] `docs/checklist/phase-0-setup.md` 작성 완료 (이 파일)
- [ ] `docs/checklist/phase-1-core-ui.md` 작성 완료
- [ ] `docs/checklist/phase-2-features.md` 작성 완료
- [ ] `docs/checklist/phase-3-qa.md` 작성 완료
- [ ] `docs/checklist/feature-acceptance.md` 작성 완료
- [ ] `docs/dev-log/dev-log-template.md` 작성 완료
- [ ] `docs/errors/error-template.md` 작성 완료

### 2. Next.js 프로젝트 초기화

```bash
npx create-next-app@latest . \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"
```

- [ ] Next.js 14 App Router 설치 완료
- [ ] TypeScript 설정 확인 (`tsconfig.json` 존재)
- [ ] Tailwind CSS 설정 확인 (`tailwind.config.ts` 존재)
- [ ] ESLint 설정 확인 (`.eslintrc.json` 존재)
- [ ] `src/` 디렉토리 구조 확인
- [ ] `npm run dev` 실행 시 `http://localhost:3000` 정상 응답 확인

### 3. Prettier 설정

- [ ] `npm install --save-dev prettier eslint-config-prettier`
- [ ] `.prettierrc` 파일 생성:
  ```json
  {
    "semi": true,
    "trailingComma": "all",
    "singleQuote": true,
    "printWidth": 100,
    "tabWidth": 2
  }
  ```
- [ ] `.eslintrc.json` 에 `"extends": ["next/core-web-vitals", "prettier"]` 추가

### 4. 디렉토리 구조 생성

```bash
mkdir -p src/components/ui
mkdir -p src/components/jobs
mkdir -p src/components/profile
mkdir -p src/components/employer
mkdir -p src/data/mock
mkdir -p src/lib
mkdir -p src/types
mkdir -p src/app/api/recommend
mkdir -p src/app/api/jobs/\[id\]/report
mkdir -p src/app/api/jobs/\[id\]/simulate
mkdir -p src/app/api/jobs/\[id\]/culture
mkdir -p src/app/api/employer/filter
mkdir -p src/app/api/employer/skill-fit
mkdir -p src/app/api/ab
mkdir -p src/app/jobs/\[id\]
mkdir -p src/app/profile
mkdir -p src/app/employer
```

- [ ] 모든 디렉토리 생성 완료
- [ ] `src/app/jobs/page.tsx` 생성 (빈 파일)
- [ ] `src/app/jobs/[id]/page.tsx` 생성 (빈 파일)
- [ ] `src/app/profile/page.tsx` 생성 (빈 파일)
- [ ] `src/app/employer/page.tsx` 생성 (빈 파일)

### 5. TypeScript 타입 정의

- [ ] `src/types/index.ts` 생성
- [ ] `Skill` 인터페이스 정의
- [ ] `ProjectCard` 인터페이스 정의
- [ ] `UserProfile` 인터페이스 정의
- [ ] `JobPreference` 인터페이스 정의
- [ ] `LevelTag` 타입 정의 (`'entry' | 'junior' | 'middle' | 'senior'`)
- [ ] `CultureTag` 인터페이스 정의
- [ ] `FilterCondition` 인터페이스 정의
- [ ] `JobPosting` 인터페이스 정의
- [ ] `CandidateScore` 인터페이스 정의
- [ ] `FilterCategory` 타입 정의
- [ ] `CandidateApplication` 인터페이스 정의
- [ ] `ABVariant` 타입 정의 (`'A' | 'B'`)
- [ ] TypeScript 컴파일 에러 없음 확인 (`npx tsc --noEmit`)

### 6. Prisma 설치 및 스키마 작성

```bash
npm install prisma @prisma/client
npx prisma init
```

- [ ] Prisma 설치 완료
- [ ] `prisma/schema.prisma` 생성
- [ ] `UserProfile` 모델 정의
- [ ] `Skill` 모델 정의
- [ ] `UserSkill` 관계 모델 정의
- [ ] `ProjectCard` 모델 정의
- [ ] `ProjectRole` enum 정의 (`LEAD`, `MEMBER`, `SOLO`)
- [ ] `JobPosting` 모델 정의
- [ ] `LevelTag` enum 정의 (`ENTRY`, `JUNIOR`, `MIDDLE`, `SENIOR`)
- [ ] `JobSkill` 관계 모델 정의
- [ ] `CultureTag` 모델 정의
- [ ] `npx prisma validate` 스키마 검증 통과
  > 주의: DATABASE_URL이 없어도 validate는 통과 가능

### 7. Mock 데이터 작성

#### `src/data/mock/jobs.json` — 공고 30건

- [ ] 프론트엔드 개발자 공고 10건 작성
  - 레벨 분포: 주니어 4건, 미들 4건, 시니어 2건
  - React/TypeScript 중심, 일부 AWS/Node.js 포함
- [ ] 백엔드 개발자 공고 8건 작성
  - Node.js 3건, Java 3건, Python 2건
- [ ] UX/UI 디자이너 공고 7건 작성
  - Figma/Sketch 중심
- [ ] 데이터 분석가 공고 5건 작성
- [ ] 전체 공고 레벨 분포: 신입 3건, 주니어 10건, 미들 12건, 시니어 5건
- [ ] 각 공고에 `cultureTags` 포함 (0~5개)
- [ ] `matchScore` 필드 없이 저장 (런타임에 계산)

#### `src/data/mock/users.json` — 사용자 5건

- [ ] 이민준 (29세, FE 주니어, 경력 2년, jQuery/HTML/CSS, React 목표)
- [ ] 박지현 (34세, UX 시니어, 경력 7년, Figma/Sketch, 프로젝트 카드 3개)
- [ ] 더미 사용자 3명 (다양한 직군)

#### `src/data/mock/candidates.json` — 지원자 20건

- [ ] 적합 후보 8건 (`filterCategory: "suitable"`)
- [ ] 검토 필요 7건 (`filterCategory: "review"`)
- [ ] 조건 미충족 5건 (`filterCategory: "unqualified"`)
- [ ] 각 지원자에 `skillFitScore` 포함 (0~100)

### 8. 환경변수 파일 준비

- [ ] `.env.example` 생성 (커밋 대상)
  ```
  DATABASE_URL="postgresql://user:password@localhost:5432/careerlog_os"
  OPENAI_API_KEY=""
  NEXT_PUBLIC_APP_URL="http://localhost:3000"
  ```
- [ ] `.env.local` 생성 (gitignore 대상)
  ```
  DATABASE_URL=""
  OPENAI_API_KEY=""
  NEXT_PUBLIC_APP_URL="http://localhost:3000"
  ```
- [ ] `.gitignore`에 `.env.local` 포함 확인

---

## 완료 기준

- [ ] `npm run dev` 정상 실행 (에러 없음)
- [ ] `npm run build` 정상 완료 (에러 없음)
- [ ] `npx tsc --noEmit` 타입 에러 없음
- [ ] `npx prisma validate` 스키마 검증 통과
- [ ] `/src/data/mock/` 데이터 파일 3개 존재 및 타입 일치 확인
- [ ] 모든 문서 파일 (`/docs/`) 생성 완료

---

## 에러 발생 시

에러 발생 시 즉시 `docs/errors/error-YYYYMMDD-{번호}.md` 파일을 생성하여 기록한다.
자세한 양식은 `docs/errors/error-template.md` 참조.
