# 개인 개발 아키텍처 표준 v1.0

> **문서 성격:** 개발자 개인의 범용 설계 규칙. 특정 서비스·브랜드·프로젝트에 종속되지 않는다.  
> **기준일:** 2026-11-20  
> **상태:** v1.0 확정 — 인터뷰 1~31번 및 마무리 3개 질문 반영  
> **주 적용 환경:** TypeScript / React / Next.js App Router / Tailwind CSS. 프레임워크 전용 규칙은 명시적으로 구분한다.  
> **핵심 목표:** 사람과 AI가 같은 파일 배치·책임 분리·코드 변경 기준을 사용하도록 한다.

## 목차

1. [문서 사용법과 판단 우선순위](#1-문서-사용법과-판단-우선순위)
2. [설계 철학](#2-설계-철학)
3. [최상위 디렉터리와 라우트 그룹](#3-최상위-디렉터리와-라우트-그룹)
4. [페이지·UI·공통 컴포넌트](#4-페이지ui공통-컴포넌트)
5. [DOM ID, HTML, 접근성](#5-dom-id-html-접근성)
6. [데이터·타입·로직 계층](#6-데이터타입로직-계층)
7. [의존성 및 Import·Export](#7-의존성-및-importexport)
8. [네이밍 규칙](#8-네이밍-규칙)
9. [React·Next.js 데이터 흐름과 상태](#9-reactnextjs-데이터-흐름과-상태)
10. [캐싱·갱신·실패·상태 UI](#10-캐싱갱신실패상태-ui)
11. [Tailwind 및 CSS 작성 규칙](#11-tailwind-및-css-작성-규칙)
12. [디자인 토큰과 전역 스타일](#12-디자인-토큰과-전역-스타일)
13. [SEO 중앙 관리](#13-seo-중앙-관리)
14. [입력 검증·API·인증·환경변수](#14-입력-검증api인증환경변수)
15. [ChatGPT 코드 변경 계약](#15-chatgpt-코드-변경-계약)
16. [기능 추가 표준 예시](#16-기능-추가-표준-예시)
17. [완료 체크리스트](#17-완료-체크리스트)
18. [미확정·후속 업데이트](#18-미확정후속-업데이트)

---

## 1. 문서 사용법과 판단 우선순위

### 1.1 규범 용어

- **[필수]**: 새로운 코드와 관련 수정 코드에서 반드시 준수한다.
- **[기본]**: 우선 적용한다. 실제 조건이 맞지 않으면 적합한 대안을 선택하고 이유를 남긴다.
- **[선택]**: 요구가 발생할 때 도입한다. 아직 사용하지 않는 폴더·도구는 미리 만들지 않는다.
- **[보류]**: v1.0에서 최종 설계를 정하지 않았다. 확정 규칙처럼 적용하지 않는다.

### 1.2 충돌 시 우선순위

1. 보안·데이터 무결성·접근성과 같은 필수 안전 제약 및 프레임워크의 실제 동작 조건
2. 사용자가 이번 작업에서 명시한 요구와 범위
3. 이 문서의 확정 개인 아키텍처 규칙
4. 현재 프로젝트에서 명시적으로 승인된 프로젝트별 예외
5. 일반적인 라이브러리 관례와 AI의 개인적 선호

규칙 충돌이나 모호함이 있으면 조용히 새 아키텍처를 도입하지 않는다. **기존 확정 규칙을 최대한 유지하고, 규칙 밖의 변경은 제안한다.**

### 1.3 범용 규칙과 프레임워크 전용 규칙

컴포넌트 분리, 계층 책임, 의존성, 명명, 변경 통제는 **범용 개인 규칙**이다. `(site)`, `page.tsx`, Server Component, Tailwind `@theme` 등은 **Next.js·React·Tailwind 사용 프로젝트에 적용하는 전용 규칙**이다. 다른 기술을 쓰는 프로젝트에는 파일명·API를 억지로 복제하지 말고 원칙만 대응시킨다.

## 2. 설계 철학

1. **[필수] 역할별 책임 분리:** UI, 공통 컴포넌트, 정적 데이터, 공유 타입, 순수 함수, 기술 연동, Hook, 비즈니스 서비스를 구분한다.
2. **[필수] 적극적인 최소 단위 분리:** 한 파일·컴포넌트는 가능한 한 작고 독립적인 책임을 가진다. 별도 의미가 있는 카드·아이템·지도·필터·액션 묶음은 분리한다.
3. **[필수] 구조의 예측 가능성:** 파일이 어떤 기능에 속하는지 경로로 추적할 수 있게 한다.
4. **[필수] 의존성 방향 유지:** 하위 공통 계층이 자신을 호출하는 페이지 UI에 의존하지 않는다. 순환 참조를 금지한다.
5. **[기본] 불필요한 깊이·추상화 배제:** 역할이 명확하다고 빈 폴더를 만들거나, 작은 코드마다 독립 폴더·라이브러리를 만들지 않는다.
6. **[기본] 실제 재사용 시 공통화:** 향후 재사용 가능성만으로 성급히 공통화하지 않는다. 단, 이미 존재하는 적합한 공통 컴포넌트는 우선 재사용한다.
7. **[필수] 검색엔진 친화적 공개 웹페이지:** 시맨틱 HTML, SEO, 서버가 읽을 수 있는 핵심 콘텐츠를 고려한다. SEO 성과 자체를 보장하지는 않는다.
8. **[필수] AI의 수정 기준:** 확정 규칙 내의 관련 코드는 적극 개선할 수 있지만, 규칙을 벗어나는 제안·무관한 대규모 재구성은 사전 승인 없이 시행하지 않는다.

## 3. 최상위 디렉터리와 라우트 그룹

### 3.1 기본 구조 (Next.js App Router)

```text
src/
└── app/
    ├── (site)/                  # 일반 사이트 화면을 모아 두는 URL 비노출 Route Group
    │   ├── page.tsx
    │   └── products/
    │       └── page.tsx
    ├── (admin)/                 # 관리자 라우트가 필요할 경우
    │   └── dashboard/
    │       └── page.tsx
    ├── (api)/                   # API 라우트 구분용 Route Group
    │   └── api/
    │       └── products/
    │           └── route.ts
    ├── _ui/                     # 페이지 전용 UI 및 조립
    │   ├── home/
    │   │   └── overview.tsx
    │   ├── products/
    │   │   ├── overview.tsx
    │   │   ├── hero/
    │   │   │   └── section.tsx
    │   │   └── list/
    │   │       ├── section.tsx
    │   │       └── card.tsx
    │   └── dashboard/
    │       └── overview.tsx
    ├── _components/             # 페이지 간 재사용하는 UI
    ├── _data/                   # 정적 콘텐츠와 구조화된 정적 데이터
    ├── _types/                  # 여러 파일에서 공유하는 타입
    ├── _utils/                  # 프레임워크 독립적인 순수 함수
    ├── _lib/                    # 외부 기술·SDK·인프라 설정 및 연동
    ├── _hooks/                  # React 커스텀 Hook
    ├── _services/               # 비즈니스 동작 및 서버 데이터 처리
    ├── _schemas/                # [선택] 복잡·재사용 입력 검증 스키마
    ├── _styles/                 # [선택] globals.css가 복잡해지면 분리
    ├── globals.css              # Tailwind·전역 스타일·기본 디자인 토큰
    ├── layout.tsx
    ├── sitemap.ts               # [선택] 실제 필요 시
    └── robots.ts                # [선택] 실제 필요 시
```

위 트리는 **역할 배치를 보여 주는 예시**다. `[선택]` 폴더나 비사용 기능까지 모두 생성하지 않는다. `_lib/env`, `_lib/seo`도 실제 사용하는 경우에만 만든다.

### 3.2 경로 규칙

- **[필수]** 앱 내 최상위 **비라우트 책임 폴더**는 `_ui`, `_components`, `_data`처럼 `_` 접두사로 구분한다.
- **[필수]** `(site)`, `(api)`, `(admin)`은 URL에 나타나지 않는 Next.js **Route Group**으로, 라우트 파일을 분류하는 목적이다. `(api)/api/.../route.ts`에서 URL은 `/api/...`이다.
- **[필수]** `_ui/site`, `_data/site`, `_types/site` 같은 **중복된 `site` 폴더는 만들지 않는다.** `_ui/products`처럼 실제 기능 경로를 직접 표현한다.
- **[선택]** 사이트·관리자의 실제 UI 경로가 충돌할 때에는 해당 역할 폴더 안에서 `admin/` 같은 구분용 네임스페이스를 도입한다. 무조건 추가하지 않는다.
- **[기본]** 단일 파일은 해당 역할 폴더에 바로 둔다. 같은 기능 파일이 늘거나 명확한 독립 묶음이 생기면 하위 폴더를 만든다. **예외적으로 `_ui`의 섹션은 처음부터 기능 폴더를 사용한다.**
- **[필수]** 폴더 깊이가 늘어난다는 이유만으로 책임을 섞거나, 책임을 나누겠다는 이유만으로 의미 없는 디렉터리 계층을 만들지 않는다.

## 4. 페이지·UI·공통 컴포넌트

### 4.1 페이지 조립 흐름

```text
(page.tsx) → (_ui/<page>/overview.tsx)
             → (<feature>/section.tsx)
             → (card.tsx, item.tsx, map.tsx, filter.tsx 등 최소 역할 UI)
             → (필요한 _components 공통 컴포넌트)
```

- **`page.tsx`**: 라우트의 진입점, 필요한 메타데이터 선언, 서버 데이터 연결, 페이지 진입 처리.
- **`overview.tsx`**: 해당 페이지 섹션의 **호출 순서와 조립**을 주로 담당.
- **`<기능>/section.tsx`**: 한 섹션의 시맨틱 구조, 필요한 하위 UI 조립.
- **`<기능>/card.tsx` / `item.tsx` 등**: 독립적인 역할의 최소 단위 UI.

```tsx
// _ui/products/overview.tsx
import ProductsHeroSection from './hero/section';
import ProductsListSection from './list/section';

export default function ProductsOverview() {
  return (
    <>
      <ProductsHeroSection />
      <ProductsListSection />
    </>
  );
}
```

여기서 `<>...</>`는 DOM을 만들지 않으므로 `overview` 자체에 별도 `id`를 넣으려고 불필요한 래퍼를 생성하지 않는다.

### 4.2 `_ui`와 `_components`의 경계

| 사용 범위 | 위치 | 예시 |
|---|---|---|
| 한 섹션에서만 사용 | `_ui/products/list/` | `card.tsx` |
| 같은 페이지의 여러 섹션에서만 사용 | `_ui/products/shared/` (필요할 때) | 페이지 전용 배지 |
| 여러 페이지가 실제로 공유 | `_components/` | 공통 Button, 공통 Card |
| 전역 공통이 아니라 특정 영역에서만 공유 | `_components/<도메인>/` | 여러 상품 페이지 공통 Card |

- **[필수]** `_ui`는 페이지 전용 UI **조립/표현의 영역**, `_components`는 **페이지 간 재사용 컴포넌트의 영역**이다. 둘 다 React 컴포넌트이지만 개인 아키텍처에서 의미를 다르게 정의한다.
- **[기본]** 한 페이지에서만 사용하는 코드는 `_ui`에 둔다. 여러 페이지에서 동일한 책임·동작을 공유할 때 `_components`로 승격한다.
- **[필수]** 모양만 비슷한 컴포넌트를 억지로 공통화하지 않는다. 지나친 조건 분기·Props 증가가 발생하면 별개로 유지한다.
- **[기본]** 컴포넌트는 적극 분리하지만, 모든 컴포넌트마다 별도 폴더를 만들지는 않는다.

## 5. DOM ID, HTML, 접근성

### 5.1 DOM 식별 규칙

- **[필수]** 독립적인 UI 컴포넌트가 최상위에 **실제 DOM 요소 하나**를 렌더링한다면 그 요소에 의미 있는 고유 `id`를 부여한다.
- **[기본]** `id`는 소문자 영문 `kebab-case`이며 `페이지-섹션-역할-식별자`처럼 경로를 추적할 수 있게 구성한다.
- **[필수]** 렌더된 **동일 문서 내 ID 중복 금지**. 반복 항목은 안정적인 데이터 식별자를 이용한다. 목록 인덱스는 가급적 쓰지 않는다.
- **[기본]** 한 컴포넌트가 여러 곳에서 렌더링되면 `id`를 Props로 받아 호출자가 고유하게 제공한다.
- **[필수]** 내부 `h3`, `p`, `button`, `div` 등에는 **모두 `id`를 강제하지 않는다.** 앵커·ARIA 연결·제어·추적이 필요할 때만 부여한다.
- **[필수]** Fragment만 반환하거나 Provider처럼 최상위 DOM 자체가 없는 컴포넌트는 예외다. `id` 때문에 불필요한 DOM 래퍼를 만들지 않는다.
- **[필수]** 기존 ID의 변경은 앵커 링크·JS 참조·테스트·ARIA 연결에 미치는 영향을 확인한다.

```tsx
// _ui/products/list/card.tsx
type ProductCardProps = {
  id: string;
  name: string;
};

export default function ProductCard({ id, name }: ProductCardProps) {
  return (
    <article id={id}>
      <h3>{name}</h3>
    </article>
  );
}
```

### 5.2 시맨틱 HTML·접근성·SEO 우선

- **[필수]** `main`, `section`, `article`, `nav`, `h1`~`h6`, `ul`/`li`, `button`, `a`를 의미에 맞게 사용하고 레이아웃 목적으로는 `div`를 사용한다.
- **[필수]** 동작 실행은 `button`, 위치 이동은 `a` 또는 Next.js `Link`를 사용한다. 클릭 가능한 `div`를 기본 인터랙션으로 사용하지 않는다.
- **[필수]** 적절한 제목 계층, 의미 있는 이미지 `alt`, 폼 `label`, 키보드 이용 가능성, 포커스 표현을 고려한다. ARIA는 필요한 경우에 정확히 사용한다.
- **[필수]** 공개 웹페이지는 검색엔진이 중요한 본문을 이해할 수 있는 HTML 구조와 렌더링 방식을 우선한다.

## 6. 데이터·타입·로직 계층

| 폴더 | 책임 | 기본 예시 |
|---|---|---|
| `_data` | 정적 카드·FAQ·목록·페이지 콘텐츠 데이터 | `_data/products/list/cards.ts` |
| `_types` | 여러 파일에서 공유하는 타입·인터페이스 | `_types/products/product.ts` |
| `_utils` | React·DB·네트워크와 무관한 순수 계산·변환 | `_utils/format-currency.ts` |
| `_lib` | 외부 SDK, HTTP·DB 클라이언트, 인증 설정, SEO 생성·환경변수·기술 인프라 | `_lib/http/client.ts`, `_lib/seo/metadata.ts` |
| `_hooks` | React 상태·이펙트·브라우저 동작을 재사용하는 Hook | `_hooks/use-scroll-position.ts` |
| `_services` | 상품 조회·주문 생성·권한 검사 등 비즈니스 흐름 | `_services/products/get-products.ts` |
| `_schemas` [선택] | 복잡하거나 재사용하는 입력 검증 | `_schemas/auth/signup.ts` |
| `_styles` [선택] | 전역 스타일·토큰이 복잡해진 후의 역할별 분리 | `_styles/colors.css` |

- **[필수]** UI 파일에 대량의 정적 반복 콘텐츠를 하드코딩하지 않고 `_data`로 분리한다. 한 번만 사용하는 짧은 제목 등까지 무조건 데이터로 옮기라는 뜻은 아니다.
- **[필수]** 여러 파일에서 사용되는 타입은 `_types`로 이동한다. **한 컴포넌트 전용 Props 타입**은 해당 파일에 둔다.
- **[필수]** `_data`가 `_ui`의 내부 타입에 의존하지 않도록 한다.
- **[기본]** 작은 순수 함수는 `_utils`에 직접 배치하고, 관련 파일이 늘어나면 `_utils/validation/...`처럼 묶는다. 다른 역할 폴더도 같은 원칙을 사용한다.
- **[필수]** `_services`는 비즈니스 작업과 권한을 담당하고, `_lib`는 기술적 연결을 담당한다. API 라우트에서 모든 비즈니스 로직을 직접 구현하지 않는다.
- **[선택]** 복잡성·실제 재사용이 생기기 전에는 `_schemas`, `_styles` 등 새 폴더를 미리 만들지 않는다.

## 7. 의존성 및 Import·Export

### 7.1 의존성 방향 — 엄격 적용

| 가져오는 쪽 | 주로 참조 가능한 쪽 | 금지 대표 사례 |
|---|---|---|
| `page.tsx` | 해당 `_ui`, 메타데이터·서비스 등 필요한 하위 역할 | 업무 전체를 진입 파일에 집중 |
| `_ui` | `_components`, `_data`, `_hooks`, `_services`, `_lib`, `_utils`, `_types` | 무관한 다른 페이지 내부 구현 직접 의존 |
| `_components` | `_hooks`, `_utils`, `_types`, 기타 적절한 공통 컴포넌트 | 특정 페이지 `_ui` 및 페이지 전용 데이터 의존 |
| `_data` | `_types` | `_ui`, `_components`, Hook 참조 |
| `_types` | 다른 공유 타입 | UI·서비스 런타임 로직 참조 |
| `_utils` | 다른 순수 함수, 공유 타입 | React UI·서비스·인프라 직접 참조 |
| `_lib` | `_utils`, `_types`, 기술 SDK | `_ui`, `_components`, `_hooks` 참조 |
| `_services` | `_lib`, `_utils`, `_types`, 적절한 다른 서비스 | `_ui`, `_components`, `_hooks` 참조 |
| `_hooks` | `_services`, `_lib`, `_utils`, `_types`, 다른 Hook | `_ui` 또는 페이지 조립 코드 참조 |

이 표는 **역할과 관계를 판단하기 위한 기준**이다. 서버 전용 모듈의 클라이언트 import처럼 프레임워크 상 안전하지 않은 참조는 위 표에서 가능해 보이더라도 금지한다. 스키마의 공유/순수 검증 의존성도 같은 원칙을 따른다.

- **[필수]** 순환 참조 금지. 의존성 위반 시 적절한 공통 타입·함수·컴포넌트로 분리한다.
- **[필수]** 공유 UI는 가능하면 Props로 필요한 데이터를 전달받고 특정 서비스에 강결합되지 않도록 한다.

### 7.2 Import 경로

- **[필수]** 같은 기능 폴더 및 같은 페이지 내부의 가까운 파일 간 참조는 **상대 경로**를 우선한다.
- **[필수]** `_ui`에서 `_components`·`_data`·`_types` 등 **다른 최상위 역할 폴더로 이동하는 참조**는 `@/app/...` **절대 별칭 경로**를 사용한다.
- **[필수]** 타입만 가져오면 `import type`을 사용한다.

```ts
import ProductCard from './card';
import ProductsHeroSection from '../hero/section';
import Button from '@/app/_components/button';
import type { Product } from '@/app/_types/products/product';
```

### 7.3 Export 정책

- **[필수]** `_ui`, `_components`의 **대표 React 컴포넌트**는 `export default`를 사용한다.
- **[필수]** `_utils`, `_hooks`, `_lib`, `_services`, `_data`, `_types`의 함수·값·타입은 **Named Export**가 기본이다.
- **[필수]** `page.tsx`, `layout.tsx`, `error.tsx` 등은 Next.js가 요구하는 Export 형식을 우선한다.
- **[기본]** 주 역할이 다른 여러 주요 컴포넌트를 하나의 파일에 넣지 않는다.

### 7.4 `index.ts` 배럴 파일

- **[기본]** 개별 파일 직접 import가 원칙이다.
- **[선택]** 여러 모듈의 명확한 **공개 인터페이스**가 필요할 때만 배럴을 만든다.
- **[필수]** `_ui`에서는 `overview.tsx`, `section.tsx` 등을 직접 참조하고 배럴을 기본적으로 사용하지 않는다.
- **[기본]** `export *`보다 명시적 `export { name } from './file'`을 우선한다.
- **[필수]** 배럴이 순환 참조 또는 서버/클라이언트 경계 혼합을 만들지 않도록 한다.

## 8. 네이밍 규칙

| 대상 | 규칙 | 예시 |
|---|---|---|
| 폴더·파일 | `kebab-case` | `filter-panel.tsx` |
| 경로만으로 역할이 명확한 파일 | 짧은 이름 우선 | `section.tsx`, `card.tsx`, `overview.tsx` |
| 같은 폴더의 여러 유사 역할 | 필요한 경우 이름 구체화 | `featured-card.tsx` |
| React 컴포넌트 | 의미 있는 `PascalCase` | `ProductsListSection` |
| 일반 함수 | 동사 중심 `camelCase` | `getProducts`, `formatCurrency` |
| 변수·지역 const | `camelCase` | `selectedProduct` |
| 타입·인터페이스·Enum | `PascalCase` | `Product`, `ProductStatus` |
| Props 타입 | 컴포넌트명 + `Props` | `ProductCardProps` |
| Custom Hook | `use` + `PascalCase` | `useProductFilter` |
| 이벤트 핸들러 | `handle` + 동작 | `handleSubmit` |
| 이벤트 Props | `on` + 동작 | `onSubmit` |
| Boolean | `is`/`has`/`can` 등 상태 의미 | `isLoading`, `hasError` |
| 모듈 수준 고정 설정 상수 | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| 일반 지역 상수 | `camelCase` | `visibleProducts` |
| API 오류 코드 | `UPPER_SNAKE_CASE` | `PRODUCT_NOT_FOUND` |
| DOM ID | 의미 있는 `kebab-case` | `products-list-card-123` |

프레임워크가 지정한 `page.tsx`, `layout.tsx`, `route.ts`, `loading.tsx`, `error.tsx`, `not-found.tsx` 등은 예외가 아니라 프레임워크 표준 파일명이다.

## 9. React·Next.js 데이터 흐름과 상태

### 9.1 Server/Client 경계 (Next.js 전용)

- **[필수]** Server Component를 기본으로 한다.
- **[필수]** 상태·이벤트 핸들러·브라우저 API 등 클라이언트 실행이 필요한 **최소 범위**에만 `'use client'`를 선언한다.
- **[기본]** `page.tsx`, `overview.tsx`, `section.tsx`는 특별한 사유가 없으면 Server Component로 유지하고 상호작용 영역을 별도 Client 컴포넌트로 분리한다.
- **[필수]** Client 경계가 import한 코드 전체의 클라이언트 의존성에 영향을 줄 수 있음을 고려한다. 서버 전용 코드와 비밀 정보를 Client에 포함하지 않는다.
- **[필수]** Server → Client Props는 해당 프레임워크에서 허용되는 직렬화 조건을 준수한다. 서버 작업은 적절한 서버 경계에서 실행한다.

### 9.2 Props·공유 상태

- **[기본]** 데이터는 상위 → 하위로 **명시적 Props**, 사용자 이벤트는 하위 → 상위 **콜백 Props**로 전달한다.
- **[기본]** 상태는 **실제 사용하는 가장 작은 범위**가 소유한다. 두 컴포넌트가 공유해야 할 때 가장 가까운 공통 상위로 끌어올린다.
- **[선택]** 여러 계층의 과도한 Props Drilling 또는 복잡한 페이지·전역 공유가 실제 생길 때 Context / 전역 상태 도구를 도입한다.
- **[필수]** Custom Hook을 여러 곳에서 호출한다고 공유 상태가 자동 생성되는 것은 아니라는 점을 구분한다.

### 9.3 상태 종류별 Source of Truth

| 상태 | 기준값과 저장 위치 |
|---|---|
| 일시적 UI 상태 | 가까운 React 로컬 상태 |
| URL로 공유·복원할 탐색 상태 | URL Path / Search Params |
| 서버의 영속 데이터 | 서버·데이터베이스 및 그 데이터의 조회 캐시 |
| 파생 값 | 기존 상태로부터 계산, 불필요한 중복 저장 배제 |

- **[필수]** URL과 React 상태, 전역 Store에 동일한 값을 무조건 복사하지 않는다.
- **[기본]** 서버 데이터와 클라이언트 UI 상태를 다른 생명주기로 취급한다.

## 10. 캐싱·갱신·실패·상태 UI

### 10.1 서버 데이터 정책

- **[필수]** 캐시는 공개 여부·민감도·최신성·변경 빈도에 맞춰 **데이터별로 선택**한다.
- **[기본]** 변경이 적은 공개 콘텐츠는 캐시 활용, 사용자별 민감 데이터나 결제·재고 등 중요한 정보는 최신성·재확인을 우선한다.
- **[필수]** Mutation 성공 후 **관련된 캐시와 화면만** 무효화·재검증한다.
- **[필수]** 조회 실패·변경 실패를 구분하고, 필요한 사용자 안내·상태 복원·안전한 재시도 경로를 제공한다.
- **[필수]** 재시도는 멱등성과 중복 실행 위험을 검토한다.
- **[기본]** 특정 버전의 Next.js 캐시 API를 범용 규칙으로 고정하지 않고 프로젝트 버전/설정에 맞춰 구현한다.

### 10.2 로딩·오류·빈 데이터

```text
(site)/products/
├── page.tsx
├── loading.tsx           # 필요할 때: 라우트 로딩 진입점
├── error.tsx             # 필요할 때: 라우트 Error Boundary (Client)
└── not-found.tsx         # 필요할 때: 404 UI

_ui/products/list/
├── section.tsx
├── loading.tsx           # 필요할 때: 섹션 전용 표시 UI
├── error.tsx             # 필요할 때: 섹션 오류 메시지 UI
└── empty.tsx             # 필요할 때: 결과가 0개인 상태
```

- **[필수]** 라우트 범위의 오류·로딩은 프레임워크의 특수 파일을 적절히 사용한다.
- **[기본]** 특정 섹션에 독립적인 상태가 필요한 경우에만 `_ui` 내부에 별도 UI 파일을 만든다.
- **[필수]** `_ui/.../error.tsx`는 **파일명만으로 자동 Error Boundary가 되지 않는다.** 필요하면 실제 경계를 별도로 구성한다.
- **[필수]** 정상 조회 결과 0건인 `Empty`와 없는 리소스의 `404`를 구별한다.
- **[기본]** 여러 페이지에서 재사용하는 Spinner·ErrorMessage 등은 `_components`로 승격한다.

## 11. Tailwind 및 CSS 작성 규칙

### 11.1 기본 스타일링 전략

- **[필수]** Tailwind CSS를 기본 스타일링 수단으로 사용한다.
- **[선택]** 복잡한 선택자·애니메이션 등 Tailwind만으로 관리하기 불편한 스타일은 CSS Modules 등으로 보완한다.
- **[필수]** 개인 규칙에 맞는 클래스 순서와 줄바꿈 방식을 유지한다. 외부 자동 정렬 도구가 이를 무조건 바꾸게 두지 않는다.
- **[기본]** DOM `id`는 식별·연결 용도이고 CSS 스타일은 클래스 중심으로 관리한다.

### 11.2 확정한 12개 클래스 그룹 — **순서 고정**

| 순서 | 그룹 | 그룹 내부 우선순위 |
|---|---|---|
| **01** | **Position / Layer** | `relative`·`absolute` 등 위치 방식 → `z-*` → `left-*` → `right-*` → `top-*` → `bottom-*` |
| **02** | **Margin** | `m-*` → `mx-*` → `my-*` → `ml-*` → `mr-*` → `mt-*` → `mb-*` |
| **03** | **Width / Height / Size** | `max-w-*` → `max-h-*` → `w-*` → `h-*` |
| **04** | **Padding** | `p-*` → `px-*` → `py-*` → `pl-*` → `pr-*` → `pt-*` → `pb-*` |
| **05** | **Display / Layout** | `block`·`hidden`·`flex`·`grid` → 방향·열 수 등 레이아웃 제어 |
| **06** | **Alignment / Gap** | `justify-*` → `items-*` → `gap-*` (그 외 정렬·간격은 인접 배치) |
| **07** | **Rounded** | `rounded-*` |
| **08** | **Overflow** | `overflow-*` |
| **09** | **Border / Ring** | `border-*` → `ring-*` |
| **10** | **Background** | `bg-*` |
| **11** | **Typography** | **`break-*` → `font-*`(weight) → `text-*`(size) → `leading-*` → `tracking-*` → `text-*`(align) → `text-*`(color)** |
| **12** | **Effects / Interaction** | `shadow-*`, `opacity-*`, `transition-*`, `cursor-*` 등 |

> **중요:** 이는 Tailwind의 기본 정렬이 아니라 **개발자가 직접 확정한 개인 정렬 규칙**이다. `font-*`가 폰트 패밀리를 뜻하거나 `text-*`가 크기·정렬·색상을 뜻하는 경우, 의미를 판별해 알맞은 위치에 둔다. 위에서 명시하지 않은 `min-w`, `min-h`, `inset`, transform 등은 성격이 가장 가까운 그룹에 배치하되, 새로운 세부 순서를 이미 확정한 것처럼 주장하지 않는다.

**Margin / Padding 세부 규칙:** 전체값(`m-*`, `p-*`)이 있다면 먼저, **가로(`mx`, `px`) → 세로(`my`, `py`) → 개별 left → right → top → bottom** 순이다. 동일 속성을 불필요하게 덮어쓰는 중복 클래스는 지양한다.

**Typography 세부 규칙:** `break → font-weight → font-size → line-height → letter-spacing → text-align → font-color` 순서가 **최신 확정안**이다. 과거의 `tracking → leading → font` 순서는 **폐기**한다.

### 11.3 줄바꿈·Variant·템플릿 리터럴

- **[필수]** `className="..."`보다 **``className={`...`}``** 를 우선 사용한다.
- **[필수]** 접두사가 없는 **기본 모바일 클래스는 한 줄**로 연속 작성한다. 기본 클래스의 중간에서 그룹마다 개행하지 않는다.
- **[필수]** **`before:`는 새 줄, `after:`는 별도의 새 줄**에 작성한다.
- **[필수]** **`hover:`, `focus:`, `active:` 등 상태 종류별로 각각 새 줄**에 작성한다. `focus-visible`, `disabled` 등도 같은 취지로 처리한다.
- **[필수]** **`sm:`, `md:`, `lg:`, `xl:`, `2xl:` 등 반응형은 브레이크포인트별 새 줄**에 작성한다. 기본 모바일 → 작은 화면 → 큰 화면 순이다.
- **[필수]** **각 줄 내부에도 12단계 정렬 순서**를 적용한다. 혼합 Variant는 해당 기능의 관련 줄에 가까이 두고 가독성을 유지한다.
- **[기본]** 기존 컴포넌트의 클래스 정렬만 수정할 때 디자인 값이나 동작은 임의로 변경하지 않는다.

```tsx
// 순서 예시: 기본(모바일) / before / after / hover / focus / active / md / xl
<section
  id="products-hero"
  className={`relative z-10 mx-auto my-5 max-w-6xl w-full px-5 py-8 flex flex-col justify-center items-center gap-4 rounded-xl overflow-hidden border border-gray-200 bg-white break-keep font-semibold text-xl leading-7 tracking-tight text-center text-primary shadow-sm transition-colors
    before:absolute before:left-0 before:bottom-0 before:w-full before:h-px before:bg-gray-200
    after:absolute after:right-0 after:top-0 after:w-px after:h-full after:bg-gray-200
    hover:bg-gray-50
    focus-visible:ring-2 focus-visible:ring-primary
    active:opacity-90
    md:my-8 md:px-8 md:py-10 md:gap-6 md:text-2xl
    xl:my-12 xl:max-w-7xl xl:px-12 xl:py-16 xl:gap-8 xl:text-4xl`}
>
  <h2>상품 안내</h2>
</section>
```

> 예시는 그룹 순서와 개행법을 나타내기 위한 것이며 모든 페이지에 위 Tailwind 속성을 넣어야 한다는 뜻은 아니다.

### 11.4 조건부 클래스 및 Variant

- **[기본]** 간단한 조건부 스타일은 기존 ``className={`...`}`` 내부에서 직접 처리한다.
- **[선택]** 조건이 많아져 읽기 어려울 때만 `cn()` 헬퍼와 필요한 충돌 해결을 도입한다.
- **[선택]** 반복해서 사용하는 복잡한 Variant는 별도 스타일 구조 또는 CVA 같은 도구를 검토한다.
- **[필수]** 조건부 클래스가 추가되어도 의미적 그룹 순서·Variant별 개행을 최대한 유지한다.
- **[필수]** Prettier Tailwind 자동 정렬 플러그인이 개인 그룹 순서를 재배열하는 경우 적용하지 않거나 호환되는 대안을 사용한다.

## 12. 디자인 토큰과 전역 스타일

### 12.1 토큰 파일 배치

- **[기본]** 디자인 시스템이 확립되지 않은 프로젝트는 **Tailwind 기본값**으로 시작한다.
- **[기본]** 초기 색상·폰트·간격·반경 등 전역 토큰은 `globals.css`에서 관리한다.
- **[선택]** 규모와 책임이 커질 때만 `_styles/colors.css`, `_styles/typography.css` 등으로 분리하고 `globals.css`를 진입점으로 유지한다.
- **[선택]** 반복되는 프로젝트 특수값이 생기면 커스텀 토큰을 도입하거나 필요한 기본값을 재정의한다. 한 곳에서만 필요한 특수값은 적절할 때 Tailwind 임의값을 사용해도 된다.
- **[필수]** 이미 확립된 프로젝트 디자인 시스템이 있다면 승인된 토큰과 규칙을 우선한다.

### 12.2 색상 3계층 병행

1. **의미 기반 토큰:** 주요 역할에 `primary`, `surface`, `border-default` 등 사용 (`bg-primary`, `text-primary`).
2. **Tailwind 기본 팔레트:** `blue-50`~`blue-950`, `gray-200` 등 기본 제공값 직접 사용 가능. 필요 시 개별 토큰 재정의 가능.
3. **브랜드 팔레트:** `ooo-blue-500`처럼 실제 회사·브랜드 식별자를 접두사로 사용한 고유 색상 등록 가능. 여기서 `ooo`는 **예시 자리표시자**다.

```css
/* src/app/globals.css — Tailwind CSS v4 예시 */
@import "tailwindcss";

@theme {
  --color-ooo-blue-500: #155e75;
  --color-ooo-blue-700: #164e63;
  /* 필요할 때만 기존 팔레트를 재정의한다. */
  /* --color-blue-500: #2563eb; */
}

@theme inline {
  --color-primary: var(--color-ooo-blue-500);
  --color-primary-strong: var(--color-ooo-blue-700);
}
```

실제 프로젝트명·색상값은 프로젝트별로 지정한다. 임의로 회사명이나 색상값을 확정하지 않는다.

## 13. SEO 중앙 관리

### 13.1 목표와 적용 범위

- **[필수]** 공개 콘텐츠는 SEO를 처음부터 고려한다. **검색 노출 순위를 보장한다는 뜻은 아니다.**
- **[필수]** 의미 있는 HTML, 대표 제목 `h1`과 논리적 `h2` 이하 구조, 고유 메타 Title/Description, 의미 있는 링크 텍스트, 이미지 `alt`, 성능과 색인 가능성을 검토한다.
- **[기본]** 중요한 콘텐츠는 서버가 렌더링 또는 사전 렌더링해 검색엔진이 읽기 쉬운 형태로 제공한다.
- **[필수]** 비공개·관리자·인증 전용 페이지의 색인 여부는 공개 페이지와 별도 정책으로 관리한다.
- **[선택]** 적절한 Canonical, Open Graph, Sitemap, Robots, 구조화 데이터는 실제 페이지 특성에 맞춰 사용한다.

### 13.2 SEO는 `_lib/seo`에서 중앙 관리, 페이지는 적용 책임

```text
_lib/seo/
├── config.ts                  # 사이트 URL, 이름, 대표 이미지 등
├── metadata.ts                # 공통 Metadata 생성 함수
├── pages.ts                   # 정적 페이지별 SEO 설정
└── structured-data/          # 실제 여러 종류가 필요해진 경우에만
    ├── organization.ts
    └── product.ts

(site)/products/page.tsx      # 자신에게 해당하는 설정 적용
app/layout.tsx               # 사이트 공통 메타데이터
app/sitemap.ts               # 프레임워크 진입점 (필요 시)
app/robots.ts                # 프레임워크 진입점 (필요 시)
```

- **[필수]** 공통 메타데이터 설정·생성 로직은 `_lib/seo`에 중앙 관리한다.
- **[필수]** 각 페이지에서 해당 SEO 설정을 **import·적용**한다. Next.js의 `metadata` 또는 `generateMetadata`를 내보내는 진입점은 프레임워크가 요구하는 위치에 유지한다.
- **[기본]** 동적 상품·게시글 등은 실제 콘텐츠를 바탕으로 메타데이터를 생성한다. 모든 동적 항목을 `pages.ts`에 미리 등록하지 않는다.
- **[필수]** JSON-LD 구조화 데이터의 **생성 함수는 `_lib/seo`에 공통 관리**, 필요한 페이지에만 적용한다. 표시된 실제 내용과 일치하는 데이터만 넣는다.
- **[필수]** JSON-LD 출력은 안전하게 직렬화하고 사용자 입력을 스크립트에 무검증으로 삽입하지 않는다.
- **[필수]** 본문용 정적 데이터는 `_data`, 화면은 `_ui`에 유지한다. `_lib/seo`는 **SEO 설정 및 기술적 생성 로직**을 담당하는 예외가 아닌 정식 책임 영역이다.

```tsx
// (site)/products/page.tsx — 구조 예시
import ProductsOverview from '@/app/_ui/products/overview';
import { createMetadata } from '@/app/_lib/seo/metadata';
import { SEO_PAGES } from '@/app/_lib/seo/pages';

export const metadata = createMetadata(SEO_PAGES.products);

export default function ProductsPage() {
  return <ProductsOverview />;
}
```

## 14. 입력 검증·API·인증·환경변수

### 14.1 입력 검증

- **[필수]** 신뢰할 수 없는 입력은 **서버에서 검증**한다. 클라이언트 검증만으로 통과시키지 않는다.
- **[기본]** 간단한 입력은 HTML 기본 검증과 필요한 순수 함수로 처리한다.
- **[선택]** 복잡하거나 반복되는 검증은 Zod 등 스키마 및 `_schemas`로 분리한다. 단순 검증을 위해 미리 스키마 폴더를 만들지 않는다.
- **[기본]** 범용 검증 함수는 `_utils/validation` 등에서 관리한다.
- **[기본]** 스키마로부터 타입을 추론할 수 있으면 불필요하게 동일 타입을 중복 선언하지 않는다.

### 14.2 API 요청·응답

- **[필수]** Next.js API 엔드포인트는 `(api)/api/.../route.ts`로 구분한다.
- **[필수]** API 진입점에서 입력 검증·인증 호출·응답 구성을 하되 업무 로직은 `_services`를 사용한다.
- **[필수]** **성공 JSON 형태는 목적에 따라 유연하게**, **오류 JSON 형식은 표준화**한다.

**성공 예시 — HTTP 200 OK**

```json
{
  "products": [{ "id": 1, "name": "상품 A" }],
  "total": 1
}
```

**실패 예시 — HTTP 404 Not Found**

```json
{
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "상품을 찾을 수 없습니다."
  }
}
```

- **[필수]** 성공/실패에 적절한 HTTP 상태(`200`, `201`, `204`, `400`, `401`, `403`, `404`, `409`, `500` 등)를 사용한다. `204`에는 JSON 본문을 강제하지 않는다.
- **[필수]** 오류 코드는 일관된 `UPPER_SNAKE_CASE`; 내부 스택·토큰·민감한 정보를 응답하지 않는다.
- **[필수]** 이 **HTTP 응답 계약을 내부 서비스 함수나 Server Action 반환값에 무조건 강제하지 않는다.**

### 14.3 인증·인가 및 서버 경계

- **[필수]** 로그인·세션·인증 SDK 연동은 `_lib/auth` 등 공통 기술 계층에서 관리한다.
- **[필수]** 기능·리소스별 **실제 권한 검사**는 해당 `_services`에서 수행한다. 공통 권한 함수가 실제로 필요해지면 공통화한다.
- **[필수]** UI에서 버튼을 숨겼다는 이유로 서버 인증·인가를 생략하지 않는다. API, Server Action 등 외부 호출 경계를 독립적으로 보호한다.
- **[선택]** 역할 정책이 복잡해질 때 중앙 권한 정책 모듈로 확장할 수 있으나 처음부터 강제하지 않는다.

### 14.4 환경변수

```text
.env.example               # 키 이름과 안전한 예시; 저장소 포함 가능
.env.local                 # 실제 로컬 비밀값; 저장소 제외
_lib/env/
├── server.ts              # 서버 전용 환경변수 접근
└── public.ts              # 공개 가능한 설정 (필요 시 분리)
```

- **[필수]** 비밀키·DB 접속정보는 서버 전용으로 보관하고 클라이언트·로그·Git에 노출하지 않는다.
- **[필수]** `NEXT_PUBLIC_`는 공개 가능한 값에만 사용한다. 클라이언트 번들에 포함될 수 있음을 고려한다.
- **[기본]** 환경변수 접근을 `_lib/env`에서 중앙 관리하며 필수값 누락 등을 검증한다. 프로젝트가 아주 단순할 때 불필요한 다중 파일을 강제하지 않는다.
- **[선택]** 검증할 설정이 복잡해지면 Zod 등 스키마를 도입한다. `server-only` 등 서버 전용 경계 표시를 사용해 잘못된 클라이언트 import를 예방한다.

## 15. ChatGPT 코드 변경 계약

### 15.1 작업 전에 할 일 — 필수

1. 요청과 관련된 파일·인접 기능·실제 공통 컴포넌트를 먼저 확인한다.
2. 이 문서의 디렉터리 책임·의존성·Server/Client·Export·ID·Tailwind·SEO 규칙을 점검한다.
3. 기존 코드의 역할·동작·스타일을 보존하면서 수정 대상을 결정한다.
4. 애매한 디자인 좌표·비즈니스 정책·존재하지 않는 라우트·민감 설정을 **추측해 확정하지 않는다.** 필요한 경우 미구현 영역은 표시하고 제안한다.

### 15.2 변경 권한 — 마무리 인터뷰 1/3 확정

- **[필수]** 요청을 해결하고 **이미 확정된 규칙을 지키는 데 필요한 관련 코드**는 적극 개선할 수 있다. 예: 섹션 분리, 잘못된 참조 수정, 역할별 파일 이동, 중복 관련 코드 공통화.
- **[필수]** 기존 규칙 **밖의 새로운 설계 방법, 추가 라이브러리, 큰 구조 개편, 요청과 무관한 리팩토링**은 먼저 **제안**한다. 승인 없이 시행하지 않는다.
- **[필수]** 기존 동작·디자인·API 계약·SEO·DOM ID를 의도하지 않게 변경하지 않는다.
- **[기본]** 규칙을 준수하는 과정에서 영향을 받은 import·타입·Props·호출부는 필요한 범위에서 함께 수정한다.

### 15.3 신규 기능 — 마무리 인터뷰 2/3 확정

1. 기존 폴더 구조·재사용 컴포넌트를 먼저 확인한다.
2. 독립 섹션은 `_ui/<페이지>/<기능>/section.tsx`로 분리한다.
3. 독립 역할의 하위 UI만 파일로 추가하고, 단순한 요소마다 폴더를 만들지 않는다.
4. 정적 데이터는 `_data`, 공유 타입은 `_types`, 비즈니스 동작은 `_services`, 순수 함수는 `_utils` 등 기존 책임 폴더로 배치한다.
5. 여러 페이지에서 동일 책임이 실제 재사용되면 `_components` 승격을 검토한다.
6. 공통 계층이 페이지 내부 구현에 의존하거나 순환 참조하지 않도록 한다.

### 15.4 검증 및 결과 보고 — 마무리 인터뷰 3/3 **B안 확정**

**검증 순서(작업 영향 범위에 따라 수행):**

1. 아키텍처 준수 확인(책임 위치, ID, 의존성, Export, Server/Client, Tailwind 규칙, 공개 페이지 SEO).
2. TypeScript 타입 검사(프로젝트에 구성된 명령을 우선 사용).
3. ESLint 또는 프로젝트 린터 확인.
4. 관련 테스트가 이미 있으면 실행.
5. 필요하거나 영향이 큰 수정이면 Next.js 프로덕션 빌드 등 수행.

**[필수]** 실행할 수 없었거나 실제 실행하지 않은 검사는 **'미실행' 또는 '검증 불가'로 정확하게 기록**한다. 도구·환경이 없는데 통과라고 주장하지 않는다. 검증 실패 시 원인과 범위를 보고한다.

**완료 보고 형식:**

```text
[작업 요약]
- 요청한 기능과 실제 수행 내용

[변경 파일]
- 신규 / 수정 / 삭제 경로별 목록

[아키텍처 적용]
- 분리·승격·의존성·ID·스타일·SEO 등 중요 판단

[검증 결과]
- TypeScript: 통과 / 실패 / 미실행
- ESLint: 통과 / 실패 / 미실행
- 관련 테스트: 통과 / 실패 / 미실행 / 해당 없음
- Build: 통과 / 실패 / 미실행

[남은 위험·미확정 사항]
- 구현하지 않았거나 확인할 수 없는 사항

[추가 개선 제안]
- 기존 규칙을 넘어서는 개선은 시행하지 않고 제안
```

## 16. 기능 추가 표준 예시

**요구:** 상품 페이지에 FAQ 섹션 추가. 실제 FAQ 데이터는 아직 주어지지 않았다고 가정한다.

```text
src/app/
├── (site)/products/page.tsx
├── _ui/products/
│   ├── overview.tsx                  # 기존 파일에 FAQ 조립 추가
│   ├── hero/section.tsx
│   ├── list/section.tsx
│   └── faq/
│       ├── section.tsx               # 새 섹션
│       └── item.tsx                  # 독립 역할이 필요할 때
├── _data/products/
│   └── faq.ts                         # 실제 콘텐츠가 준비되었을 때
├── _types/products/
│   └── faq.ts                         # 여러 파일에서 공유할 때만
└── _components/
    └── ...                            # 다른 페이지에서 실제 재사용되는 UI만
```

- `page.tsx`는 대체로 그대로 두고 `_ui/products/overview.tsx`에 새 섹션을 추가한다.
- 섹션은 `<section id="products-faq">`처럼 의미 있는 최상위 ID를 갖는다.
- 반복 UI가 실제로 독립된 역할이면 `item.tsx`로 분리하고 `id`를 Props로 전달한다.
- FAQ 내용이 없으면 내용을 꾸며 넣지 않고 데이터 정의 대기 사항으로 남긴다.
- 기존 SEO 설정과 페이지 제목 구조에 영향이 있는지 확인한다. 구조화 데이터는 페이지 콘텐츠와 검색엔진 정책에 적절한 경우에만 적용한다.
- 변경 보고서에 실제 실행한 검증과 미실행 검증을 구분한다.

## 17. 완료 체크리스트

### 구조·코드

- [ ] `(site)` 등을 `_ui`, `_data`에 불필요하게 반복하지 않았는가?
- [ ] `page → overview → section → 독립 블록` 원칙을 적용했는가?
- [ ] 페이지 전용 `_ui`와 재사용 `_components`의 역할이 구분되는가?
- [ ] `_data`, `_types`, `_utils`, `_lib`, `_hooks`, `_services`의 책임이 섞이지 않는가?
- [ ] 불필요한 파일·폴더·배럴·라이브러리를 만들지 않았는가?
- [ ] 순환 참조와 반대 방향 의존성이 없는가?
- [ ] 상대/절대 import, `import type`, Default/Named Export 규칙을 지켰는가?
- [ ] 파일명, 타입명, 핸들러 등 네이밍이 일관적인가?

### UI·스타일·검색

- [ ] 실제 최상위 DOM이 있는 독립 UI에 고유 `id`가 있는가?
- [ ] 시맨틱 HTML과 기본 접근성이 적절한가?
- [ ] 서버 우선·최소 Client 경계를 지켰는가?
- [ ] Tailwind **12개 그룹 내부 정렬 및 Typography 최신 순서**를 지켰는가?
- [ ] 기본 모바일 한 줄, `before`/`after`/상태/브레이크포인트별 개행을 지켰는가?
- [ ] ``className={`...`}`` 우선 규칙과 기존 디자인 유지 원칙을 지켰는가?
- [ ] 디자인 토큰은 필요한 만큼만 도입했는가?
- [ ] 공개 페이지의 Metadata·본문 계층·SEO·이미지·링크를 확인했는가?

### 데이터·보안·AI 작업

- [ ] 서버 입력 검증·기능별 권한 검사·환경변수 비공개 경계를 지켰는가?
- [ ] 성공 API 응답은 목적에 맞게, 오류 응답은 표준 형식으로 반환하는가?
- [ ] 변경 후 데이터 캐시를 필요한 부분만 갱신하는가?
- [ ] 요청 범위 안의 적극 개선과 규칙 밖의 제안을 구분했는가?
- [ ] 검증 도구의 **실제 실행/미실행/실패** 결과를 구분해 보고했는가?

## 18. 미확정·후속 업데이트

**v1.0 범위에 포함되지 않는 상세 정책:**

| 주제 | 추후 정할 항목 | 상태 |
|---|---|---|
| 테스트 | 단위·통합·E2E 범위, 도구, 테스트 파일 배치, CI 통과 기준 | **[보류]** |
| 로깅·모니터링 | 미응답 **인터뷰 32번**: `console`/Logger, 로그 수준, 외부 모니터링 도입 | **[보류]** |
| 배포·CI/CD | 환경별 배포·롤백·브랜치·릴리스 정책 | **[보류]** |
| 보안 심화 | 위협 모델, 레이트 리밋, 감사 로그, 보안 자동 검사 (기본 인증·인가·환경변수 보안은 확정) | **[보류]** |
| 성능 | Web Vitals, 번들·이미지·폰트 성능 예산, 측정·최적화 | **[보류]** |

**업데이트 시 원칙:** 새 정책은 기존 v1.0 원칙을 무조건 덮어쓰지 않고, 확정·권장·예외·보류 여부를 표시한다. 변경 이유, 영향 범위, 이전 규칙과의 관계를 남긴다. 기술 버전이나 프로젝트 특수 조건에 따른 구현 사항은 범용 개인 규칙과 구분한다.

---

## 결정 출처 요약

이 문서는 **청맥이라는 특정 프로젝트의 구조 규정이 아니라, 해당 코드를 참고자료로 삼아 대화 인터뷰에서 개발자가 직접 결정한 개인 표준**이다. 주요 결정의 추적표는 다음과 같다.

| 인터뷰/결정 | 최종 선택 |
|---|---|
| 1~5 | 최소 단위 컴포넌트 분리, 최상위 DOM ID, `_ui`/`_components` 분리, 기능별 폴더, `_data`/`_types` 역할별 분리 |
| 6~8 | `_utils`·`_lib`·`_hooks`·`_services` 역할 구분, 필요할 때 하위 폴더, 엄격한 의존성 방향 |
| 9~14 | 가까운 내부는 상대 import, 역할 경계는 절대 import, Server 우선, 경로 중심 네이밍, 역할별 네이밍, UI Default·그 외 Named Export, 선택적 배럴 |
| 15~19 | Props 기본, 최소 범위 상태, UI·URL·서버 Source of Truth 구분, 선택적 캐싱·명시적 실패 처리, 라우트/섹션 상태 UI 분리 |
| 20~24 | Tailwind 중심, 개인 클래스 순서·줄바꿈, `globals.css`부터 토큰 관리, 의미·기본·브랜드 색상 병행, 기본 간격·타이포그래피·반경 우선, 조건부 스타일 필요시 도구화 |
| 25~27 | 시맨틱 HTML·접근성 및 SEO 우선, SEO `_lib/seo` 중앙 관리, JSON-LD 생성 로직 중앙 관리 |
| 28~31 | 선택적 `_schemas`, API 성공 유연·오류 표준, 인증 공통·기능별 인가, `_lib/env` 환경변수 중앙 관리 |
| 마무리 1/3 | 규칙 내 관련 코드 적극 개선, 규칙 밖 개선은 제안 |
| 마무리 2/3 | 기존 구조 활용 + 책임에 따른 신규 파일 분리 |
| 마무리 3/3 | 단계별 검증 + 구조화된 변경 결과 보고(B) |
| 인터뷰 32 | 로깅·모니터링 **미확정**, v1.0 이후로 보류 |

**문서 종료 — v1.0**
