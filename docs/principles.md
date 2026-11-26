# 개발 원칙

모든 spec, plan, build는 이 원칙을 따른다. 위반이 필요하면 사람 승인을 받고 spec의 결정 기록에 남긴다.

- 폐쇄망 전제: 외부 API/CDN 금지, 의존성은 Nexus 반입 가능한 것만
- 기존 패턴 재사용 우선 (새 그래프는 design_graph.py 구조를 따름)
- 모든 LLM 호출은 Langfuse trace에 남김
- 폰트는 맑은 고딕 고정
- 새 추상화 도입 전 사람 승인
