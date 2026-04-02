# TODOS

## ~~BLOCKER: API Terms of Service 확인~~ ✅ RESOLVED (2026-04-02)
- **결과:** 세 provider 모두 허용 확인.
  - OpenAI: 출력물 소유권 사용자, 상업적 사용 가능. 경쟁 AI 학습만 금지.
  - Anthropic: 출력물 소유권 사용자, 상업적 사용 가능. API 래퍼(passthrough) 금지만 해당.
  - Google Gemini: 유료 티어 사용 시 출력물 소유권 사용자. 무료 티어는 학습에 사용됨.
  - 선례: LMSYS Chatbot Arena가 동일한 방식으로 OpenAI/Anthropic/Google과 파트너십 운영 중.
- **주의사항:** Google Gemini는 반드시 유료 티어(pay-as-you-go) 사용할 것.

## CI/CD 파이프라인 자동화
- **What:** GitHub Actions → fly deploy 자동 배포 파이프라인 구축
- **Why:** 수동 배포는 사이드 프로젝트에서 마찰을 만들어 업데이트 빈도를 낮춤.
- **Pros:** push-to-deploy로 배포 마찰 제거
- **Cons:** GitHub Actions + fly.io 설정 필요 (~30분)
- **Context:** Phase 1에서는 수동 배포(fly deploy)로 시작. 안정화 후 추가. Dockerfile은 이미 존재할 예정.
- **Depends on:** Phase 1 MVP 완성, fly.io 배포 성공

## LLM 프롬프트 품질 자동 eval 스위트
- **What:** "양비론 금지" 프롬프트 동작 검증, "공격적 페르소나"의 혐오 발언 경계 검증을 자동화
- **Why:** AUTO_PUBLISH=true로 전환하려면 프롬프트 품질이 안정적임을 증명해야 함. 수동 검토 20개가 끝나면 자동 eval이 그 역할을 이어받아야 함.
- **Pros:** 프롬프트 변경 시 회귀 감지, 안전한 자동 게시 전환
- **Cons:** eval 기준 정의 필요, LLM 호출 비용 발생
- **Context:** 초기 20개 수동 검토가 사실상 eval 역할. 자동화는 수동 검토에서 발견한 패턴을 기반으로 구축. toxicity 필터와 별개로, 프롬프트 자체의 품질 검증.
- **Depends on:** Phase 1 MVP 완성, 수동 검토 20개 완료
