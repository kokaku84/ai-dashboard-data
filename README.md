# ai-dashboard-data

AI 트렌드 대시보드의 **GitHub 스타 트렌드**와 **의료영상 레이더** 데이터를 매일 모으는 저장소입니다.

## 동작
1. GitHub Actions가 매일 한국 시간 06:17에 `scripts/collect.py`를 실행합니다.
2. `config/tags.json`의 토픽으로 GitHub 검색 API를 조회해 정확한 스타 수를 `data/snapshots/날짜.json`에 저장합니다.
3. 일간·주간·30일 증가, 증가율, 가속도, 활동성, 확인 필요 신호를 계산합니다.
4. 화면에 나갈 저장소 중 설명이 없는 것만 Claude API(Max/Ultra 월 크레딧)로 한국어 설명을 만듭니다. 설명은 `data/meta.json`에 캐시합니다.
5. Hugging Face 의료(medical) 모델 트렌드를 더해 `data/results.json`을 만듭니다.
6. 대시보드의 매일 예약 작업(07:00)이 `data/results.json`을 읽어 공개 페이지에 반영합니다.

## 설정
- Claude API 인증은 **Workload Identity Federation**을 씁니다. 저장소에 API 키(시크릿)를 두지 않습니다.
  - GitHub Actions가 발급한 짧은 OIDC 토큰을 Claude Console의 규칙 `dashboard-stars-daily`(이 저장소, main 브랜치, 계정 ID 20944647)로 검증해 몇 분짜리 Anthropic 토큰으로 바꿉니다.
  - 워크플로의 `ANTHROPIC_*` 값은 식별자일 뿐 비밀값이 아닙니다.
  - 사용량은 Console 워크스페이스 `dashboard-bot`(월 지출 한도 설정)으로 잡히고, Max/Ultra 월 API 크레딧에서 먼저 차감됩니다.
- 인증이 실패해도 수집은 계속되고, 설명만 영문 GitHub 설명으로 표시됩니다.
- 선택 변수: `CLAUDE_MODEL`(기본 `claude-sonnet-5-5`), `MAX_DESCRIBE`(하루 최대 설명 수, 기본 60)
- `anthropic-wif-test.yml`은 Console 연결 테스트용입니다. 확인이 끝나면 지워도 됩니다.

## 수동 실행
Actions 탭 → daily-stars → Run workflow

## 로컬 테스트
```
python scripts/collect.py --no-llm --no-hf --data-dir /tmp/data
```

## 지표 정의
| 지표 | 정의 |
|---|---|
| 일간 증가 | 오늘 스타 − 어제 스타 |
| 주간 증가 | 오늘 스타 − 7일 전 스타 |
| 증가율 | 주간 증가 ÷ 7일 전 스타 (7일 전 100개 이상만) |
| 가속 ▲ / 둔화 ▼ | 이번 주 증가가 지난주보다 25% 이상 많음(▲) / 적음(▼), 차이 10개 이상 |
| 확인 필요 | 주간 300개 이상 늘었는데 포크가 스타의 1% 미만이고 열린 이슈가 0개 |
| 업데이트 없음 | 마지막 푸시가 90일 넘게 지남 |
| 의료영상 레이더 | `medical` 토픽 저장소를 30일 증가 순으로 (기록이 30일 미만이면 7일·1일 증가) |
