# claude-harness

사이드 프로젝트에 바로 까는 개인용 Claude Code 하네스입니다. AI가 길게 일해도 규칙이 흐려지지 않게, **판단이 필요한 건 짧은 지침으로 권하고 판정할 수 있는 건 훅 · CI로 강제**합니다.

## 적용하는 법

새 프로젝트에서 Claude Code에게 이렇게만 말합니다.

```
이 하네스 적용해줘: github.com/gasic37200/claude-harness
```

AI가 [INSTALL.md](INSTALL.md)를 읽고 점검 → 플러그인 설치 → 템플릿 복사 → git 훅 → CI → 확인까지 진행합니다. 단계마다 바꿀 목록을 먼저 보여 주고 승인을 받으며, 이미 있는 파일은 덮어쓰지 않습니다.

직접 설치하려면:

```
/plugin install harness --marketplace gasic37200/claude-harness
```

그다음 프로젝트에서 `/harness:setup`, 확인은 `/harness:doctor`.

## 들어 있는 것 (v0.1.0)

| 층 | 무엇 | 어디 | 강제 정도 |
|---|---|---|---|
| 지침 | 30줄 `AGENTS.md` (명령 · 폴더 지도 · 금지 · 승인 범위) + `CLAUDE.md`(`@AGENTS.md`) | `templates/` | 권고 |
| 결정 | `docs/decisions.md`: 근거 · 누가(나 · AI · 측정) · 상태(유효 · 대체됨 · 철회) · 대체 관계 | `templates/docs/` | 권고 |
| 리뷰 | PR 템플릿 「AI 사용」 칸 (위험도 · 내가 정한 것 · 내 예측 · 검증) | `templates/.github/` | 판단 보조 |
| 강제 | `pre-push` 훅: main · master 직접 push 차단 (`HARNESS_SKIP=1`로 탈출) | `templates/.githooks/` | 강제(로컬) |
| 검증 | 재사용 CI: 테스트 실행 → 모듈별 통과 · 실패 · 시간 · 느린 테스트를 PR 댓글 하나로 | `.github/workflows/ci.yml` | 강제(CI) |
| 설치 | `/harness:setup` (단계별 승인) · `/harness:doctor` (상태표) | `skills/` | — |

### 같은 마켓에서 고를 수 있는 것

이 레포의 마켓(`claude-harness`)에 원본 레포를 가리키게 등록만 해 두었습니다(복사하지 않음). 층마다 도구는 하나만 씁니다.

| 플러그인 | 층 | 설치 |
|---|---|---|
| [ponytail](https://github.com/DietrichGebert/ponytail) | 구현 규율 | `/plugin install ponytail@claude-harness` |
| [mattpocock-skills](https://github.com/mattpocock/skills) | 결정 · 테스트 (grilling · tdd) | `/plugin install mattpocock-skills@claude-harness` |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 조율 | `/plugin install oh-my-claudecode@claude-harness` |

## CI만 쓰기

```yaml
# .github/workflows/ci.yml
name: ci
on: [pull_request]
permissions:
  contents: read
  pull-requests: write
jobs:
  test:
    uses: gasic37200/claude-harness/.github/workflows/ci.yml@v0.1.0
    with:
      test-command: "./gradlew test"
      java-version: "21"
```

입력값: `test-command`(필수) · `java-version` · `node-version` · `junit-glob` · `slowest`. 필수 체크 이름은 `test / test`입니다.

## 앞으로 (버전을 올려 가며)

카드 자동 생성 · SessionStart 카드 주입 · PostToolUse 린트 훅 · `recall` · `decide` · `risk-check` 스킬 · 위험도 라벨 · 실패 시 AI 분석 · 시각 diff.

## 제거

[INSTALL.md 「제거」](INSTALL.md#제거)를 보세요.

## 라이선스

MIT
