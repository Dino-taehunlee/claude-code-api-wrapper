# Claude Code API Wrapper

Claude Code CLI를 HTTP API로 래핑하여 웹 애플리케이션에서 사용할 수 있게 해주는 Next.js 프로젝트입니다.

## 왜 이 프로젝트를 만들었나?

- **비용 절감**: Anthropic API 직접 호출 대신 Claude Code 구독으로 무제한 사용
- **도구 활용**: WebSearch, WebFetch 등 Claude Code의 내장 도구 사용 가능
- **에이전트 시스템**: 서브에이전트를 통한 전문화된 분석

## 사전 요구사항

### 1. Claude Code CLI 설치

```bash
npm install -g @anthropic-ai/claude-code
```

### 2. 최초 인증

```bash
# 터미널에서 실행 (브라우저 OAuth 인증)
claude
```

브라우저가 열리면 Anthropic 계정으로 로그인하세요. 인증이 완료되면 CLI 사용 가능합니다.

### 3. 인증 확인

```bash
# 테스트
claude --print "hello"
```

## 설치 및 실행

```bash
# 의존성 설치
npm install

# 개발 서버 실행
npm run dev

# http://localhost:3000 접속
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
awesome-demo-generate-agent/
├── .claude/
│   ├── settings.json              # 팀 공유 설정 (hooks, permissions)
│   ├── settings.local.json        # 개인 설정 (git 무시됨)
│   └── skills/
│       └── macro-analysis/        # 커스텀 스킬
│           └── SKILL.md
├── app/
│   ├── api/claude/
│   │   ├── route.ts               # 동기 API
│   │   └── stream/route.ts        # 스트리밍 API
│   ├── page.tsx                   # 테스트 UI
│   ├── layout.tsx
│   └── globals.css
├── .gitignore
├── package.json
└── README.md
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

- Claude Code CLI가 설치되고 인증된 로컬 환경에서만 작동합니다
- `--dangerously-skip-permissions` 플래그를 사용하므로 신뢰할 수 있는 환경에서만 실행하세요
- 타임아웃: 20분 (복잡한 분석 작업 고려)

## 라이선스

MIT
