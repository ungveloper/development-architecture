# ChatGPT 개발 작업 지침 — 개인 아키텍처 v1.0

> **용도:** 새 프로젝트 또는 기존 코드 수정 시 ChatGPT 프로젝트 지침·프롬프트에 붙여넣을 짧은 실행 규약.  
> **정식 기준:** `MY_DEVELOPMENT_ARCHITECTURE_V1.0.md` 전체가 우선한다. 이 파일은 요약이며, 충돌 시 정식 문서를 따른다.

## 핵심 원칙

- 이 표준은 **개발자 개인의 아키텍처**이며 특정 프로젝트 구조를 복제하기 위한 것이 아니다.
- 각 책임의 경계를 명확히 하고 컴포넌트는 **의미 있는 최소 단위로 적극 분리**한다. 불필요한 추상화·폴더 깊이는 만들지 않는다.
- Next.js: `(site)`·`(api)`는 라우트 분류용이다. `_ui/site/` 등 중복 분류 폴더는 만들지 않는다.
- `page.tsx → _ui/<page>/overview.tsx → <feature>/section.tsx → 하위 최소 단위 UI` 구조를 기본으로 한다.
- `_ui`는 **페이지 전용**, `_components`는 **실제로 여러 페이지에서 재사용하는 UI**다.
- 정적 데이터 `_data`, 공유 타입 `_types`, 순수 유틸 `_utils`, 기술·SDK `_lib`, React Hook `_hooks`, 비즈니스 로직 `_services`로 책임을 분리한다. 복잡한 입력 스키마 `_schemas`, 추가 전역 CSS `_styles`는 필요할 때만 만든다.
- UI의 실제 최상위 DOM에 고유한 의미적 `id`를 부여한다. 반복 ID 중복 금지. DOM 없는 Fragment/Provider는 예외이고 내부 DOM에는 ID를 강제하지 않는다.
- 시맨틱 HTML과 기본 접근성 준수. 공개 사이트는 SEO 우선: 페이지 메타데이터·제목 계층·링크·이미지·렌더링을 점검한다. 공통 SEO 생성은 `_lib/seo`, 페이지는 이를 적용한다. JSON-LD는 필요한 페이지에서만 사용한다.
- 같은 기능 내부는 상대 import, 역할 폴더 간에는 절대 별칭 import. 타입은 `import type`. 의존성 역전·순환 참조 금지.
- UI의 대표 React 컴포넌트는 Default Export, 함수·Hook·서비스·데이터·타입은 Named Export. `index.ts` 배럴은 명확한 공개 API가 필요할 때만 사용한다.
- 파일·폴더 `kebab-case`, 컴포넌트·타입 `PascalCase`, 함수·변수 `camelCase`, Hook `use*`, Handler `handle*`, Boolean `is*`·`has*`·`can*`, 설정 상수 `UPPER_SNAKE_CASE`.
- Server Component 기본, 필요한 최소 단위에만 `'use client'`. Props 기본, 최소 범위 상태, URL 공유 상태는 URL, 서버 데이터는 서버를 기준으로 한다. 캐싱·갱신은 데이터 성격에 따라 선택한다.
- API 성공 응답은 목적에 따라 설계, 오류는 `{ "error": { "code": "...", "message": "..." } }`로 표준화. 인증은 공통화하되 리소스 권한은 해당 `_services`에서 검증한다. `_lib/env`에서 서버·공개 설정을 구분한다.

## Tailwind 작성 강제 규칙

**클래스 그룹 순서:**

1. Position / Layer (`relative|absolute` → `z` → `left` → `right` → `top` → `bottom`)
2. Margin (`m` → `mx` → `my` → `ml` → `mr` → `mt` → `mb`)
3. Size (`max-w` → `max-h` → `w` → `h`)
4. Padding (`p` → `px` → `py` → `pl` → `pr` → `pt` → `pb`)
5. Display / Layout
6. Alignment / Gap (`justify` → `items` → `gap`)
7. Rounded
8. Overflow
9. Border / Ring
10. Background
11. Typography (**`break` → `font-weight` → `font-size` → `line-height` → `letter-spacing` → `text-align` → `font-color`**)
12. Effects / Interaction

**형식:** ``className={`...`}`` 선호. **모바일 기본 클래스 전체 한 줄**, `before:`와 `after:`는 **각각 새 줄**, `hover:`·`focus:`·`active:` 등은 **상태별 새 줄**, `sm:`·`md:`·`lg:`·`xl:`·`2xl:` 등은 **브레이크포인트별 새 줄**. 각 줄 안에서도 위 그룹 순서를 유지한다. 자동 정렬 플러그인이 개인 순서를 깨면 사용하지 않는다.

**토큰:** Tailwind 기본값 우선 → 반복·특수 요구가 생기면 커스텀 확장. 의미 토큰(`primary`), Tailwind 기본 팔레트(`blue-500`), 브랜드 팔레트(`회사명-blue-500`)를 병행한다. 초기에는 `globals.css`에서 관리하고 규모가 커지면 `_styles`로 분리한다. 간단한 조건부 스타일은 직접, 복잡할 때만 `cn()`/Variant 도구를 도입한다.

## 요청 수행 절차

1. 현 구조·요청 범위·사용 중인 공통 자산을 파악한다.
2. 확정된 구조를 따르는 범위에서 **관련 코드의 적극적인 리팩토링을 허용**한다.
3. 확정 규칙 밖의 개선·새 기술·무관한 대규모 재편은 **변경하지 말고 별도로 제안**한다.
4. 신규 섹션은 기능 폴더의 `section.tsx`로, 독립 UI는 같은 기능 폴더의 작은 파일로 분리한다.
5. 실제 실행 가능한 범위의 **아키텍처 점검 → 타입 검사 → 린트 → 관련 테스트 → 필요한 경우 빌드** 순서로 검증한다.
6. **작업 요약 / 신규·수정·삭제 파일 / 아키텍처 적용 판단 / 검사 결과(통과·실패·미실행) / 잔여 위험 / 추가 개선 제안**을 구분하여 보고한다.
7. 실행하지 않은 검사를 성공했다고 말하지 않는다. 알 수 없는 데이터·디자인·좌표·라우트는 임의로 완성하지 않는다.

## 향후 업데이트 — 아직 강제하지 말 것

테스트 방법·테스트 폴더 규정, 로깅·모니터링(미응답 인터뷰 32번), 배포·CI/CD, 보안 심화, 성능 측정·예산은 v1.0의 **보류** 항목이다. 이미 확정된 기본 서버 검증·인증·인가·환경변수 보호와 혼동하지 않는다.
