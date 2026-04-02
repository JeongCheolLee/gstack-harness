# Phase 1: LLM Arena MVP — Execution Plan

Status: READY TO BUILD
Created: 2026-04-02
Source: /office-hours design doc + /plan-eng-review

## 목표

매일 1회 AI 모델 대결을 자동 생성하고, 웹사이트에서 투표 + ELO 리더보드를 제공하는 MVP.

## 기술 스택

| 항목 | 선택 | 이유 |
|------|------|------|
| 언어 | Python 3.11+ | 백엔드 중심, LLM 라이브러리 생태계 |
| 웹 프레임워크 | FastAPI | Layer 1, async 지원, 충분히 boring |
| DB | SQLite + WAL mode | 단일 서버, 일 1회 쓰기, 단순 |
| DB 백업 | Litestream → S3 | fly.io volume 단일 장애점 대비 |
| LLM 호출 | litellm | 3개 provider를 단일 completion() 호출로 |
| ELO 레이팅 | elo (PyPI) | K=32, 초기 1500, 검증된 패키지 |
| SNS | atproto SDK (Bluesky) | 무료 API, 개발자 커뮤니티 활발 |
| Toxicity | OpenAI Moderation API | Layer 1, 무료 |
| 배포 | fly.io (free tier) | cron + web 단일 서버, Dockerfile |
| 테스트 | pytest | Python 표준 |

## 모듈 구조 (플랫)

```
packages/arena/
├── config.py       # 환경변수, 설정 (API keys, AUTO_PUBLISH, DB path)
├── models.py       # SQLAlchemy 모델 (Debate, Vote, Model, EloHistory)
├── debate.py       # 토론 엔진 (litellm 호출, 4라운드 로직, 주제/모델 선택)
├── elo.py          # ELO 업데이트 (elo 패키지 래핑, 승/패/무 판정)
├── social.py       # Bluesky 포스팅 (atproto SDK, 스레드 포맷팅)
├── api.py          # FastAPI 라우트 (투표, 리더보드, 토론 조회)
├── safety.py       # Toxicity 필터 (OpenAI Moderation, fail-closed)
├── main.py         # 앱 진입점 + APScheduler cron
├── Dockerfile
├── requirements.txt
└── tests/
    ├── test_debate.py
    ├── test_elo.py
    ├── test_safety.py
    ├── test_social.py
    ├── test_api.py
    └── test_e2e.py
```

## 데이터 모델

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    Model     │     │    Debate    │     │     Vote     │
├──────────────┤     ├──────────────┤     ├──────────────┤
│ id           │◄────│ model_a_id   │     │ id           │
│ name         │◄────│ model_b_id   │     │ debate_id    │──►Debate
│ provider     │     │ topic        │     │ choice       │  (a or b)
│ elo_rating   │     │ rounds (JSON)│     │ ip_hash      │
│ wins         │     │ status       │     │ fingerprint  │
│ losses       │     │ created_at   │     │ created_at   │
│ draws        │     │ vote_deadline│     └──────────────┘
│ match_count  │     │ result       │
└──────────────┘     │ published    │
                     └──────────────┘
```

## 핵심 플로우

```
[Daily Cron - APScheduler]
    │
    ├── 1. select_topic() ─── 주제 풀에서 랜덤 선택 (중복 방지)
    │
    ├── 2. select_models() ── 2개 모델 랜덤 페어링 (같은 모델 방지)
    │
    ├── 3. run_debate() ───── litellm.completion() x 8 (4라운드, 모델당 4회)
    │   │                     시스템 프롬프트: "한쪽 입장만, 양비론 금지, 100 tokens"
    │   │
    │   ├── [성공] → 4. check_toxicity() ── OpenAI Moderation API
    │   │               │
    │   │               ├── [통과] → 5. save_to_db(status=active)
    │   │               │               │
    │   │               │               ├── [AUTO_PUBLISH=true] → 6. post_to_bluesky()
    │   │               │               └── [AUTO_PUBLISH=false] → 로그 출력, 수동 검토 대기
    │   │               │
    │   │               └── [차단] → 로그 경고, 재시도 또는 스킵
    │   │
    │   └── [실패] → 로그 에러 (API timeout/429), 내일 재시도
    │
    └── 7. update_elo() ───── 투표 마감된 이전 토론의 ELO 업데이트

[User Flow - FastAPI]
    │
    ├── GET /debate/{id} ──── 토론 상세 보기 + 투표 UI
    ├── POST /vote/{id} ───── 투표 (IP+fingerprint 중복 체크)
    ├── GET /leaderboard ──── ELO 랭킹 (5경기 이상만)
    └── GET /debates ──────── 토론 목록 (페이지네이션)
```

## 투표 규칙

- 투표 기간: 토론 생성 후 24시간
- 중복 방지: IP hash + browser fingerprint 조합
- 판정: >55% → 승리, 45-55% → 무승부, <10표 → 미반영
- ELO: K-factor=32, 초기 1500, 5경기 미만 모델은 리더보드 비표시

## 콘텐츠 안전

- 시스템 프롬프트: 강한 입장 + 혐오/차별 금지 경계
- Toxicity 필터: OpenAI Moderation API (fail-closed: 필터 실패 = 게시 안 함)
- 금지 주제: 인종, 종교 비하, 아동, 개인 비방
- 초기 20개: AUTO_PUBLISH=false, 수동 검토 후 true 전환
- 프롬프트 강도: 필터 발동률 20% 이하 목표 (수동 검토에서 튜닝)

## 환경변수

```
# LLM API Keys
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
GOOGLE_API_KEY=

# Bluesky
BLUESKY_HANDLE=
BLUESKY_APP_PASSWORD=

# App Config
DATABASE_URL=sqlite:///data/arena.db
AUTO_PUBLISH=false
DAILY_CRON_HOUR=9
VOTE_DURATION_HOURS=24

# Backup (Litestream)
LITESTREAM_REPLICA_URL=s3://bucket/arena.db
```

## 빌드 순서 (병렬 가능)

```
Phase A: 기반
  1. config.py + models.py + DB 초기화
  2. pytest 설정

Phase B: 핵심 로직 (A 완료 후 병렬)
  Lane 1: debate.py + safety.py + tests
  Lane 2: api.py + elo.py + tests
  Lane 3: social.py + tests

Phase C: 통합 (B 전체 완료 후)
  1. main.py (cron + FastAPI 통합)
  2. E2E 테스트
  3. Dockerfile + fly.toml
  4. Litestream 설정
```

## 테스트 목표

- 35개 단위/E2E 테스트 (38개 코드 경로 중 35개 자동화)
- LLM 프롬프트 eval: 수동 검토 20개 (Phase 2에서 자동화)
- Critical: toxicity 필터 fail-closed 테스트 필수

## 성공 기준

- 2주 운영: 웹사이트 투표 평균 50+/매치, SNS 포스트당 10+ 좋아요/리포스트
- 기술: 하루 1회 크론 안정 실행, 다운타임 없음

## NOT in scope (Phase 1)

1. 영상 생성 파이프라인 (Phase 2)
2. 자동화된 LLM eval 스위트 (수동 검토로 대체)
3. CI/CD 자동 배포 (수동 fly deploy)
4. OAuth 로그인 기반 투표
5. 관리자 UI
6. 다국어 지원 (영어 우선)
7. Twitter/X 연동 (Bluesky만)
