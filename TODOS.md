# TODOS

## BLOCKER: API Terms of Service 확인
- **What:** OpenAI, Anthropic, Google API 이용약관에서 (1) 모델 출력물 상업적 재배포, (2) 모델 브랜드명을 경쟁적 맥락에서 사용, (3) "대결" 형식의 사용 허용 여부 확인
- **Why:** 약관 위반 시 API 키 정지 또는 법적 문제. 코드 작성 전에 확인 필수.
- **Pros:** 위험 사전 차단
- **Cons:** 확인에 시간 소요 (1-2시간)
- **Context:** 디자인 문서에서 BLOCKER로 지정. LMSYS Chatbot Arena 같은 선례가 있으므로 허용될 가능성 높지만, 브랜드명 사용과 "대결" 프레이밍이 다를 수 있음.
- **Depends on:** 없음 (빌드 전 최우선)

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
