---
name: setup
description: 지금 레포에 harness 템플릿(AGENTS.md · 결정 기록 · PR 템플릿 · git 훅 · CI)을 설치한다. 사용자가 「하네스 적용해줘」「harness setup」처럼 직접 설치를 요청했을 때만 쓴다. 단계마다 바꿀 목록을 먼저 보여 주고 승인받는다.
---

# harness setup

지금 작업 폴더의 git 레포에 하네스를 설치합니다. 원칙:

- **보여 주고 승인**: 단계마다 바꿀 것을 표로 먼저 보여 주고, 사용자가 「응」이라고 한 단계만 실행합니다. 한 단계의 승인은 다음 단계에 쓰지 않습니다.
- **덮어쓰기 금지**: 이미 있는 파일은 절대 덮어쓰지 않습니다. 내용이 다르면 차이를 보여 주고 합칠 부분을 제안만 합니다.
- **다시 실행해도 안전**: 이미 되어 있는 단계는 「건너뜀 (이미 있음)」으로 표에 적고 넘어갑니다.
- **기본 브랜치에 직접 커밋 금지**: 바뀐 파일은 마지막에 새 브랜치 하나에 커밋합니다.
- 템플릿 원본: `${CLAUDE_PLUGIN_ROOT}/templates/`, 하네스 버전: `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`의 `version`

## 0. 점검 (아무것도 바꾸지 않음)

1. `git rev-parse --show-toplevel`로 레포 루트를 찾습니다. git 레포가 아니면 멈추고 알립니다.
2. 아래를 표로 보고합니다.

| 항목 | 확인 방법 |
|---|---|
| 스택 | `build.gradle*` · `pom.xml` · `package.json` · `pyproject.toml` 등 |
| 기본 브랜치 | `git symbolic-ref refs/remotes/origin/HEAD` (없으면 현재 브랜치) |
| 지금 브랜치 · 변경 | `git status --short --branch` |
| AGENTS.md · CLAUDE.md | 있음 / 없음, 줄 수 |
| git 훅 | `git config --get core.hooksPath`, `.git/hooks/`에 `.sample` 아닌 파일, `.husky/` |
| CI | `.github/workflows/` 파일 목록 |

작업 트리에 커밋 안 된 변경이 있으면 그대로 두고, 7단계에서 그 파일들은 커밋에 넣지 않는다고 알립니다.

## 1. 템플릿 파일 복사

`${CLAUDE_PLUGIN_ROOT}/templates/` 아래 파일을 같은 상대 경로로 레포 루트에 놓습니다. 이름 끝의 `.tmpl`은 뗍니다.

| 템플릿 | 놓을 곳 |
|---|---|
| `AGENTS.md.tmpl` | `AGENTS.md` |
| `CLAUDE.md` | `CLAUDE.md` (한 줄 `@AGENTS.md`) |
| `docs/decisions.md` | `docs/decisions.md` |
| `.github/pull_request_template.md` | `.github/pull_request_template.md` |
| `.githooks/pre-push` | `.githooks/pre-push` |

CI 파일(`.github/workflows/ci.yml.tmpl`)은 여기서 복사하지 않고 4단계에서 다룹니다.

1. 각 파일을 「새로 만듦 / 건너뜀 (같음) / 이미 있음 (다름)」으로 분류해 표로 보여 주고 승인받습니다.
2. 승인받으면 「새로 만듦」만 Write로 만듭니다. 줄바꿈은 LF로 둡니다.
3. 「이미 있음 (다름)」은 손대지 않고 차이 요약과 합칠 제안만 적습니다. 특히:
   - `CLAUDE.md`가 이미 있으면 맨 위에 `@AGENTS.md` 한 줄을 넣을지 묻습니다.
   - `AGENTS.md`가 이미 있으면 템플릿의 「결정 · 금지 · 승인」 절 중 없는 것만 제안합니다.
4. `.githooks/pre-push`를 새로 만들었으면 실행 권한을 줍니다: `git add --chmod=+x .githooks/pre-push` (Windows에서도 커밋에 실행 비트가 남음), macOS · Linux면 `chmod +x .githooks/pre-push`도.

## 2. AGENTS.md 자리표시 채우기

새로 만든 `AGENTS.md`에서 `{...}` 자리표시를 찾아 채웁니다.

| 자리표시 | 찾는 곳 |
|---|---|
| `{빌드 명령}` · `{테스트 명령}` · `{실행 명령}` | `gradlew` · `package.json`의 `scripts` · `Makefile` · README |
| `{폴더 지도}` | 최상위 폴더 3~6개와 역할 |
| `{프로젝트 한 줄 설명}` | README 첫 문단 |

- 레포에서 확신할 수 없는 값은 지어내지 말고 모아서 한 번에 묻습니다. 답이 없으면 `{...}`를 그대로 두고 보고합니다.
- 채운 빌드 · 테스트 명령은 한 번 실행해 통과하는지 확인합니다(오래 걸리면 백그라운드로). 실패하면 결과를 그대로 보고합니다.
- 30줄을 넘기지 않습니다.

## 3. git 훅 켜기

1. `git config --get core.hooksPath`가 이미 `.githooks`면 건너뜁니다.
2. 다른 값(예: `.husky`)이 있거나 `.git/hooks/`에 직접 만든 훅이 있으면 **바꾸지 않고** 멈춰서 알립니다. 바꾸면 기존 훅이 꺼지기 때문입니다.
3. 비어 있으면 「`git config core.hooksPath .githooks` 를 실행합니다 (이 PC의 이 레포에만 적용, 다른 사람은 각자 setup 필요)」라고 보여 주고 승인받은 뒤 실행합니다.
4. 확인: `git config --get core.hooksPath` → `.githooks`

## 4. CI 한 줄

1. `${CLAUDE_PLUGIN_ROOT}/templates/.github/workflows/ci.yml.tmpl`을 읽어 `{하네스 버전}`을 plugin.json의 `version`으로, `{테스트 명령}`을 2단계의 테스트 명령으로 바꿉니다. 스택이 Java면 `java-version`, Node면 `node-version` 줄의 주석을 풀고 값을 채웁니다. `push.branches`는 0단계의 기본 브랜치로 바꿉니다.
2. `.github/workflows/ci.yml`이 없으면 그 내용을 보여 주고 승인받아 만듭니다.
3. 이미 있으면 덮어쓰지 않습니다. 기존 파일에 harness job을 더하는 diff를 제안만 합니다.
4. 알릴 것:
   - 브랜치 보호의 필수 체크 이름은 `test / test` (쓰는 쪽 job 이름 / 하네스 job 이름)
   - 비밀값은 필요 없음 (기본 `GITHUB_TOKEN`만 씀). 나중에 비밀값이 필요한 기능이 생기면 이름만 알리고 값은 사용자가 GitHub에 넣습니다.

## 5. 선택 플러그인

아래 설치 여부를 묻고, 원하는 것만 설치 명령을 안내합니다(이 스킬이 직접 설치하지 않음). 구현 규율은 포니테일 하나만 씁니다.

| 플러그인 | 층 | 설치 |
|---|---|---|
| ponytail | 구현 규율 | `/plugin install ponytail@claude-harness` |
| mattpocock-skills | 결정 · 테스트 (grilling · tdd) | `/plugin install mattpocock-skills@claude-harness` |
| oh-my-claudecode | 조율 | `/plugin install oh-my-claudecode@claude-harness` |

## 6. 점검

`/harness:doctor`를 실행해 상태표를 보여 줍니다.

## 7. 커밋

1. 바뀐 파일 목록을 보여 주고 승인받습니다.
2. 지금 브랜치가 기본 브랜치면 `chore/harness-setup` 브랜치를 새로 만듭니다(이미 있으면 `-2`를 붙임).
3. 이 스킬이 만든 파일만 `git add` 하고 `Chore: harness v{버전} 설치` 로 커밋합니다. push · PR은 사용자가 원할 때만 합니다.

## 끝에 보고

| 단계 | 결과 |
|---|---|
| 0~7 각각 | ✅ 함 / ⏭️ 건너뜀 (이유) / ⚠️ 사용자 확인 필요 |

그리고 남은 자리표시, 덮어쓰지 않고 제안만 한 파일, 실행해 보지 못한 것을 따로 적습니다.
