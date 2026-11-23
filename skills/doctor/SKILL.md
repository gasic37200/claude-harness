---
name: doctor
description: 지금 레포의 harness 설치 상태를 점검해 ✅ · ❌ 표로 보여 준다. 아무것도 바꾸지 않는다. 「하네스 상태」「harness doctor」, 또는 setup이 끝난 뒤 확인할 때 쓴다.
---

# harness doctor

레포 루트(`git rev-parse --show-toplevel`)에서 아래를 하나씩 확인하고 표 하나로 보고합니다. **파일 · 설정을 바꾸지 않습니다.** 고칠 것이 있으면 「고치는 법」 칸에 적기만 합니다.

하네스 버전은 `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`의 `version`입니다.

| # | 항목 | ✅ 조건 | 고치는 법 |
|---|---|---|---|
| 1 | AGENTS.md | 파일이 있고 30줄 이하 | `/harness:setup` 1단계, 길면 줄이기 |
| 2 | 자리표시 | `AGENTS.md`에 `{` 로 시작해 `}` 로 끝나는 자리표시가 없음 | setup 2단계 |
| 3 | CLAUDE.md | 있고 `@AGENTS.md` 줄을 포함 | 맨 위에 `@AGENTS.md` 추가 |
| 4 | 결정 기록 | `docs/decisions.md`가 있음 | setup 1단계 |
| 5 | PR 템플릿 | `.github/pull_request_template.md`에 `## AI 사용` 이 있음 | setup 1단계 |
| 6 | pre-push 훅 | `.githooks/pre-push`가 있고 `git ls-files -s .githooks/pre-push`의 모드가 `100755` (아직 커밋 전이면 ⚠️) | `git add --chmod=+x .githooks/pre-push` |
| 7 | hooksPath | `git config --get core.hooksPath` 가 `.githooks` | setup 3단계 |
| 8 | CI | `.github/workflows/` 어딘가에 `gasic37200/claude-harness/.github/workflows/ci.yml@v` 가 있음 | setup 4단계 |
| 9 | CI 버전 | 8의 `@v` 뒤 버전이 하네스 버전과 같음 (다르면 ⚠️, 실패 아님) | 버전 올릴지 사용자에게 |
| 10 | 기본 브랜치 보호 | `gh api repos/{owner}/{repo}/rulesets` 에 기본 브랜치 ruleset이 있음 (gh가 없거나 권한이 없으면 ❔ 확인 못 함) | GitHub 설정 > Rules, 필수 체크 `test / test` |
| 11 | 선택: ponytail | `claude plugin list` 출력에 `ponytail` (없어도 ✅ 대신 ➖ 「설치 안 함」) | setup 5단계 |
| 12 | 선택: mattpocock-skills | 위와 같음 | setup 5단계 |

출력 형식:

```
harness doctor · v{버전} · {레포 이름}

| # | 항목 | 상태 | 메모 |
|---|---|---|---|
| 1 | AGENTS.md | ✅ | 24줄 |
...

요약: ✅ n · ❌ n · ⚠️ n · ❔ n · ➖ n
```

- ❌가 하나라도 있으면 맨 아래에 「다음에 할 일」을 위에서부터 한 줄씩 적습니다.
- 확인 명령이 실패하면 추측하지 말고 ❔ 와 실패 이유를 적습니다.
