# Personal Development Architecture

개인 개발 아키텍처 및 AI 협업 코딩 규칙을 관리하는 저장소입니다.

이 저장소는 특정 서비스의 소스 코드가 아닌, 여러 프로젝트에 공통으로 적용할 **개인 개발 표준**을 보관합니다.

## 문서

| 파일 | 설명 | 활용 방법 |
| --- | --- | --- |
| [MY_DEVELOPMENT_ARCHITECTURE_V1.0.md](./MY_DEVELOPMENT_ARCHITECTURE_V1.0.md) | v1.0 정식 아키텍처 표준. 폴더·컴포넌트·상태·Tailwind·SEO·API·AI 수정 규칙을 포함 | 개발 설계와 코드 수정의 최우선 기준 |
| [AI_DEVELOPMENT_INSTRUCTIONS.md](./AI_DEVELOPMENT_INSTRUCTIONS.md) | ChatGPT에 전달할 간결한 실행 지침 | ChatGPT 프로젝트 지침에 넣어 활용 |
| [FUTURE_UPDATE_BACKLOG.md](./FUTURE_UPDATE_BACKLOG.md) | 후속 버전에서 논의할 테스트·로깅·배포·보안 심화·성능 항목 | 규칙 업데이트 시 참조 |

## 다른 프로젝트에서 사용하기

1. 새 프로젝트의 ChatGPT 프로젝트 지침에 `AI_DEVELOPMENT_INSTRUCTIONS.md`의 내용을 반영합니다.
2. 정식 아키텍처 문서 `MY_DEVELOPMENT_ARCHITECTURE_V1.0.md`를 첨부하거나 접근 가능한 자료로 제공합니다.
3. 실제 소스 코드도 함께 제공하고, 이 문서의 확정 규칙에 따라 작성·수정하도록 요청합니다.
4. 요약 지침과 정식 문서가 충돌하면 **정식 아키텍처 문서**를 우선합니다.

## 적용 범위

- 범용 개인 원칙: 책임 분리, 컴포넌트 최소 단위 설계, 의존성, 네이밍, AI 수정·보고 계약
- 주요 기술 환경: TypeScript, React, Next.js App Router, Tailwind CSS
- 다른 프레임워크를 사용할 때는 프레임워크 전용 파일명보다 설계 원칙을 우선하여 적용합니다.

## 버전

- 현재 정식 버전: **v1.0**
- 미확정 확장 항목은 `FUTURE_UPDATE_BACKLOG.md`에 별도로 기록합니다.
