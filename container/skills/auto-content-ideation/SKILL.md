# auto-content-ideation

AI서대표 유튜브 채널 콘텐츠 아이디어 자동 생성 스킬.

매일 아침 TOK-PMO 봇이 공유한 GeekNews + AI 업데이트를 읽고,
AI서대표 채널에 활용 가능한 콘텐츠 아이디어를 뽑아서 공유한다.

## 실행 방법

이 스킬이 호출되면 아래 단계를 실행:

### 1. 오늘 날짜 기준 메시지 수집

`C0AJT0TCFK3` 채널에서 오늘 날짜의 메시지만 수집:
- `:newspaper: _GeekNews 업데이트_` 로 시작하는 메시지
- `:bell: _AI 업데이트_` 로 시작하는 메시지

각 메시지의 **쓰레드(replies)**도 함께 읽어서 실제 아이템 내용 수집.

**중요**: 이 채널에서는 절대 메시지를 보내지 말 것. Read Only.

Slack API 사용:
```bash
SLACK_BOT_TOKEN=$(grep SLACK_BOT_TOKEN /workspace/project/data/env/env | cut -d= -f2)

# 채널 히스토리 가져오기
curl -s "https://slack.com/api/conversations.history?channel=C0AJT0TCFK3&limit=50" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN"

# 쓰레드 내용 가져오기
curl -s "https://slack.com/api/conversations.replies?channel=C0AJT0TCFK3&ts=THREAD_TS&limit=50" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN"
```

### 2. 콘텐츠 아이디어 선별 기준

AI서대표 채널 특성:
- 대상: AI에 익숙하지 않은 일반인 ~ 초중급 AI 활용자
- 채널 방향: AI 활용법, AI 네이티브 업무, AI 도구 리뷰, AI 트렌드
- 인물 등장 영상 선호 (인스타 릴스 기준)
- 핫한 키워드에 조회수 집중 (AI 에이전트 등)

**선별 기준**:
- ✅ 일반인도 이해 가능한 AI 활용 사례
- ✅ 화제성 있는 AI 도구/서비스 (ChatGPT, Claude 등)
- ✅ "이걸로 돈 벌 수 있다", "생산성 N배" 류의 실용적 내용
- ✅ 국내 트렌드와 연결 가능한 것
- ❌ 너무 기술적인 개발자용 내용 (WASM, Rust 등)
- ❌ 이미 많이 다뤄진 진부한 내용

### 3. 아이디어 포맷 작성

각 아이디어는 다음 형식으로:
```
*[번호]. [콘텐츠 제목 아이디어]*
> [원문 요약 1줄]
• 콘텐츠 방향: [어떻게 만들면 좋을지]
• 예상 타겟: [누구에게 도움될지]
• 참고 링크: <URL|링크텍스트>
```

### 4. 메시지 전송

**메인 메시지** → `C0ANDGBCLTT` (ai-서대표-nanoclaw 채널)에 전송:
```
:camera_with_flash: _AI서대표 컨텐츠 아이디어_ | 2026-03-XX — N개
```

**쓰레드** → 메인 메시지의 thread_ts로 상세 아이디어 전송:
```
*오늘의 AI서대표 콘텐츠 아이디어 [N개]*

[아이디어 목록]

_출처: #tok-pmo-ai-updates 채널 기반_
```

Slack API로 메시지 전송:
```bash
SLACK_BOT_TOKEN=$(grep SLACK_BOT_TOKEN /workspace/project/data/env/env | cut -d= -f2)

# 메인 메시지 전송
curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel": "C0ANDGBCLTT", "text": ":camera_with_flash: _AI서대표 컨텐츠 아이디어_ | 2026-03-XX — N개"}'

# 쓰레드 답글 전송 (위 응답의 ts 사용)
curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel": "C0ANDGBCLTT", "thread_ts": "MAIN_TS", "text": "상세내용..."}'
```

### 5. 아이디어가 없을 경우

오늘 날짜 업데이트가 아직 없거나, 활용할 만한 내용이 없으면:
- 아무것도 전송하지 않음 (조용히 종료)

## 주의사항

- `C0AJT0TCFK3` 채널에는 절대 메시지 전송 금지 (Read Only)
- 아이디어는 최소 2개, 최대 5개 선별
- 너무 기술적인 내용은 제외
- 항상 오늘 날짜(KST) 기준으로 필터링
