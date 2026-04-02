# gstack-harness

에이전트 우선(Agent-First) 개발을 위한 하네스 엔지니어링 템플릿.

[OpenAI의 하네스 엔지니어링](https://openai.com/index/harness-engineering/) 철학과 [gstack](https://github.com/anthropics/claude-code) 스킬 팩을 결합하여, **사람이 조정하고 에이전트가 수행하는** 개발 환경을 구축합니다.

## 핵심 철학

> 코드를 직접 작성하지 않는다. 환경을 설계하고, 의도를 명시하며, 에이전트가 안정적인 작업을 수행할 수 있도록 피드백 루프를 구축한다.

- **리포지터리 = 기록 시스템** — 에이전트가 읽을 수 없는 것은 존재하지 않는 것
- **AGENTS.md = 목차** — 백과사전이 아닌 맵으로 점진적 공개
- **제약이 곧 속도** — 레이어 경계를 기계적으로 강제하여 드리프트 방지
- **가비지 컬렉션** — 기술 부채를 매일 조금씩 정리

## 구조

```
gstack-harness/
├── AGENTS.md              # 에이전트 네비게이션 맵
├── CLAUDE.md              # 코딩 컨벤션 & 파이프라인 실행 가이드
├── ARCHITECTURE.md        # 레이어 아키텍처 & 피드백 루프
├── packages/              # 제품 패키지 (모노레포)
└── docs/
    ├── GSTACK_SKILLS.md   # gstack 36개 스킬 레퍼런스
    ├── QUALITY_SCORE.md   # 도메인별 품질 추적
    ├── SECURITY.md        # 보안 원칙
    ├── design-docs/       # 설계 문서 & 핵심 신념
    ├── exec-plans/        # 실행 계획 (active/completed)
    ├── references/        # 외부 레퍼런스
    └── generated/         # 자동 생성 문서
```

## gstack 파이프라인 (14단계)

기능 요청 시 전체 흐름:

```
/office-hours → /plan-ceo-review → /plan-eng-review → /design-consultation
→ /autoplan → BUILD → /design-review → /review → /qa → /cso
→ /ship → /document-release → /retro → /checkpoint
```

| 상황 | 파이프라인 |
|------|-----------|
| 기능 개발 | 전체 14단계 |
| 버그 수정 | 1~4단계 스킵 |
| 빠른 모드 | `BUILD → /review → /qa → /ship` |

## 레이어 아키텍처

```
Types → Config → Models → Repositories → Services → Routes/UI
```

- 위 → 아래 방향만 허용, 역방향 의존성 금지
- 교차 관심사(Auth, Telemetry)는 Providers를 통해 주입
- 린터와 구조적 테스트로 기계적 강제

## 사용법

1. 이 템플릿을 클론하거나 포크
2. 제품을 정하고 `/office-hours`로 시작
3. 에이전트가 코드를 작성하고, 사람은 검토와 의사결정에 집중

## 참고

- [OpenAI: Harness Engineering](https://openai.com/index/harness-engineering/)
- [OpenAI: Codex 하네스 활용하기](https://openai.com/index/using-the-codex-harness/)

## License

MIT
