# MCP 서버 설정 가이드 🚀

Claude Desktop에 MCP(Model Context Protocol) 서버들을 연결하여 다양한 외부 서비스와 상호작용할 수 있도록 하는 완전한 설정 가이드입니다.

## 📋 지원하는 서버들

- **GitHub** - 코드 저장소 관리
- **Notion** - 문서/데이터베이스 관리
- **Slack** - 팀 커뮤니케이션
- **Make** - 워크플로우 자동화
- **Firecrawl** - 웹 크롤링
- **Perplexity** - AI 검색

## 🛠️ 기본 설정

### Claude Desktop 설정 파일 위치:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

## 🔑 각 서버별 인증 설정

### 1. 🐙 GitHub Personal Access Token

#### 토큰 생성:
1. GitHub → **Settings** → **Developer settings**
2. **Personal access tokens** → **Tokens (classic)**
3. **Generate new token** → **Generate new token (classic)**
4. 다음 권한들을 **모두 체크**:
   - `repo` (전체)
   - `workflow`
   - `write:packages`
   - `delete:packages`
   - `admin:org`
   - `admin:public_key`
   - `admin:repo_hook`
   - `admin:org_hook`
   - `gist`
   - `notifications`
   - `user`
   - `delete_repo`
   - `write:discussion`
   - `admin:enterprise`
5. **Generate token** → 토큰 복사

#### 설정값:
```
GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

### 2. 📝 Notion Integration Token

#### Integration 생성:
1. [Notion Developers](https://www.notion.so/my-integrations) 접속
2. **+ New integration** 클릭
3. **Name** 입력 → **Submit**
4. **Internal Integration Token** 복사 (`secret_`로 시작)

#### 워크스페이스 연결:
1. 사용할 Notion 페이지 우상단 **⋯** 클릭
2. **Add connections** → 생성한 Integration 선택

#### 설정값:
```
NOTION_API_KEY: "secret_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

### 3. 💬 Slack Bot Token

#### 앱 생성:
1. [Slack API](https://api.slack.com/apps) → **Create New App**
2. **From scratch** → 앱명/워크스페이스 설정

#### 설정 방법:
1. **OAuth & Permissions** 메뉴 진입

#### Scopes 섹션:
2. **Bot Token Scopes**에서 **Add an OAuth Scope** 클릭
3. 다음 권한들을 **하나씩 개별적으로** 추가:
   - `channels:read` → **Add** 클릭
   - `channels:history` → **Add** 클릭
   - `groups:read` → **Add** 클릭
   - `groups:history` → **Add** 클릭
   - `chat:write` → **Add** 클릭
   - `users:read` → **Add** 클릭
   - `reactions:read` → **Add** 클릭

#### OAuth Tokens 섹션:
4. **Install to [워크스페이스명]** 버튼 클릭 → **Allow**
5. **Bot User OAuth Token** 복사 (`xoxb-`로 시작)

#### Team ID 확인:
1. 웹 브라우저에서 슬랙 워크스페이스 접속
2. URL에서 `T`로 시작하는 문자열 확인
   ```
   https://app.slack.com/client/T09049C5C9G/C09049C70LS
                              ↑ 이 부분이 Team ID
   ```

#### 설정값:
```
SLACK_BOT_TOKEN: "xoxb-xxxxxxxxxxxx-xxxxxxxxxxxx-xxxxxxxxxxxxxxxxxxxxxxxx"
SLACK_TEAM_ID: "T01234567"
SLACK_CHANNEL_IDS: "C01234567,C76543210" (선택사항 - 특정 채널만 접근 제한)
```

### 4. 🔄 Make.com MCP Token

#### MCP 토큰 생성:
1. Make.com 로그인 → 우상단 프로필 클릭
2. **API** → **MCP Server** 탭
3. **Generate new token** → 토큰명 입력
4. **Token**, **Zone** 확인

#### 설정값:
```
URL: "https://<MAKE_ZONE>/mcp/api/v1/u/<MCP_TOKEN>/sse"
예시: "https://us1.make.com/mcp/api/v1/u/abc123token/sse"
```

### 5. 🔥 Firecrawl API Key

#### API 키 발급:
1. [Firecrawl](https://firecrawl.dev) 회원가입/로그인
2. 대시보드 → **API Keys** 메뉴
3. **Create API Key** → 키 복사

#### 설정값:
```
FIRECRAWL_API_KEY: "fc-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

### 6. 🔍 Perplexity API Key

#### API 키 발급:
1. [Perplexity](https://www.perplexity.ai) 로그인
2. **Settings** → **API** → **Generate**
3. API 키 복사

#### 설정값:
```
PERPLEXITY_API_KEY: "pplx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## 📁 최종 설정 파일

아래 JSON을 `claude_desktop_config.json`에 붙여넣고 각 토큰을 교체하세요:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>"
      }
    },
    "notionApi": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "OPENAPI_MCP_HEADERS": "{\"Authorization\": \"Bearer <YOUR_NOTION_API_KEY>\", \"Notion-Version\": \"2022-06-28\" }"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "<YOUR_SLACK_BOT_TOKEN>",
        "SLACK_TEAM_ID": "<YOUR_SLACK_TEAM_ID>",
        "SLACK_CHANNEL_IDS": "<YOUR_SLACK_CHANNEL_IDS>"
      }
    },
    "make": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://<MAKE_ZONE>/mcp/api/v1/u/<MCP_TOKEN>/sse"
      ]
    },
    "mcp-server-firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "<YOUR_FIRECRAWL_API_KEY>"
      }
    },
    "perplexity": {
      "command": "npx",
      "args": ["-y", "server-perplexity-ask"],
      "env": {
        "PERPLEXITY_API_KEY": "<YOUR_PERPLEXITY_API_KEY>"
      }
    }
  }
}
```

## 🔧 설정 완료 후

1. **Claude Desktop 재시작**
2. 새 채팅에서 "도구 목록 보여줘" 또는 MCP 서버 기능 테스트
3. 각 서버별 도구들이 정상적으로 표시되는지 확인

## 💡 선택적 사용

- 필요 없는 서버는 해당 블록 전체 삭제
- `SLACK_CHANNEL_IDS`는 선택사항 (제거 가능)
- Make URL은 전체 URL로 교체

## 🎯 지원하는 기능들

### GitHub
- 저장소 생성/관리
- 이슈/PR 관리
- 코드 검색
- 파일 업로드/수정

### Notion
- 페이지 생성/수정
- 데이터베이스 쿼리
- 블록 조작
- 댓글 관리

### Slack
- 메시지 송수신
- 채널 관리
- 반응 추가
- 사용자 정보 조회

### Make
- 워크플로우 자동화
- 서비스 연동

### Firecrawl
- 웹 페이지 크롤링
- 구조화된 데이터 추출
- 대량 웹사이트 분석

### Perplexity
- AI 기반 웹 검색
- 실시간 정보 조회

---

**🚀 이제 Claude Desktop에서 모든 MCP 서버를 활용해보세요!**

문제가 있거나 추가 도움이 필요하시면 이슈를 생성해주세요.
