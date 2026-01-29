# Claude Code API Wrapper

Claude Code CLI를 HTTP API로 래핑하여 웹 애플리케이션에서 사용할 수 있게 해주는 Next.js 프로젝트입니다.

## 왜 이 프로젝트를 만들었나?

- **비용 절감**: Anthropic API 직접 호출 대신 Claude Code 구독으로 무제한 사용
- **도구 활용**: WebSearch, WebFetch 등 Claude Code의 내장 도구 사용 가능
- **에이전트 시스템**: 서브에이전트를 통한 전문화된 분석

## 사전 요구사항

### 0. 기본 환경 설정 (처음 시작하는 경우)

**Node.js 설치 확인**
```bash
node --version
npm --version
```

**Node.js가 없다면 설치:**

<details>
<summary>macOS</summary>

```bash
# Homebrew 설치 (없을 경우)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Node.js 설치
brew install node
```
</details>

<details>
<summary>Linux (Ubuntu/Debian)</summary>

```bash
# Node.js 20.x 설치
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```
</details>

<details>
<summary>Windows</summary>

[Node.js 공식 사이트](https://nodejs.org/)에서 LTS 버전 다운로드 및 설치
</details>

**Git 설치 확인 및 설치**
```bash
# 확인
git --version

# macOS: 없다면 자동 설치 프롬프트
# Linux: sudo apt-get install git
# Windows: https://git-scm.com/download/win
```

### 1. Claude Code CLI 설치

```bash
npm install -g @anthropic-ai/claude-code
```

**설치 확인:**
```bash
claude --version
# 출력 예시: 2.0.21
```

**문제 발생 시:**
- 권한 오류: `sudo npm install -g @anthropic-ai/claude-code`
- 경로 오류: `~/.npm-global/bin`을 PATH에 추가

### 2. 최초 인증

```bash
# 터미널에서 실행
claude
```

**인증 과정:**
1. 터미널에서 `claude` 명령어 실행
2. 브라우저가 자동으로 열림
3. Anthropic 계정으로 로그인 (없으면 가입)
4. "Authorize" 버튼 클릭
5. 터미널로 돌아가면 인증 완료

**브라우저가 안 열린다면:**
```bash
# 수동으로 URL 복사해서 브라우저에 붙여넣기
claude --auth
```

### 3. 인증 확인

```bash
# 간단한 테스트
claude --print "hello"
```

**성공 시 출력:**
```
Hello! How can I help you today?
```

**실패 시:**
- `~/.claude` 폴더 확인: `ls -la ~/.claude`
- 재인증: `claude` 명령어 다시 실행
- 로그 확인: `claude --verbose --print "test"`

## 설치 및 실행

```bash
# 의존성 설치
npm install

# 개발 서버 실행
npm run dev

# 웹 브라우저에서 http://localhost:3000 접속
```

## 웹 UI로 테스트하기

브라우저에서 `http://localhost:3000`을 열면 바로 사용할 수 있는 테스트 UI가 제공됩니다.

**주요 기능:**
- 🎯 **예시 프롬프트**: 거시경제, 금리 전망, 삼성전자 분석 등 원클릭 테스트
- 💬 **실시간 스트리밍**: 응답이 타이핑되는 것처럼 실시간으로 표시
- 🛠️ **도구 사용 내역**: WebSearch, WebFetch 등 사용된 도구 확인
- 📊 **메트릭 표시**: 소요시간, 비용, 턴수 등 실시간 모니터링
- 🔍 **Raw 메시지**: 디버깅을 위한 전체 JSON 메시지 확인

![웹 UI 스크린샷](https://via.placeholder.com/800x400?text=Claude+Code+API+Web+UI)

## HTTP API로 사용하기

서버가 실행되면 어떤 언어에서든 HTTP API로 사용할 수 있습니다.

## 다양한 언어에서 사용하기

### cURL

```bash
# 스트리밍 API
curl -X POST http://localhost:3000/api/claude/stream \
  -H "Content-Type: application/json" \
  -d '{"prompt": "현재 미국 경제 상황 분석해줘"}' \
  --no-buffer

# 동기 API
curl -X POST http://localhost:3000/api/claude \
  -H "Content-Type: application/json" \
  -d '{"prompt": "삼성전자 주가 분석해줘"}'
```

### Python

```python
import requests
import json

# 스트리밍 응답
response = requests.post(
    "http://localhost:3000/api/claude/stream",
    json={"prompt": "거시경제 분석해줘"},
    stream=True
)

for line in response.iter_lines():
    if line:
        data = json.loads(line.decode('utf-8'))
        if data['type'] == 'assistant':
            print(data['message']['content'][0].get('text', ''))
        elif data['type'] == 'result':
            print(f"\n\n비용: ${data['total_cost_usd']:.4f}")

# 동기 응답
response = requests.post(
    "http://localhost:3000/api/claude",
    json={"prompt": "삼성전자 분석해줘"}
)
result = response.json()
print(result['result'])
```

### JavaScript/Node.js

```javascript
// 스트리밍
const response = await fetch('http://localhost:3000/api/claude/stream', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ prompt: '미국 경제 분석해줘' })
});

const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const text = decoder.decode(value);
  const lines = text.split('\n').filter(line => line.trim());

  for (const line of lines) {
    const msg = JSON.parse(line);
    if (msg.type === 'assistant') {
      console.log(msg.message.content[0]?.text);
    }
  }
}

// 동기
const syncResponse = await fetch('http://localhost:3000/api/claude', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ prompt: '삼성전자 분석해줘' })
});

const data = await syncResponse.json();
console.log(data.result);
```

### Go

```go
package main

import (
    "bufio"
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

func main() {
    // 스트리밍 요청
    body := map[string]string{"prompt": "거시경제 분석해줘"}
    jsonData, _ := json.Marshal(body)

    resp, _ := http.Post(
        "http://localhost:3000/api/claude/stream",
        "application/json",
        bytes.NewBuffer(jsonData),
    )
    defer resp.Body.Close()

    scanner := bufio.NewScanner(resp.Body)
    for scanner.Scan() {
        var msg map[string]interface{}
        json.Unmarshal(scanner.Bytes(), &msg)

        if msg["type"] == "assistant" {
            content := msg["message"].(map[string]interface{})["content"]
            fmt.Println(content)
        }
    }
}
```

### Ruby

```ruby
require 'net/http'
require 'json'
require 'uri'

# 동기 요청
uri = URI('http://localhost:3000/api/claude')
req = Net::HTTP::Post.new(uri, 'Content-Type' => 'application/json')
req.body = { prompt: '삼성전자 분석해줘' }.to_json

res = Net::HTTP.start(uri.hostname, uri.port) { |http| http.request(req) }
result = JSON.parse(res.body)
puts result['result']
```

## API 엔드포인트

### POST /api/claude/stream (스트리밍)

실시간 스트리밍 응답을 반환합니다.

```javascript
const response = await fetch('/api/claude/stream', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    prompt: '현재 미국 경제 상황 분석해줘'
  })
});

const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const text = decoder.decode(value);
  const lines = text.split('\n').filter(line => line.trim());

  for (const line of lines) {
    const msg = JSON.parse(line);
    // msg.type: 'system' | 'assistant' | 'result' | 'stream_event'
    console.log(msg);
  }
}
```

### POST /api/claude (동기)

전체 응답을 대기 후 반환합니다.

```javascript
const response = await fetch('/api/claude', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    prompt: '삼성전자 주가 분석해줘'
  })
});

const data = await response.json();
console.log(data.result);
```

### GET /api/claude

API 문서를 반환합니다.

## 요청 옵션

| 옵션 | 타입 | 설명 |
|------|------|------|
| `prompt` | string | **필수**. 분석 요청 내용 |
| `agents` | object | 커스텀 서브에이전트 추가 |
| `useDefaultAgents` | boolean | 기본 에이전트 사용 여부 (기본값: true) |
| `allowedTools` | string[] | 허용할 도구 목록 |
| `disallowedTools` | string[] | 차단할 도구 목록 |

## 프로젝트 구조

```
.
├── .claude/                        # Claude Code 설정
│   ├── settings.json               # 팀 공유 설정
│   ├── settings.local.json         # 개인 설정 (git 무시)
│   └── skills/
│       └── macro-analysis/         # 커스텀 스킬
│           └── SKILL.md
├── app/                            # Next.js App Router
│   ├── api/
│   │   └── claude/
│   │       ├── route.ts            # 동기 API (/api/claude)
│   │       └── stream/
│   │           └── route.ts        # 스트리밍 API (/api/claude/stream)
│   ├── layout.tsx                  # Root layout
│   ├── page.tsx                    # 웹 UI (테스트 인터페이스)
│   └── globals.css                 # Tailwind CSS
├── .gitignore
├── next.config.ts                  # Next.js 설정
├── package.json
├── package-lock.json
├── postcss.config.mjs              # PostCSS 설정
├── README.md                       # 프로젝트 문서
├── tailwind.config.ts              # Tailwind CSS 설정
├── TEST.md                         # 테스트 가이드
└── tsconfig.json                   # TypeScript 설정
```

---

## Claude Code 설정 가이드

### 설정 파일 구조

| 파일 | 위치 | 용도 | Git |
|------|------|------|-----|
| `settings.json` | `.claude/` | 팀 공유 설정 | ✅ 커밋 |
| `settings.local.json` | `.claude/` | 개인 로컬 설정 | ❌ 무시 |

### settings.json 예시

```json
{
  "permissions": {
    "allow": [
      "WebFetch(domain:fred.stlouisfed.org)",
      "Bash(npm:*)",
      "WebSearch"
    ],
    "deny": [
      "Bash(rm -rf:*)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "WebSearch",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[WebSearch] $(date +%H:%M:%S): $CLAUDE_TOOL_INPUT\" >> /tmp/claude.log"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"파일 수정됨: $CLAUDE_TOOL_INPUT\" >> /tmp/claude.log"
          }
        ]
      }
    ]
  }
}
```

### Permissions (권한 설정)

Claude Code가 도구를 사용할 때 자동 허용/거부 규칙:

```json
{
  "permissions": {
    "allow": [
      "WebFetch(domain:example.com)",    // 특정 도메인만 허용
      "Bash(npm:*)",                     // npm 명령어 허용
      "Bash(git:*)",                     // git 명령어 허용
      "Read",                            // 파일 읽기 전체 허용
      "WebSearch"                        // 웹 검색 허용
    ],
    "deny": [
      "Bash(rm -rf:*)",                  // 위험한 명령어 차단
      "Write(*.env)"                     // .env 파일 수정 차단
    ]
  }
}
```

### Hooks (훅 설정)

도구 실행 전/후에 자동으로 명령어 실행:

| 이벤트 | 설명 |
|--------|------|
| `PreToolUse` | 도구 실행 **전** |
| `PostToolUse` | 도구 실행 **후** |

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "WebSearch",          // WebSearch 도구에만 적용
        "hooks": [
          {
            "type": "command",
            "command": "echo 검색 시작 >> /tmp/log.txt"
          }
        ]
      }
    ]
  }
}
```

**환경 변수:**
- `$CLAUDE_TOOL_INPUT` - 도구 입력값
- `$CLAUDE_TOOL_NAME` - 도구 이름
- `$CLAUDE_SESSION_ID` - 세션 ID

---

## Skills (스킬)

스킬은 특정 키워드나 상황에서 Claude가 참조하는 가이드라인입니다.

### 스킬 생성

`.claude/skills/<skill-name>/SKILL.md` 파일 생성:

```markdown
# 거시경제 분석 스킬

## 트리거
- "거시경제", "금리", "인플레이션", "GDP" 키워드

## 분석 항목
1. **성장**: GDP, PMI
2. **물가**: CPI, PCE
3. **고용**: 실업률, NFP
4. **금리**: 기준금리, 10년물 금리

## 출력 형식
| 지표 | 수치 | 평가 |
|------|------|------|

면책: 정보 제공 목적이며 투자 조언 아님
```

### 스킬 구조

```
.claude/skills/
├── macro-analysis/
│   └── SKILL.md           # 거시경제 분석
├── stock-analysis/
│   └── SKILL.md           # 주식 분석
└── code-review/
    └── SKILL.md           # 코드 리뷰
```

---

## Subagents (서브에이전트)

API에서 커스텀 서브에이전트 추가:

```javascript
fetch('/api/claude/stream', {
  method: 'POST',
  body: JSON.stringify({
    prompt: '최신 AI 뉴스 요약해줘',
    agents: {
      'news-researcher': {
        description: '뉴스 리서치 전문가',
        prompt: '최신 뉴스를 검색하고 핵심만 요약하세요.',
        tools: ['WebSearch', 'WebFetch'],
        model: 'haiku'  // sonnet | opus | haiku
      }
    }
  })
});
```

### 기본 제공 에이전트

**API 내장:**
- `financial-analyst` - 금융/투자 분석 전문가

**Claude Code 내장:**
- `Explore` - 코드베이스 탐색 (읽기 전용)
- `Plan` - 계획 모드 리서치
- `general-purpose` - 범용 멀티스텝 작업

---

## MCP 서버 (선택)

외부 데이터 소스 연동 시 `route.ts`에서 설정:

```typescript
const MCP_SERVERS: MCPServer[] = [
  // HTTP 방식
  {
    name: 'alphavantage',
    transport: 'http',
    url: 'https://mcp.alphavantage.co/mcp?apikey=YOUR_KEY',
  },
  // stdio 방식 (npx)
  {
    name: 'github',
    transport: 'stdio',
    command: 'npx',
    args: ['-y', '@modelcontextprotocol/server-github'],
  },
];
```

**참고:** WebSearch, WebFetch는 Claude Code 내장 도구이므로 MCP 없이 사용 가능.

---

## 내장 도구 목록

| 카테고리 | 도구 | 설명 |
|----------|------|------|
| 파일 | `Read`, `Edit`, `Write` | 파일 읽기/수정/생성 |
| 검색 | `Glob`, `Grep`, `LS` | 패턴 매칭, 내용 검색 |
| 실행 | `Bash` | 셸 명령어 실행 |
| 웹 | `WebSearch`, `WebFetch` | 웹 검색, 페이지 가져오기 |
| 작업 | `TodoRead`, `TodoWrite` | 작업 관리 |
| 노트북 | `NotebookRead`, `NotebookEdit` | Jupyter 노트북 |

---

## 스트리밍 메시지 타입

```typescript
interface StreamMessage {
  type: 'system' | 'assistant' | 'user' | 'result' | 'stream_event';
  subtype?: string;  // 'init' 등
  message?: { content: Array<{ type: string; text?: string }> };
  event?: { type: string; delta?: { text: string } };
  result?: string;
  duration_ms?: number;
  total_cost_usd?: number;
}
```

---

## 주의사항

- Claude Code CLI 설치 및 인증이 필수입니다 (`claude` 명령어로 로그인)
- Claude Code 구독이 있어야 사용 가능합니다 (API 키 불필요)
- `--dangerously-skip-permissions` 플래그를 사용하므로 신뢰할 수 있는 환경에서만 실행하세요
- 타임아웃: 20분 (복잡한 분석 작업 고려)
- **비용 절감**: Anthropic API 대신 Claude Code 구독을 사용하므로 별도 API 비용 없음

## 라이선스

MIT
