# AGENTS.md — 에이전트 네비게이션 맵

> 이 파일은 목차(맵)이다. 백과사전이 아니다.
> 에이전트가 읽을 수 없는 것은 존재하지 않는 것이다.
> 모든 지식은 리포지터리에 버전관리되는 아티팩트로 존재해야 한다.

## 핵심 원칙

1. **사람이 조정하고, 에이전트가 수행** — 코드 한 줄도 수동으로 작성하지 않음
2. **레이어 경계는 강제, 내부는 자율** — 불변조건만 잠그고 구현은 에이전트에게 위임
3. **빠른 병합, 후속 PR로 개선** — 수정 비용이 대기 비용보다 저렴
4. **가비지 컬렉션** — 기술 부채는 매일 조금씩 갚음

## 리포지터리 구조

```
.
├── AGENTS.md              # ← 지금 여기 (에이전트 맵)
├── CLAUDE.md              # 코딩 컨벤션 & 실행 가이드
├── ARCHITECTURE.md        # 시스템 아키텍처 다이어그램
├── packages/              # 모노레포 패키지들
├── docs/
│   ├── design-docs/       # 설계 문서 (core-beliefs, index)
│   ├── exec-plans/        # 실행 계획 (active/, completed/)
│   ├── references/        # 외부 레퍼런스 (llms.txt 등)
│   ├── generated/         # 자동 생성 문서 (DB 스키마 등)
│   ├── GSTACK_SKILLS.md   # gstack 스킬 문서
│   ├── QUALITY_SCORE.md   # 도메인별 품질 점수
│   └── SECURITY.md        # 보안 원칙
└── infra/                 # 인프라 스크립트
```

## 작업 시 참조 순서

1. 이 파일(AGENTS.md)로 위치 파악
2. `CLAUDE.md`로 컨벤션 확인
3. `ARCHITECTURE.md`로 경계 확인
4. `docs/design-docs/`에서 설계 의도 확인
5. `docs/exec-plans/active/`에서 진행 중인 계획 확인

## gstack 파이프라인 (14단계)

기능 요청 시 전체 흐름:
`/office-hours` → `/plan-ceo-review` → `/plan-eng-review` → `/design-consultation` → `/autoplan` → BUILD → `/design-review` → `/review` → `/qa` → `/cso` → `/ship` → `/document-release` → `/retro` → `/checkpoint`

빠른 모드: `BUILD` → `/review` → `/qa` → `/ship`
버그 수정: 1~4단계 스킵
