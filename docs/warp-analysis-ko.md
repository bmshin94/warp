# Warp 저장소 분석 및 활용 검토 (한국어)

> 작성일: 2026-09-08
> 이 문서는 Warp 저장소를 처음 받은 뒤 진행한 분석·검토 내용을 정리한 개인 학습/기획 노트입니다.

## 관련 GitHub 및 링크

| 구분 | 주소 |
| --- | --- |
| 이 저장소 (내 포크) | https://github.com/bmshin94/warp |
| 원본 저장소 (upstream) | https://github.com/warpdotdev/warp |
| 이슈 목록 | https://github.com/warpdotdev/warp/issues |
| 공식 홈페이지 | https://www.warp.dev |
| 다운로드 | https://www.warp.dev/download |
| 공식 문서 | https://docs.warp.dev |
| 기여자 대시보드 | https://build.warp.dev |
| Warp Factories | https://warp.dev/factories |

---

## 1. Warp란 무엇인가

Warp는 **"터미널에서 태어난 AI 개발 환경(Agentic Development Environment)"** 이다.

| | 기존 터미널 | Warp |
| --- | --- | --- |
| 동작 | 명령어를 입력하면 실행 | AI 에이전트가 작업을 대신 수행 |
| 구성 | 셸 + 화면 출력 | 셸 + AI 에이전트 + 자체 GPU UI |

이 저장소는 완성된 앱이 아니라 **앱의 전체 소스코드와 개발 문서**다.

### 최근 주목받는 이유

1. 상용 제품의 클라이언트 코드를 통째로 오픈소스로 공개했다.
2. OpenAI가 창립 스폰서로 참여하고 있다.
3. **AI 에이전트가 실제로 이 저장소의 개발 프로세스를 운영한다.**
   이슈 분류 → 스펙 작성 → 구현 → PR 리뷰까지 에이전트가 수행하고, 사람은 최종 확인을 맡는다.

---

## 2. 저장소 구조 분석

### 실측 규모

| 항목 | 수치 |
| --- | --- |
| 전체 용량 | 약 523 MB |
| Rust 소스 파일 (`crates/`, `app/`) | 4,031개 |
| 크레이트(모듈) 수 | 81개 |
| 스펙 문서 폴더 (`specs/`) | 286개 |
| AI 에이전트 스킬 | 21개 |
| GitHub Actions 워크플로 | 23개 |

### 주요 디렉터리

| 경로 | 설명 |
| --- | --- |
| `crates/` | 핵심 모듈 81개 (터미널 엔진, AI, UI 프레임워크 등) |
| `app/` | GUI 데스크톱 앱 본체 |
| `specs/` | 기능별 기획서 286개 (`PRODUCT.md` + `TECH.md` 구조) |
| `agents/specs/` | 에이전트가 작성한 최신 스펙 문서 |
| `.agents/skills/`, `.claude/skills/` | AI 에이전트 스킬 21개 |
| `.warp/workflows/` | Warp Drive 워크플로 정의 (YAML) |
| `.github/workflows/` | CI/CD 자동화 23개 |
| `script/` | 빌드·번들·검사 스크립트 (플랫폼별 하위 디렉터리 포함) |
| `docker/` | Linux 개발용 컨테이너 정의 |
| `AGENTS.md` | 엔지니어링 가이드 (약 18KB) |
| `CONTRIBUTING.md` | 기여 가이드 (약 17KB) |

### 눈여겨볼 크레이트

| 크레이트 | 역할 |
| --- | --- |
| `warpui`, `warpui_core` | 자체 제작 UI 프레임워크 (GPU/WGSL 렌더링) |
| `warp_terminal` | 터미널 엔진 |
| `warp_tui` | 헤드리스 TUI 프런트엔드 (GPU 불필요) |
| `ai`, `ai_types`, `warp_multi_agent_client` | AI 에이전트 계층 |
| `mcp` | Model Context Protocol 지원 (외부 CLI 에이전트 연동) |
| `computer_use` | 화면 인식 기반 자동 조작 |
| `lsp`, `editor`, `vim` | 에디터 기능 |
| `voice_input` | 음성 입력 |

### 두 개의 프런트엔드

Warp는 `warp_core` / `warpui` 코어를 공유하는 두 프런트엔드를 가진다.

- **GUI 데스크톱 앱** (`app/`) — WarpUI 픽셀/GPU 프레임워크 기반. `cargo run` 또는 `./script/run`
- **헤드리스 TUI** (`crates/warp_tui`) — 셀 그리드 기반 콘솔 앱. `./script/run-tui`

### 스펙 주도 개발 (`specs/`)

각 스펙 폴더는 다음 두 문서로 구성된다.

```
specs/APP-1915/
├── PRODUCT.md   # 문제 정의, 비목표(non-goals), 사용자 관점 동작
└── TECH.md      # 구현 위치, 기술적 접근
```

"무엇을 만들지"와 "어떻게 만들지"를 분리해 문서화한 실무 예제 286개가 그대로 들어 있다.

### AI 에이전트 스킬 (`.agents/skills/`)

반복 업무를 AI에게 위임하기 위한 지침 문서 모음이다.

| 분류 | 스킬 |
| --- | --- |
| 기능 플래그 | `add-feature-flag`, `remove-feature-flag`, `promote-feature` |
| 테스트 | `rust-unit-tests`, `gui-integration-test`, `tui-testing` |
| 리뷰·트리아지 | `review-pr-local`, `triage-issue-local`, `dedupe-issue-local` |
| UI 가이드 | `gui-ui-guidelines`, `gui-settings-ui`, `tui-ui-guidelines` |
| 릴리스 | `changelog-draft`, `classify-changelog-pr` |
| 기타 | `add-telemetry`, `logging-and-error-reporting`, `cross-platform-cloud-verification` |

`changelog-draft` 스킬에는 실제 동작하는 Python 스크립트도 포함되어 있다.

---

## 3. 설치 및 사용법

### 경로 A — 앱으로 사용하기 (권장)

소스 빌드가 필요 없다.

```bash
# macOS
brew install --cask warp

# Windows
winget install Warp.Warp

# Linux — 배포판에 맞는 패키지를 https://www.warp.dev/download 에서 받는다
sudo dpkg -i warp-terminal_*.deb   # 예: Ubuntu/Debian
```

Linux는 `.deb`, `.rpm`, AUR, `.AppImage` 네 가지로 배포된다
(`script/linux/bundle_deb`, `bundle_rpm`, `bundle_arch`, `bundle_appimage` 참고).

#### 핵심 단축키

| 단축키 (macOS / Windows·Linux) | 기능 |
| --- | --- |
| `Cmd+P` / `Ctrl+Shift+P` | 명령 팔레트 |
| `` Cmd+` `` / `` Ctrl+` `` | AI에게 질문 |
| `Cmd+D` / `Ctrl+Shift+D` | 화면 분할 |
| `Cmd+T` / `Ctrl+T` | 새 탭 |
| `Cmd+R` / `Ctrl+R` | 명령어 기록 검색 |
| `Cmd+,` / `Ctrl+,` | 설정 |

#### 주요 기능

- **블록(Block)** — 명령어 하나가 카드 단위로 묶여 출력이 섞이지 않는다.
- **AI 명령어 생성** — 자연어로 설명하면 명령어를 만들어 준다.
- **에이전트 모드** — 테스트 실행 → 에러 분석 → 수정 → 재실행을 자동 수행.
- **Warp Drive** — 자주 쓰는 명령어를 저장·공유 (`.warp/workflows/*.yaml` 형식).
- **외부 에이전트 연동** — MCP를 통해 Claude Code, Codex, Gemini CLI 등을 Warp 안에서 실행.

### 경로 B — 소스에서 빌드하기

> 첫 빌드에 1~2시간, 디스크 수십 GB가 필요하다. 코드를 직접 수정할 목적이 아니라면 경로 A를 권장한다.

#### 사전 준비

**macOS**

- Xcode 설치 (Metal 툴체인은 bootstrap이 처리)
- Homebrew 설치

**Linux (Debian 계열)** — `script/bootstrap`이 아래를 자동 설치한다.

```
build-essential cmake pkg-config curl git
libssl-dev libfreetype-dev libexpat1-dev libgit2-dev
libfontconfig1-dev libasound2-dev libclang-dev clang-format musl-tools
jq brotli python-is-python3
protoc 25.1 (별도 다운로드 설치)
```

Debian 계열이 아니면 경고만 출력되므로 위 목록을 수동 설치해야 한다.

**Windows** — 관리자 권한 PowerShell에서 `.\script\windows\bootstrap.ps1` 실행.
설치 파일(.exe) 생성에는 [Inno Setup](https://jrsoftware.org/isdl.php)이 추가로 필요하다.

#### 빌드 및 실행

```bash
cd ~/warp

# 환경 세팅 (Rust 1.92.0 자동 설치 — rust-toolchain.toml에 고정)
./script/bootstrap

# 실행
./script/run        # GUI 데스크톱 앱
./script/run-tui    # 헤드리스 TUI
```

`bootstrap`은 sudo 비밀번호, 공용 에이전트 스킬 설치 위치, gcloud 인증을 물어본다.
개인 학습 목적이라면 다음 조합이 편하다.

```bash
./script/bootstrap -y --skip-common-skills --skip-gcloud-auth
```

#### 개발용 명령어

```bash
./script/presubmit    # 포맷 + 린트 + 테스트 (PR 전 필수)
./script/format       # 코드 포맷팅
cargo nextest run --no-fail-fast --workspace --exclude command-signatures-v2
cargo clippy --workspace --all-targets --all-features --tests -- -D warnings
cargo bundle --bin warp   # 앱 번들 생성
```

#### Docker로 Linux 개발 환경 구성 (macOS 호스트)

```bash
brew install --cask xquartz
defaults write org.xquartz.X11 enable_iglx -bool true

docker build -t warp-client-linux-dev docker/linux-dev
docker run -dp 127.0.0.1:22:22/tcp \
  -v /Users/$USER/src:/src \
  -v $HOME/.ssh:/home/dev/.ssh \
  warp-client-linux-dev

xhost +localhost
ssh dev@localhost          # 비밀번호: password
cd /src && cargo run --features fast_dev
```

### 기여(PR) 흐름

```
이슈 등록
   ↓
메인테이너가 라벨 부착
   ├─ ready-to-spec      → specs/ 에 스펙 PR 먼저
   └─ ready-to-implement → 코드 PR 가능
   ↓
브랜치 생성 (형식: <핸들>/<기능명>, 예: bmshin94/fix-parser)
   ↓
./script/presubmit 통과 확인
   ↓
PR 생성 → AI 에이전트(Oz)가 1차 리뷰
   ↓
Oz 승인 후 사람 전문가에게 자동 배정
```

- 리뷰 반영 후 PR에 `/warp-agent-review` 코멘트로 재검토 요청 (PR당 3회)
- 스크린샷 / 화면 녹화 첨부가 사실상 필수
- 14일간 활동이 없으면 자동 종료 (7일·10일에 사전 알림)

### 자주 발생하는 문제

| 증상 | 해결 |
| --- | --- |
| `linker cc not found` | Linux 빌드 의존성 미설치 → `./script/bootstrap` 재실행 |
| `protoc: command not found` | protoc 3.15 이상 수동 설치 필요 |
| `No space left on device` | `cargo clean`으로 `target/` 정리 |
| 빌드 중 시스템 정지 | `cargo build -j 4`로 병렬 작업 수 제한 |
| Metal / GPU 오류 (macOS) | Xcode 설치 + `xcodebuild -downloadComponent MetalToolchain` |
| `install_channel_config` 실패 | 사내 전용 설정이므로 OSS 빌드에서는 정상 동작 |

---

## 4. 라이선스

| 대상 | 라이선스 | 의미 |
| --- | --- | --- |
| `warpui_core`, `warpui` 크레이트 | **MIT** | 상업적 이용 자유, 소스 공개 의무 없음 |
| 그 외 전체 | **AGPL v3** | 네트워크 서비스로 제공만 해도 소스 공개 의무 발생 |

추가로 기여 시 CLA가 적용된다.

**정리**

- 읽고 학습하는 것 — 자유
- 방법론(스펙 작성 방식, 스킬 구조 등)을 따라 하는 것 — 자유
- 코드를 복사해 상용 제품에 사용하는 것 — AGPL 의무 발생, 주의 필요

> 실제 사업화 시에는 반드시 법률 검토를 받을 것.

---

## 5. 수익화 아이디어 검토

### 전제: 코드 자체로는 수익화가 어렵다

| 장벽 | 내용 |
| --- | --- |
| AGPL v3 | SaaS로 제공해도 소스 공개 의무 (네트워크 조항) |
| 경쟁 상대 | Warp 본사와 동일 제품으로 경쟁하는 구조 |
| 상표 | "Warp" 이름·로고 사용 불가 |

### 실제 자산은 "방법론"

라이선스와 무관하게 배워서 활용할 수 있는 것들:

- `AGENTS.md` 작성 방식
- `.claude/skills` 설계 방식
- `specs/` 스펙 주도 개발 프로세스
- AI 기반 PR 리뷰 / 이슈 트리아지 워크플로
- CI 자동화 구성

### 아이디어 목록

| # | 아이디어 | 난이도 | 초기 비용 | 비고 |
| --- | --- | --- | --- | --- |
| 1 | **AI 개발환경 셋업 컨설팅** | 낮음 | 0원 | 팀별 `AGENTS.md` + 스킬 제작 + 교육. 최우선 추천 |
| 2 | **한국어 콘텐츠 (블로그/영상/강의)** | 낮음 | 0원 | 국내 자료 부재 → 선점 효과 |
| 3 | **Skills 템플릿 팩 판매** | 중간 | 낮음 | 도메인별 번들, Gumroad 등에서 판매 |
| 4 | **AGENTS.md 자동 생성 SaaS** | 중간 | 중간 | Warp 코드 미사용 → 라이선스 무관 |
| 5 | **MIT 크레이트(`warpui`) 활용** | 높음 | 중간 | Rust GPU UI 프레임워크. 상업적 이용 가능 |
| 6 | **오픈소스 기여 → 커리어 자산** | 중간 | 0원 | `build.warp.dev` 기여자 등재로 증명 가능 |
| 7 | **AI 온보딩 문서 대행** | 낮음 | 0원 | 사람 신입 + AI 에이전트 공용 온보딩 문서 통합 |

### 추천 실행 순서

```
1~2개월차  학습 + 실습
           스킬 21개 정독 → 내 프로젝트에 직접 적용 → 효과 측정

2~4개월차  콘텐츠로 신뢰 확보 (무료 공개)
           블로그·영상 시리즈, 스킬 템플릿 GitHub 공개

4~6개월차  수익화
           컨설팅 수주, 유료 템플릿/강의 출시
```

초기 비용이 0원이고, 진행하면서 실력이 축적되며, 이후 어떤 방향으로 확장하든 연결된다는 점이 장점이다.

### 주의사항

- Warp 코드 복사 금지, 방식 참고만 할 것
- "Warp 공식/파트너" 등 사칭 금지
- "Warp 사례 기반"이라는 표현까지가 적절한 선

---

## 6. React / PHP 기술 검토

### 결론 요약

| 질문 | 답 |
| --- | --- |
| Warp 같은 터미널을 React/PHP로 만들 수 있나? | **불가능** |
| 위 수익화 아이디어(SaaS)를 React/PHP로 만들 수 있나? | **가능하며, 오히려 적합** |

### 터미널 자체가 불가능한 이유

| 필요 조건 | React/PHP |
| --- | --- |
| OS 프로세스 직접 제어 (PTY, 셸 실행) | 브라우저 샌드박스 밖 |
| GPU 기반 고빈도 렌더링 | 네이티브 성능 불가 |
| 로컬 파일시스템 직접 접근 | 브라우저 보안 정책이 차단 |
| 상시 실행되는 데스크톱 프로세스 | PHP는 요청-응답 모델 |

Warp가 Rust로 작성된 이유가 여기에 있다.
xterm.js + WebSocket + SSH 조합으로 "웹 터미널"은 만들 수 있으나, 이는 별개의 제품군이다.

### SaaS(아이디어 4)는 React/PHP가 적합

만들려는 것은 터미널이 아니라 웹 서비스이므로 일반적인 웹 스택으로 충분하다.

```
저장소 URL 입력 → 코드 분석 → AI가 AGENTS.md + skills 생성 → PR 생성
```

#### 아키텍처

```
React (프론트엔드)
  로그인 · 저장소 목록 · 결과 미리보기
        │  REST API
        ▼
Laravel / PHP (백엔드)
  1. GitHub OAuth 로그인
  2. 저장소 스캔 (GitHub API)
  3. Claude API 호출
  4. PR 자동 생성
  5. 결제 처리
        │
   ┌────┴────┐
 MySQL    Redis Queue
(사용자·이력)  (장시간 작업)
```

#### 스택

| 영역 | 선택 | 이유 |
| --- | --- | --- |
| 프런트엔드 | React + Vite + Tailwind | 개발 속도 |
| 백엔드 | Laravel (PHP 8.3+) | 인증·큐·결제·스케줄러 내장 |
| DB | MySQL / PostgreSQL | — |
| 큐 | Laravel Queue + Redis | 스캔이 장시간 작업이므로 필수 |
| AI | Anthropic PHP SDK (`anthropic-ai/sdk`) | 공식 SDK |
| GitHub | `knplabs/github-api` | PHP용 GitHub 클라이언트 |
| 결제 | Laravel Cashier(Stripe) / 토스페이먼츠 | — |
| 배포 | 프런트 Vercel + 백엔드 VPS | 초기 월 2만원 수준 |

#### 설치

```bash
composer require anthropic-ai/sdk
composer require knplabs/github-api
```

#### 핵심 코드 (Laravel Job)

```php
<?php

namespace App\Jobs;

use Anthropic\Client;
use Illuminate\Contracts\Queue\ShouldQueue;

class GenerateAgentsMd implements ShouldQueue
{
    public function __construct(
        private string $repoFullName,
        private array  $repoSummary,   // 언어·프레임워크·주요 파일 요약
    ) {}

    public function handle(): void
    {
        $client = new Client(apiKey: getenv('ANTHROPIC_API_KEY'));

        // 규칙·템플릿은 매 요청 동일하므로 캐싱해 비용을 줄인다
        $systemPrompt = <<<'TXT'
        You are a senior engineer who writes AGENTS.md files.
        Given a repository summary, produce a complete AGENTS.md covering:
        build commands, test commands, architecture overview, and code style.
        Output raw markdown only.
        TXT;

        $message = $client->messages->create(
            model: 'claude-opus-5',
            maxTokens: 16000,
            system: [
                [
                    'type'         => 'text',
                    'text'         => $systemPrompt,
                    'cacheControl' => ['type' => 'ephemeral'],
                ],
            ],
            messages: [
                [
                    'role'    => 'user',
                    'content' => json_encode($this->repoSummary),
                ],
            ],
        );

        // thinking 블록이 먼저 올 수 있으므로 블록 타입을 반드시 확인한다
        $markdown = '';
        foreach ($message->content as $block) {
            if ($block->type === 'text') {
                $markdown .= $block->text;
            }
        }

        CreatePullRequest::dispatch($this->repoFullName, $markdown);
    }
}
```

#### 프런트엔드 (React)

```jsx
function ScanForm() {
  const [repo, setRepo] = useState("");
  const [status, setStatus] = useState(null);

  const scan = async () => {
    const res = await fetch("/api/scan", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ repo }),
    });
    const { jobId } = await res.json();
    setStatus("분석 중...");
    pollUntilDone(jobId, setStatus);
  };

  return (
    <div>
      <input
        value={repo}
        onChange={(e) => setRepo(e.target.value)}
        placeholder="owner/repo"
      />
      <button onClick={scan}>AGENTS.md 만들기</button>
      {status && <p>{status}</p>}
    </div>
  );
}
```

#### 원가 계산

Claude API 가격 (100만 토큰 기준):

| 모델 | 입력 | 출력 |
| --- | --- | --- |
| Claude Opus 5 | $5 | $25 |
| Claude Sonnet 5 | $2 | $10 |
| Claude Haiku 4.5 | $1 | $5 |

저장소 1개 분석 기준 추정:

```
입력 50,000 토큰  → $0.25
출력  8,000 토큰  → $0.20
────────────────────────
합계             ≈ $0.45 (약 600원)
```

구독 $19/월, 월 10회 스캔 가정 시 원가 $4.5 → 마진율 약 76%.
시스템 프롬프트 캐싱으로 반복 호출 비용을 추가로 낮출 수 있다.
초기에는 Opus 5로 품질을 확보하고, 데이터가 쌓인 뒤 더 저렴한 모델과 품질을 직접 비교해 결정하는 편이 안전하다.

#### MVP 4주 계획

| 주차 | 작업 | 산출물 |
| --- | --- | --- |
| 1주 | Laravel + React 세팅, GitHub OAuth | 로그인 동작 |
| 2주 | 저장소 스캔 (파일 트리, `package.json` / `composer.json` 등) | JSON 요약 |
| 3주 | Claude API 연동, AGENTS.md 생성, PR 생성 | 동작하는 MVP |
| 4주 | 결제 연동 및 배포 | 첫 수익 |

> 이 SaaS는 Warp 코드를 전혀 사용하지 않으므로 AGPL 제약과 무관하다.
> 참고하는 것은 "AGENTS.md를 어떤 구조로 쓰는가"라는 아이디어뿐이다.

---

## 7. 다음 단계 후보

- [ ] `.agents/skills/` 21개 정독 후 내 프로젝트용 스킬 제작
- [ ] `specs/` 예제를 참고해 스펙 문서 작성 연습
- [ ] 한국어 콘텐츠 1편 작성 (주제: AI 에이전트가 운영하는 오픈소스)
- [ ] SaaS MVP 1주차 (Laravel + React 세팅, GitHub OAuth)
- [ ] `ready-to-implement` 라벨 이슈 중 하나 골라 기여 시도

---

## 부록: 명령어 치트시트

```bash
# 문서 읽기 (빌드 불필요)
cat AGENTS.md CONTRIBUTING.md FAQ.md

# 스킬 구조 확인
ls .agents/skills
cat .agents/skills/add-feature-flag/SKILL.md

# 스펙 작성 예제 확인
cat specs/APP-1915/PRODUCT.md
cat specs/APP-1915/TECH.md

# 워크플로 예제 확인
cat .warp/workflows/run_unit_test.yaml

# 빌드 및 실행
./script/bootstrap -y --skip-common-skills --skip-gcloud-auth
./script/run          # GUI
./script/run-tui      # TUI

# 검사
./script/presubmit
```
