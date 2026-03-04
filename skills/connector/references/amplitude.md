# Amplitude 설정 가이드

Amplitude MCP는 프로덕트 분석 데이터(이벤트, 차트, 퍼널, 실험, 코호트 등)를 자연어로 조회·생성할 수 있는 원격 MCP 서버다. OAuth 인증을 사용하므로 토큰을 직접 관리할 필요가 없다.

## 설치 명령

아래 Bash 명령을 실행하여 Amplitude MCP를 설치한다:

```bash
claude mcp add -t http -s user amplitude "https://mcp.amplitude.com/mcp"
```

> EU 리전 사용자는 URL을 `https://mcp.eu.amplitude.com/mcp` 로 변경한다.

## OAuth 인증

Amplitude MCP는 **OAuth 2.0** 인증을 사용한다. 별도 토큰 발급이 필요 없다.

**중요**: Amplitude는 다른 MCP 서버와 달리, 설치 + 재시작만으로는 도구가 로드되지 않는다. OAuth 인증을 먼저 완료해야 도구가 활성화된다.

**인증 절차**:

**방금 `claude mcp add` 명령을 실행한 경우** (설치 직후):
1. Claude Code를 재시작한다. (새 설정을 인식하기 위해 필요)
2. 재시작된 세션에서 `/mcp` 를 입력한다.
3. 목록에서 `amplitude` 를 선택한다.
4. "Authenticate" 를 선택한다.
5. 브라우저에서 Amplitude 로그인 화면이 자동으로 열린다.
6. 로그인을 완료하면 인증이 연결된다.

**이미 설정이 되어 있는 경우** (재방문 또는 재시작 후 복귀):
1. 현재 세션에서 바로 `/mcp` 를 입력한다. (재시작 불필요)
2. 목록에서 `amplitude` 를 선택한다.
3. "Authenticate" 를 선택한다.
4. 브라우저에서 Amplitude 로그인 화면이 자동으로 열린다.
5. 로그인을 완료하면 인증이 연결된다.

- Amplitude 계정이 없으면, 조직 관리자에게 접근 권한을 요청한다.
- 인증 완료 후 도구가 활성화되며, 이후에는 토큰이 자동 갱신된다.

## 테스트 전 안내

Amplitude 연결 테스트 전에 반드시 아래 메시지를 먼저 출력한다:

"Amplitude 연결 테스트를 시작합니다. OAuth 인증이 아직 안 되어 있으면 `/mcp` → amplitude → Authenticate 순서로 먼저 로그인을 완료해주세요."

## 연결 테스트

**인증 완료 후** 테스트를 진행한다:
1. ToolSearch로 `+amplitude` 검색 → 도구가 나타나는지 확인
2. 도구가 나타나면 `mcp__amplitude__get_context()` 호출로 실제 테스트
3. 성공하면: 현재 사용자, 조직, 접근 가능한 프로젝트 정보 표시
4. 실패하면: 아래 트러블슈팅 항목 확인 → 진전 없으면 [트러블슈팅 및 이슈 등록](troubleshoot.md) 절차 진행

**도구가 나타나지 않으면**: OAuth 인증이 완료되지 않은 것. 위 인증 절차를 다시 안내한다.

## 트러블슈팅

연결 실패 시 확인할 것:
1. Claude Code를 재시작했는지 확인
2. `claude mcp add` 명령이 정상 실행되었는지 확인 (에러 메시지가 없었는지)
3. `~/.claude.json`(Mac/Linux) 또는 `%USERPROFILE%\.claude.json`(Windows)에 `amplitude` 설정이 반영되었는지 확인
4. Amplitude 계정이 있고 해당 프로젝트에 접근 권한이 있는지 확인
5. OAuth 인증 시 하나의 Amplitude 조직에만 로그인된 상태인지 확인 (여러 조직이 있으면 충돌 가능)
6. 브라우저가 OAuth 인증 팝업을 차단하지 않는지 확인

## 관련 URL

| 항목 | URL |
|------|-----|
| Amplitude MCP 서버 (US) | `https://mcp.amplitude.com/mcp` |
| Amplitude MCP 서버 (EU) | `https://mcp.eu.amplitude.com/mcp` |
| Amplitude MCP 공식 문서 | https://amplitude.com/docs/amplitude-ai/amplitude-mcp |
