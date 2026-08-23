# context-skills

Claude Code용 컨텍스트 엔지니어링 skill 플러그인.

컨텍스트 윈도우는 휘발성 캐시다 — compact, `/clear`, 세션 종료로 언제든 사라진다.
이 플러그인은 **"상태는 컨텍스트가 아니라 디스크에"** 원칙을 Claude가 스스로
지키게 한다. Anthropic 공식 가이드
([Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents),
[Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents))의
원칙을 따른다.

## 시작하기 (3단계)

### 1. 설치

```
/plugin marketplace add zkfmapf123/context_skills
/plugin install context-skills@context-skills
```

로컬에서 쓰려면 (이 repo를 clone한 경우):

```
/plugin marketplace add <clone한-절대경로>
/plugin install context-skills@context-skills
```

### 2. 프로젝트 초기화 (프로젝트마다 1회)

사용할 프로젝트에서:

```
/context-init
```

CLAUDE.md에 포인터 두 줄이 심어진다. CLAUDE.md는 매 세션 항상 로드되므로,
이후 세션은 별도 요청 없이도 이 규율을 상기한다. 여러 번 실행해도 안전하다(멱등).

### 3. 그냥 작업 시킨다

"이 기능 구현해줘, 여러 세션 걸릴 것 같아" 같은 장기 작업을 주면
loop-engineering이 자동으로 동작한다. 명시 호출도 가능:
"loop-engineering 따라서 진행해".

## 포함된 Skills

| Skill | 언제 동작하나 | 무엇을 하나 |
|---|---|---|
| **context-init** | `/context-init` 호출 시 (프로젝트마다 1회) | CLAUDE.md에 포인터를 심어 이후 세션이 규율을 자동 상기하게 만듦 |
| **context-engineering** | 정보를 어디에 둘지 결정할 때 — CLAUDE.md가 비대해질 때, /clear 후 진행 상태가 유실될 때, sub agent 출력이 컨텍스트를 범람시킬 때 | 정보 수명에 따른 배치 기준 강제: 안정적 전역 맥락은 CLAUDE.md, 절차는 skill, 진행 상태는 `.claude/state/`, 이력은 git |
| **loop-engineering** | 여러 세션에 걸친 장기·반복 작업을 오케스트레이션할 때 | supervisor 사이클 강제: 상태 파악 → 작업 하나 위임 → 검증 → 디스크 기록 + 커밋. `/clear` 후에도 무손실로 이어감 |

## 실제로 어떻게 굴러가나

```
설치
 └─ skill description만 모든 세션에 상주 (~100토큰, 본문은 0토큰)

/context-init  (1회)
 └─ CLAUDE.md에 포인터 2줄 → 매 세션 규율 상기

장기 작업 시작
 └─ loop-engineering 트리거
     ├─ 첫 세션: 요구사항 분해 → .claude/state/tasks.json 생성
     └─ 매 사이클: 상태 파악 → 작업 하나 → 검증 → 기록 + 커밋

컨텍스트가 차면
 └─ /clear → 새 세션이 .claude/state/ 를 읽고 무손실로 이어감
```

작업이 시작되면 프로젝트에 상태 파일이 생긴다:

```
.claude/state/
├── tasks.json     # 작업 원장 (무엇이 남았나) — JSON
└── progress.md    # 진행 로그 (무엇을 했나) — 사람이 읽는 Markdown
```

## 자주 묻는 것

**Q. 설치만 하면 자동으로 동작하나?**
반자동. skill은 상황이 description과 매칭될 때 로드되는 지침이라,
트리거를 놓칠 수 있다. `/context-init`으로 포인터를 심으면 매 세션
상기되므로 실질적으로 자동에 가까워진다.

**Q. `/context-init`을 안 하면?**
loop-engineering이 첫 트리거 때 같은 포인터를 스스로 심는다.
`/context-init`은 그 시점을 "장기 작업 첫 요청 때"에서 "지금"으로
앞당기는 것.

**Q. 확실하게 강제하고 싶다면?**
skill은 지침이라 모델이 건너뛸 수 있다. hook으로 "세션 종료 시
progress.md 갱신 확인" 같은 검사를 추가할 수 있다 — 규율이 자주 새는
경우에만 도입 권장.

## License

MIT
