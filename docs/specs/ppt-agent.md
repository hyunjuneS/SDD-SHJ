# PPT agent (DeepPresenter 기반)

> 예제 spec. clarify 답(가정): 입력은 주제+md/pdf(RAG는 v2) / 10장 내외, 사내 템플릿 1종 / 목차 승인 후 진행 / FastAPI 비동기 job / 성공 = 깨짐 없는 pptx + 승인 목차와 일치

## 1. 목적
주제와 md/pdf 자료를 받아 사내 템플릿 1종으로 10장 내외의 pptx를 생성한다.
사용자가 목차를 승인한 뒤에만 슬라이드를 생성해, 결과가 승인된 목차와 일치하도록 한다.

## 2. 범위 밖
- RAG 연동 (v2)
- 이미지 생성·검색
- 생성 후 슬라이드 개별 수정
- 템플릿 2종 이상

## 3. 수용 기준
- AC1: job 생성 시 job_id를 반환한다.
- AC2: 목차가 생성되면 status=awaiting_approval이 된다.
- AC3: 승인된 목차대로 슬라이드가 생성된다.
- AC4: pptx가 정상적으로 열리고 텍스트 넘침이 없다.
- AC5: job 1건의 전 과정이 Langfuse 단일 trace로 남는다.

## 4. 설계
### 구조
POST /jobs → outline_graph → LangGraph interrupt(승인 대기) → POST /jobs/{id}/approve → slide_graph → HTML → html2pptx(Playwright+PptxGenJS)

### 주요 결정
- 승인 대기는 LangGraph interrupt + checkpointer로 처리한다.
- pdf는 기존 파서를 재사용한다.
- overflow는 렌더 시 측정해 해당 슬라이드만 재생성한다 (최대 2회).
- job 상태는 기존 Redis Sentinel에 저장한다.

### 고려했지만 안 한 것
- 슬라이드 병렬 생성: v1은 순차 생성 (trace 가독성 우선).

## 5. 작업
- [ ] T1 Job API 골격 (검증: curl로 status=pending)
- [ ] T2 outline_graph + interrupt (검증: 샘플 md 2종으로 awaiting_approval)
- [ ] T3 승인 API + resume (검증: 목차 1개 삭제 후 반영 확인)
- [ ] T4 slide_graph HTML (검증: 10장 렌더 스크린샷)
- [ ] T5 overflow 재생성 (검증: 긴 텍스트 주입 시 2회 내 해소)
- [ ] T6 html2pptx + 다운로드 (검증: PowerPoint 육안 확인)
- [ ] T7 Langfuse trace (검증: job 1건 = trace 1개, span 구분)

### 셀프체크
- 커버리지: AC1→T1, AC2→T2, AC3→T3·T4, AC4→T5·T6, AC5→T7
- 원칙 위반: 없음 (기존 파서·Redis 재사용, 새 추상화 없음)
- 미해결: pdf 표 처리 확인 필요 (T2 전)

## 6. 결정 기록
