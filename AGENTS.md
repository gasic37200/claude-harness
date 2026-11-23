# AGENTS.md

이 레포는 Claude Code 플러그인 `harness`이자 마켓 `claude-harness`입니다. 다른 프로젝트에 설치되는 파일을 고치는 곳이므로, 바꾼 것은 설치한 모든 프로젝트에 영향이 갑니다.

## 구조

- `.claude-plugin/plugin.json` — 플러그인 이름 · 버전, `marketplace.json` — 이 레포 + 외부 플러그인 목록
- `skills/` — `setup`(설치) · `doctor`(점검)
- `templates/` — 대상 레포에 복사되는 파일 (경로 그대로, `.tmpl`은 뗌)
- `.github/workflows/ci.yml` — 재사용 CI (`workflow_call`)
- `INSTALL.md` — AI용 설치 절차, `README.md` — 사람용

## 규칙

- 고친 뒤 `claude plugin validate .` 가 통과해야 합니다
- 릴리스: `plugin.json`의 `version`을 올리고, `INSTALL.md` · `README.md` · `templates/.github/workflows/ci.yml.tmpl`의 버전 표기를 맞춘 뒤, 같은 이름의 태그(`v0.2.0`)를 답니다. 버전을 안 올리면 설치한 쪽이 업데이트를 받지 못합니다
- 이미 낸 태그는 옮기거나 지우지 않습니다 (CI가 `@v0.1.0`으로 고정해 씀)
- 스킬은 대상 레포의 기존 파일을 덮어쓰지 않고, 바꾸기 전에 목록을 보여 주고 승인받는 원칙을 지킵니다
- 훅 · 스크립트 경로는 `${CLAUDE_PLUGIN_ROOT}`, 상태 저장은 `${CLAUDE_PLUGIN_DATA}`. Windows를 위해 경로는 큰따옴표로 감쌉니다
- 외부 플러그인(ponytail · mattpocock · OMC)은 복사하지 않고 `marketplace.json`에 등록만 합니다
- 비밀값은 어떤 파일에도 쓰지 않습니다
