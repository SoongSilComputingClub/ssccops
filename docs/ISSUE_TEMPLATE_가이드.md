# 이슈 템플릿 · 칸반 보드 운영 가이드

> **작성일**: 2026-08-03
> **목적**: Jira 스타일 칸반 관리를 GitHub Projects v2로 구현하기 위한 이슈 체계와 보드 구성
> **전제**: GitHub Issues + Projects v2 (Sub-issues · Issue Types 사용)

---

## 1. 먼저 알아야 할 것 — GitHub의 구조

Jira에서 옮겨올 때 가장 헷갈리는 지점이다. **정보가 두 군데에 나뉘어 있다.**

```
┌─ 이슈 (Repository) ──────────┐   ┌─ Projects v2 (보드) ─────────┐
│                              │   │                              │
│  제목 · 본문                  │   │  Status   (칸반 컬럼)         │
│  라벨                         │──▶│  Priority (우선순위)          │
│  Assignee                    │   │  Size     (작업 크기)         │
│  Milestone                   │   │  Iteration (스프린트)         │
│  Issue Type (Epic/Task/…)    │   │  담당 영역                    │
│  Sub-issues (부모-자식)       │   │  … 커스텀 필드                │
└──────────────────────────────┘   └──────────────────────────────┘
      템플릿으로 채우는 곳              보드에서 채우는 곳
```

**핵심 제약**: 이슈 템플릿은 **Projects 커스텀 필드를 채울 수 없다.**
템플릿으로 Status·Priority를 지정하는 기능은 GitHub에 없다 ([community discussion #13382](https://github.com/orgs/community/discussions/13382)).

그래서 설계 원칙이 이렇게 된다:

| 정보 | 어디에 두나 | 이유 |
|---|---|---|
| 요구사항 ID · AC · 재현 절차 | **이슈 본문** (템플릿) | 변하지 않는 내용, 검색 대상 |
| Status · Priority · Size · Iteration | **Projects 필드** | 자주 바뀌고, 보드에서 드래그로 조작 |
| Epic-Story-Task 계층 | **Sub-issues** | 보드에서 계층 뷰로 표시됨 |

> 템플릿에 "우선순위" 드롭다운을 넣으면 **본문 텍스트로만 남고 보드 필터에 안 걸린다.**
> 이전 버전 템플릿에서 이 항목들을 뺀 이유다.

---

## 2. 이슈 계층 — Jira와 대응

RFP에 이미 Epic이 `EP-O1 ~ EP-O4`로 정의되어 있다. 이를 그대로 쓴다.

| Jira | GitHub | 템플릿 | 칸반에서 |
|---|---|---|---|
| Epic | Issue Type `Epic` + Sub-issues | `01-epic.yml` | 보드에 안 올림 (진행률만 확인) |
| Story | Issue Type `Feature` | `02-story.yml` | **기본 카드** |
| Sub-task | Issue Type `Task` | `03-task.yml` | 필요할 때만 |
| Bug | Issue Type `Bug` | `04-bug.yml` | 기본 카드 |

```
🗂️ [EP-O2] 상태·승인 통제                    ← Epic (보드에 안 올림)
   │
   ├── 📌 [OPS-010] 업무 상태 전이 관리        ← Story (칸반 카드)
   │      ├── 🔧 DB: 상태 이력 테이블          ← Task (선택)
   │      ├── 🔧 API: 전이 검증 로직
   │      └── 🔧 UI: 상태 변경 화면
   │
   └── 📌 [OPS-014] 승인자 지정                ← Story (칸반 카드)
```

**Task는 필요할 때만 만든다.** Story가 작으면 Task 없이 바로 진행한다.
2인 개발이라 카드가 많아지면 오히려 관리가 어려워진다.

---

## 3. 템플릿 6종

| 파일 | 이름 | Issue Type | 언제 쓰나 |
|---|---|---|---|
| `01-epic.yml` | 🗂️ Epic | Epic | RFP의 Epic 단위. 프로젝트당 10개 내외 |
| `02-story.yml` | 📌 Story | Feature | **가장 많이 쓴다.** 요구사항 ID 1개 = Story 1개 |
| `03-task.yml` | 🔧 Task | Task | Story가 여러 계층에 걸칠 때만 |
| `04-bug.yml` | 🐞 Bug | Bug | 결함. 운영진도 사용 |
| `05-chore.yml` | ⚙️ Chore | Task | 인프라·문서·설정 — 기능이 아닌 모든 것 |
| `06-decision.yml` | ❓ 결정 필요 | — | 운영진 확정이 필요한 사항 |

### 왜 6종인가

이전 설계(9종)는 설계 단계와 개발 단계를 나눴는데, **칸반에서는 카드 종류가 적을수록 좋다.**
인터뷰·요구사항정의·설계 작업은 전부 Story 또는 Chore로 수렴한다.

| 통합한 것 | 어디로 |
|---|---|
| 인터뷰 진행 | `05-chore` (종류: 인터뷰·자료 수집) |
| 요구사항 정의 | `02-story` — 정의와 구현이 같은 카드에서 진행 |
| 설계 작업 | `05-chore` (종류: 문서) 또는 Story의 Task |
| 테스트·UAT | `05-chore` 또는 Bug — 별도 카드 종류가 불필요 |

---

## 4. Projects 보드 구성

### 4.1 보드 만들기

조직 또는 저장소 → **Projects** → **New project** → **Board** 템플릿

이름: `SSCC 운영시스템`

### 4.2 필수 커스텀 필드

보드 우측 `⚙️ Settings` → `+ New field`

| 필드명 | 타입 | 옵션 | 용도 |
|---|---|---|---|
| **Status** | Single select | `Backlog` `Todo` `In Progress` `In Review` `Blocked` `Done` | 칸반 컬럼 |
| **Priority** | Single select | `P0` `P1` `P2` `P3` | MoSCoW |
| **Size** | Single select | `XS` `S` `M` `L` `XL` | 작업 크기 (Story Point 대용) |
| **Iteration** | Iteration | 1주 단위 | 스프린트 |
| **담당 영역** | Single select | `개발자A` `개발자B` `운영진` | Task 시트 분담 |
| **Epic** | Text | — | Epic ID (`EP-O2`) — Sub-issue 계층의 보조 |

> **Size를 숫자가 아닌 T셔츠 사이즈로 둔 이유**: 2인 개발에서 스토리 포인트 추정은 오버헤드다.
> XS(1일 이내) / S(1~2일) / M(3일) / L(1주) / XL(쪼개야 함) 정도로만 감을 잡는다.

### 4.3 칸반 컬럼 정의

| 컬럼 | 들어가는 조건 | 나가는 조건 |
|---|---|---|
| **Backlog** | 등록됨. 아직 착수 계획 없음 | 이번 Iteration에 포함되면 Todo로 |
| **Todo** | 이번 주에 할 것. 선행 이슈 해결됨 | 착수하면 In Progress |
| **In Progress** | 브랜치 생성 후 작업 중 | PR 올리면 In Review |
| **In Review** | PR 올라감. 리뷰 대기 | 승인·머지되면 Done |
| **Blocked** | 결정 대기 · 선행 작업 미완 | 막힌 게 풀리면 원래 컬럼으로 |
| **Done** | AC 전부 통과, 머지 완료 | — |

**WIP 제한**: `In Progress`는 **1인당 2개까지.** 넘어가면 새로 시작하지 말고 끝내는 걸 우선한다.

### 4.4 뷰 5개

보드 하단 `+ New view`

| 뷰 이름 | 레이아웃 | 설정 |
|---|---|---|
| **칸반** | Board | Group by `Status`. 기본 화면 |
| **이번 주** | Board | Filter `iteration:@current`. 스프린트 집중용 |
| **Epic별** | Table | Group by `Epic`. 진행률 확인 |
| **내 작업** | Board | Filter `assignee:@me` |
| **운영진 확인 대기** | Table | Filter `label:status:needs-decision`. **회의 때 이 화면만 띄운다** |

### 4.5 자동화 (Workflows)

보드 `⚙️ Settings` → `Workflows`에서 켠다. 수동 관리를 크게 줄여준다.

| 자동화 | 동작 | 켜는 이유 |
|---|---|---|
| **Item added to project** | Status를 `Backlog`로 | 신규 이슈가 컬럼 없이 뜨는 것 방지 |
| **Item reopened** | Status를 `Todo`로 | 재오픈된 이슈가 Done에 남는 것 방지 |
| **Pull request merged** | Status를 `Done`으로 | 머지 = 완료. 수동 이동 불필요 |
| **Item closed** | Status를 `Done`으로 | |
| **Auto-add to project** | 저장소 신규 이슈 자동 추가 | **가장 중요.** 보드에 안 올라간 이슈가 생기는 것 방지 |

> `Auto-add`를 켜면 이슈를 만들 때마다 보드에 수동으로 추가할 필요가 없다.
> Projects의 `Workflows` → `Auto-add to project` → 저장소 지정.

---

## 5. 이슈 등록 절차

템플릿이 채우지 못하는 필드가 있으므로, **등록 직후 보드에서 채우는 습관**이 필요하다.

```
1. New issue → 템플릿 선택 → 본문 작성 → Create
                                    ↓
2. 이슈 화면 우측에서:
   - Labels    → domain:OPS, P0:must
   - Assignees → 담당자
   - Milestone → M4. 개발 (MVP)
   - Type      → Feature (Story인 경우)
   - Parent issue → 소속 Epic 연결        ← Sub-issue 계층
                                    ↓
3. 보드로 이동 (Auto-add로 자동 추가됨):
   - Status    → Backlog 또는 Todo
   - Priority  → P0
   - Size      → M
   - Iteration → 이번 주
   - 담당 영역  → 개발자A
```

**2·3단계를 빠뜨리면 카드가 보드에서 안 보이거나 필터에 안 걸린다.**

---

## 6. 라벨 vs Projects 필드 — 무엇을 어디에

둘 다 분류 수단이라 헷갈린다. 기준은 **"검색·자동화에 쓰는가"** 이다.

| | 라벨 | Projects 필드 |
|---|---|---|
| 보이는 곳 | 이슈 목록·검색 | 보드에서만 |
| 검색 | `label:domain:OPS` 가능 | 이슈 검색 불가 |
| 자동화 | GitHub Actions 조건으로 사용 | 보드 워크플로에서만 |
| 변경 빈도 | 거의 안 바뀜 | 자주 바뀜 |

**정리**

| 항목 | 위치 | 이유 |
|---|---|---|
| `type:*` | 라벨 | Issue Type과 중복이지만, 검색·색상 구분에 유용 |
| `domain:*` | 라벨 | 요구사항 ID 접두어와 1:1. 검색에 씀 |
| `P0~P3` | **라벨 + Projects Priority 둘 다** | 라벨은 검색용, 필드는 보드 정렬용 |
| `status:needs-decision` | 라벨 | 운영진 확인 대기 뷰 필터 |
| Status(칸반 컬럼) | Projects | 드래그로 바꾸는 값 |
| Size · Iteration | Projects | 보드 전용 |

> P0~P3만 양쪽에 둔다. 중복이지만 **이슈 목록에서도 우선순위가 보여야** 하기 때문이다.
> 나머지는 한쪽에만 둬서 관리 부담을 줄인다.

---

## 7. 작업 흐름 (이슈 → PR)

```
📌 #87 [OPS-010] 업무 상태 전이 관리
    Status: Todo → In Progress          (보드에서 드래그)
        ↓
브랜치  feat/OPS-010-status-transition
        ↓
커밋    feat(OPS): 상태 전이 검증 로직 추가 (OPS-010)
        ↓
PR      [OPS-010] 업무 상태 전이 관리
        본문: Closes #87
    Status: In Progress → In Review     (자동 아님, 수동)
        ↓
머지    Status → Done                    (자동화로 처리)
        이슈 자동 종료, AC 체크박스가 완료 근거로 남음
```

**브랜치 접두어**: `feat/` `fix/` `docs/` `chore/` `test/`

---

## 8. 운영 규칙

### 8.1 카드를 만들 때

1. **요구사항 ID를 제목 맨 앞에** — `[OPS-010]`. ID 없으면 `[CHORE]` `[BUG]`
2. **Story의 AC가 10개를 넘으면 쪼갠다** — 한 PR로 닫을 수 없는 크기
3. **Epic은 보드에 올리지 않는다** — 진행률만 Epic별 뷰에서 확인
4. **M4 이후 새 기능 요청은 Story부터** — 바로 Task로 만들지 않는다 (범위 통제)

### 8.2 카드를 옮길 때

- `In Progress`는 **1인당 2개까지** — 넘으면 끝내는 걸 우선
- 막히면 **Blocked로 옮기고 이유를 코멘트로** — 조용히 멈춰 있는 카드가 가장 위험
- `In Review`에 3일 이상 있으면 리뷰 재촉

### 8.3 카드를 닫을 때

| 종류 | 닫는 조건 |
|---|---|
| Story | AC 체크박스 **전부** 체크 |
| Bug | 재현 절차대로 다시 해서 발생하지 않음 |
| Chore | 문서화 위치에 실제로 기록됨 |
| Epic | 하위 이슈 전부 종료 + Epic 완료 조건 충족 |

**닫지 말아야 할 때**: "일단 되니까" (AC 미달) · "나중에 문서 쓸게" (문서가 완료 조건) · 결함을 원인 파악 없이 우회만 함

---

## 9. 적용 순서

- [ ] 1. 템플릿 6종을 `.github/ISSUE_TEMPLATE/` 에 커밋
- [ ] 2. 보드 생성 후 `config.yml` 의 칸반 보드 URL에 프로젝트 번호 추가
      (`/projects` → `/projects/1`)
- [ ] 3. 조직 설정에서 **Issue Types** 확인 — `Epic` `Feature` `Task` `Bug`
      (조직 저장소만 사용 가능. 개인 저장소면 라벨 `type:*` 로 대체)
- [ ] 4. 라벨 생성 — `scripts/setup-github.sh`
- [ ] 5. Projects 보드 생성 + 커스텀 필드 6개 (4.2절)
- [ ] 6. 뷰 5개 생성 (4.4절)
- [ ] 7. 자동화 5개 켜기 (4.5절) — 특히 `Auto-add to project`
- [ ] 8. Epic 이슈부터 등록 (RFP의 `EP-*`)
- [ ] 9. Story를 Epic의 Sub-issue로 연결

---

## 부록 A. Jira 기능 대응표

| Jira | GitHub 대응 | 비고 |
|---|---|---|
| Epic | Issue Type `Epic` + Sub-issues | 최대 8단계 중첩, 부모당 100개 |
| Story Point | Projects `Size` 필드 | 숫자 필드로 만들면 합계 표시 가능 |
| Sprint | Projects `Iteration` 필드 | 기간·휴식 설정 가능 |
| Board | Projects Board 뷰 | |
| Backlog | `Status: Backlog` 컬럼 | |
| Workflow | Projects `Workflows` | Jira보다 단순 |
| Component | 라벨 `domain:*` | |
| Burndown | Projects `Insights` | 기본 차트 제공 |
| Roadmap | Projects `Roadmap` 뷰 | Iteration·Date 필드 기준 |
| **Swimlane** | ❌ 없음 | `Group by` 로 부분 대체 |
| **필수 필드 강제** | ❌ 보드 필드는 불가 | 템플릿 본문 항목만 강제 가능 |
| **워크플로 전이 규칙** | ❌ 없음 | 아무 컬럼으로나 이동 가능 |

## 부록 B. 만들지 않은 템플릿

| 유형 | 제외 이유 |
|---|---|
| 리팩터링 | `05-chore` 로 충분 |
| 스파이크(기술 조사) | `06-decision` 이 대체 — 조사 결과가 결국 결정으로 이어짐 |
| 릴리스 | 마일스톤이 그 역할 |
| 보안 취약점 | Private 저장소이므로 Bug(Critical)로 처리 |
| 테스트·UAT | `05-chore` 또는 Bug — 별도 카드 종류가 불필요 |

---

**출처**
- [Issue templates: specify a project — community discussion #13382](https://github.com/orgs/community/discussions/13382)
- [Sub-issues Public Preview — community discussion #148714](https://github.com/orgs/community/discussions/148714)
- [Configuring issue templates — GitHub Docs](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository)
- [About Projects — GitHub Docs](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects)
