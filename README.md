# context-skills

Claude Code용 컨텍스트 엔지니어링 skill 플러그인.

컨텍스트 윈도우는 휘발성 캐시다 — compact, `/clear`, 세션 종료로 언제든 사라진다.
이 플러그인은 **"상태는 컨텍스트가 아니라 디스크에"** 원칙을 Claude가 스스로
지키게 하는 두 개의 skill을 제공한다. Anthropic 공식 가이드
([Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents),
[Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents))의
원칙을 따른다.

## 포함된 Skills

| Skill | 언제 동작하나 | 무엇을 하나 |
|---|---|---|
| **context-engineering** | 정보를 어디에 둘지 결정할 때 — CLAUDE.md가 비대해질 때, /clear 후 진행 상태가 유실될 때, sub agent 출력이 컨텍스트를 범람시킬 때 | 정보 수명에 따른 배치 기준 강제: 안정적 전역 맥락은 CLAUDE.md, 절차는 skill, 진행 상태는 `.claude/state/`, 이력은 git |
| **loop-engineering** | 여러 세션에 걸친 장기·반복 작업을 오케스트레이션할 때 | supervisor 사이클 강제: 상태 파악 → 작업 하나 위임 → 검증 → 디스크 기록 + 커밋. `/clear` 후에도 무손실로 이어감 |

## 설치

### 로컬에서

```
/plugin marketplace add /path/to/context-skills
/plugin install context-skills@context-skills
```

### GitHub에서

```
/plugin marketplace add zkfmapf123/context-skills
/plugin install context-skills@context-skills
```

## 동작 방식

- 설치하면 skill **description만** 모든 세션에 상주한다 (~100토큰). 본문은 0토큰.
- 상황이 description과 매칭되면 Claude가 skill 본문을 로드하고 절차를 따른다.
- loop-engineering은 첫 실행 시 프로젝트 CLAUDE.md에 포인터 한 줄을 심는다
  (멱등 — 이미 있으면 건너뜀). 이후 세션은 skill 트리거를 놓쳐도 규율을 상기한다.
- 명시 호출도 가능: "loop-engineering 따라서 진행해".

## 상태 파일 구조

skill이 동작하면 프로젝트에 아래가 생긴다.

```
.claude/state/
├── tasks.json     # 작업 원장 (무엇이 남았나) — JSON, 함부로 덮어쓰이지 않음
└── progress.md    # 진행 로그 (무엇을 했나) — 사람이 읽는 Markdown
```

## License

MIT
