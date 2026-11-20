# Personal Development Architecture

개인 개발 아키텍처 및 AI 협업 코딩 규칙을 관리하는 저장소입니다.

이 저장소는 특정 서비스의 소스 코드가 아닌, 여러 프로젝트에 공통으로 적용할 **개인 개발 표준**을 보관합니다.

## 문서

| 파일 | 설명 | 활용 방법 |
| --- | --- | --- |
| [MY_DEVELOPMENT_ARCHITECTURE.md](./MY_DEVELOPMENT_ARCHITECTURE.md) | 최신 정식 아키텍처 표준(현재 v1.0). 폴더·컴포넌트·상태·Tailwind·SEO·API·AI 수정 규칙을 포함 | 개발 설계와 코드 수정의 최우선 기준 |
| [AI_DEVELOPMENT_INSTRUCTIONS.md](./AI_DEVELOPMENT_INSTRUCTIONS.md) | ChatGPT에 전달할 간결한 실행 지침 | ChatGPT 프로젝트 지침에 넣어 활용 |
| [FUTURE_UPDATE_BACKLOG.md](./FUTURE_UPDATE_BACKLOG.md) | 후속 버전에서 논의할 테스트·로깅·배포·보안 심화·성능 항목 | 규칙 업데이트 시 참조 |
| [versions/v1.0.md](./versions/v1.0.md) | v1.0 확정 시점의 변경 전 원본 보관본 | 과거 버전 비교·보존 |

## 다른 프로젝트에서 사용하기

1. 새 프로젝트의 ChatGPT 프로젝트 지침에 `AI_DEVELOPMENT_INSTRUCTIONS.md`의 내용을 반영합니다.
2. 정식 아키텍처 문서 `MY_DEVELOPMENT_ARCHITECTURE.md`의 최신 내용을 첨부하거나 접근 가능한 자료로 제공합니다.
3. 실제 소스 코드도 함께 제공하고, 이 문서의 확정 규칙에 따라 작성·수정하도록 요청합니다.
4. 요약 지침과 정식 문서가 충돌하면 **정식 아키텍처 문서**를 우선합니다.

## ChatGPT에서 사용하는 프롬프트

다른 프로젝트에서 코드를 작성하거나 수정할 때 아래 내용을 복사하여 ChatGPT의 프로젝트 지침 또는 요청 메시지에 사용할 수 있습니다.

```text
다음 GitHub Repository에 정의된 최신 개인 개발 아키텍처를 기준으로 코드를 작성하고 수정해줘.

아키텍처 저장소
https://github.com/ungveloper/development-architecture

AI 개발 지침
https://github.com/ungveloper/development-architecture/blob/main/AI_DEVELOPMENT_INSTRUCTIONS.md

정식 아키텍처 규칙
https://github.com/ungveloper/development-architecture/blob/main/MY_DEVELOPMENT_ARCHITECTURE.md

작업마다 두 문서의 최신 실제 내용을 다시 읽고, 확정된 모든 규칙을 적용해줘.

1. 두 문서가 충돌한다면 정식 아키텍처 문서를 우선한다.
2. 기존 아키텍처 규칙 안에서는 적극적으로 코드를 개선한다.
3. 규칙을 벗어나는 새로운 설계나 추가 개선은 임의로 적용하지 않고 제안한다.
4. 기존 프로젝트 구조를 먼저 분석하고, 재사용 가능한 코드를 우선 확인한다.
5. Tailwind 클래스 정렬, 개행, DOM ID, 컴포넌트 분리, SEO 규칙을 준수한다.
6. 수정 후 변경 파일, 아키텍처 준수 여부, 실제 검증 결과를 보고한다.

GitHub 문서를 읽을 수 없다면 임의로 규칙을 추측하지 말고 접근할 수 없음을 알려줘.
```

> **참고:** GitHub 링크를 제공하는 것만으로 모든 AI 환경에서 파일을 자동으로 읽을 수 있는 것은 아닙니다. 문서에 접근할 수 없다면 Markdown 파일을 직접 첨부하세요. AI 개발 지침은 요약본이며 정식 아키텍처 문서를 우선합니다.

## 적용 범위

- 범용 개인 원칙: 책임 분리, 컴포넌트 최소 단위 설계, 의존성, 네이밍, AI 수정·보고 계약
- 주요 기술 환경: TypeScript, React, Next.js App Router, Tailwind CSS
- 다른 프레임워크를 사용할 때는 프레임워크 전용 파일명보다 설계 원칙을 우선하여 적용합니다.

## 버전 관리

- 현재 정식 버전: **v1.0**
- 최신 규칙의 **고정 경로**는 `MY_DEVELOPMENT_ARCHITECTURE.md`, 최신 AI 지침의 고정 경로는 `AI_DEVELOPMENT_INSTRUCTIONS.md`입니다. 버전이 올라가도 파일명을 바꾸지 않습니다.
- 이전 정식 버전은 `versions/vX.Y.md`에 원본으로 보존합니다. 또한 릴리스별 Git Tag(예: `v1.0`, `v1.1`)를 사용하면 당시 저장소 전체 상태를 다시 볼 수 있습니다.
- v1.1 개정 시: 현재 문서 보관 → 최신 정식 문서 수정 → AI 지침과 README 동기화 → 보류 목록 갱신 → 커밋 및 태그 추가 순으로 진행합니다.
- Git 커밋 메시지는 **`추가: `**(새 기능), **`고침: `**(치명적 오류 수정), **`변경: `**(그 외 수정)을 사용합니다. 본 문서 구조 변경은 `변경:`을 사용합니다.
- 미확정 확장 항목은 `FUTURE_UPDATE_BACKLOG.md`에 별도로 기록합니다.

> **참고:** 이전의 `MY_DEVELOPMENT_ARCHITECTURE_V1.0.md` 주소는 이번 구조 전환 후 더 이상 사용하지 않습니다. 최신 파일 URL을 기준으로 다른 프로젝트의 지침을 업데이트하세요. 이전 내용은 `versions/v1.0.md`에 보존되어 있습니다.
