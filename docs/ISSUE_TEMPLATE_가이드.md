# 이슈 템플릿 · 칸반 보드 운영 가이드

> **작성일**: 2026-08-03 (개정: 2026-08-07 — 레포 역할 재정의)
> **목적**: Jira 스타일 칸반 관리를 GitHub Projects v2로 구현하기 위한 이슈 체계와 보드 구성
> **전제**: `ssccops`는 문서·이슈 전용 최상위 관리 레포. 코드는 개발 레포(`ssccops-server` 등)에.
> GitHub Issues + Projects v2 (Sub-issues · Issue Types 사용)

---

## 0. 계층부터 — Epic → Story/Task/Bug → Sub-task

`Epic`·`Story`·`Task`·`Bug`·`Subtask` 다섯 개 전부 조직에 등록한 **GitHub Issue Type**이다
(GitHub이 기본 제공하는 건 `Bug`/`Feature`/`Task`뿐이라, `Epic`·`Story`·`Subtask`는 커스텀으로
추가했다). 계층은 Jira와 동일하게 4단으로 잡는다 — **Sub-task는 항상 Story/Task/Bug의
자식**이고, Epic의 직계 자식이 되거나 스스로 자식을 가질 수 없다.

**Sub-task를 어느 레포에 만드는지는 "코드냐 아니냐"로 가른다** — `ssccops`냐 개발 레포냐가
기준이 아니다.

```
코드가 아닌 경우 — 전부 [ssccops] 안에서 완결
🗂️ #10 [EPIC] 2차 인터뷰
   └── 🔧 #11 [TASK] 인터뷰 진행
          ├── 🔹 #12 [SUB] 총무 인터뷰       ──Parent──▶ #11
          ├── 🔹 #13 [SUB] 학술국장 인터뷰    ──Parent──▶ #11
          └── 🔹 #14 [SUB] 회장 인터뷰       ──Parent──▶ #11

코드인 경우 — [ssccops]의 Story가 [ssccops-server]의 Sub-task를 cross-repo로 거느림
[ssccops]                              [ssccops-server]
🗂️ #20 [EPIC] 상태·승인 통제
   └── 📌 #21 [OPS] 업무 상태 전이 관리 ──Parent──▶ 🔹 #101 [SUB] DB: 상태 이력 테이블
                                       ──Parent──▶ 🔹 #102 [SUB] API: 전이 검증 로직
                                       ──Parent──▶ 🔹 #103 [SUB] UI: 상태 변경 화면
```

번호(`#87`, `#101` 등)는 사람이 매기지 않는다 — **이슈를 만드는 순간 GitHub이 자동으로 부여**한다.
`OPS`는 도메인 라벨과 맞춘 제목 태그일 뿐, 추적용 식별자가 아니다. 자세한 이유는 `GITHUB_PROJECTS_설계안.md` 6절 참조.

GitHub Sub-issues는 다른 레포의 이슈도 자식으로 연결할 수 있어(cross-repo), 코드 Sub-task를
개발 레포에 둬도 `ssccops`의 Story에서 진행률이 그대로 집계된다. `ssccops`가 여러 개발
레포(백엔드·프론트엔드·추후 인프라 등)를 아우르는 상위 추적 지점 역할을 하는 이유다.
GitHub 조직 플랜에 따라 cross-repo Sub-issues 가능 여부가 다를 수 있으니 저장소 생성 후 확인한다.

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
| Epic-Story/Task/Bug-Sub-task 계층 | **Sub-issues** (레포 내부 · cross-repo 둘 다) | 보드에서 계층 뷰로 표시됨 |

> 템플릿에 "우선순위" 드롭다운을 넣으면 **본문 텍스트로만 남고 보드 필터에 안 걸린다.**
> 이전 버전 템플릿에서 이 항목들을 뺀 이유다.

---

## 2. 이슈 계층 — Jira와 대응

RFP에 이미 Epic이 `EP-O1 ~ EP-O4`로 정의되어 있다. 이를 그대로 쓴다.

| Jira | GitHub Issue Type | 만드는 레포 | 템플릿 | 칸반에서 |
|---|---|---|---|---|
| Epic | `Epic` | `ssccops`만 | `1-epic.yml` | 보드에 안 올림 (진행률만 확인) |
| Story | `Story` | `ssccops`만 | `2-story.yml` | **기본 카드** |
| Task | `Task` | `ssccops`만 | `2-task.yml` | 필요할 때만 |
| Bug | `Bug` | `ssccops`만 | `2-bug.yml` | 기본 카드 |
| **Sub-task** | `Subtask` | **코드면 개발 레포, 아니면 `ssccops`** | `3-subtask.yml` (`ssccops`) / 개발 레포 자체 템플릿 | 조직 보드 (하나로 통합) |

**Task는 Sub-task가 아니다.** `ssccops`의 Task는 Story에 속하지 않는 독립 작업
(인프라·문서·설정·인터뷰 등 — 이전 버전의 `Chore`)을 가리킨다. **Sub-task는 Task와 별개의
Issue Type**이며 Story·Task·Bug 밑에서만 존재한다 — Task를 더 쪼개고 싶다고 또 다른 Task를
자식으로 붙이지 않는다(계층이 무너진다). 코드 구현이면 개발 레포에, 코드가 아니면 `ssccops`
안에 Sub-task를 만들어 Parent를 지정한다.

---

## 3. 템플릿 5종

| 파일 | 이름 | Issue Type | 언제 쓰나 |
|---|---|---|---|
| `1-epic.yml` | 🗂️ Epic | Epic | RFP의 Epic 단위. 프로젝트당 10개 내외 |
| `2-story.yml` | 📌 Story | Story | **가장 많이 쓴다.** 요구사항 ID 1개 = Story 1개 |
| `2-task.yml` | 🔧 Task | Task | Story에 속하지 않는 독립 작업 (인프라·문서·설정·인터뷰·**결정 필요**) |
| `3-subtask.yml` | 🔹 Sub-task | Subtask | Story/Task/Bug를 쪼갠 가장 작은 실행 단위. **코드가 아닐 때만** 여기서 생성 |
| `2-bug.yml` | 🐞 Bug | Bug | 결함. 운영진도 사용 |

**파일명 숫자는 등록 순서가 아니라 계층 깊이다** — Epic이 1단(`1-`), 그 자식인 Story·Task·Bug가
2단(`2-`), 다시 그 자식인 Sub-task가 3단(`3-`). `2-`를 공유하는 세 파일의 표시 순서는 파일 이름
알파벳순(`2-bug` → `2-story` → `2-task`)으로 정해지고 기능에는 영향이 없다.

### 왜 5종인가 (6종 → 5종 → 6종 → 5종, 두 번 걷어냈다)

**Sub-task**(0절)와 **Decision**(3.1절) 둘 다 한때 별도 파일이 있었다가 없어졌다 — 이유는 서로 다르다.

- Sub-task는 "GitHub에 맞는 Issue Type이 없어서" 임시로 없앴다가, 조직에 `Subtask` 타입을
  커스텀으로 등록하면서 **`3-subtask.yml`로 부활**했다. 없앤 게 실수였고 되살렸다.
- Decision(`06-decision.yml`)은 애초에 별도 Issue Type을 만들지 않고 `Task`를 그대로 썼는데,
  폼만 따로 두고 있었다. 폼을 분리해 둘 이유가 딱히 없어서 **`2-task.yml`에 완전히 합쳤다**
  (아래 3.1절). 이번엔 되살리지 않는다 — 계층 문제가 아니라 단순 정리다.

지금은 GitHub Issue Type 5개(`Epic`·`Story`·`Task`·`Subtask`·`Bug`)에 템플릿 파일 5개가
정확히 1:1로 대응한다. "결정 필요"는 `Task`의 하위 개념(작업 종류 드롭다운 옵션)으로 존재한다.

### 3.1 "결정 필요"는 별도 파일이 아니다 — `2-task.yml`에 흡수

Jira 기본 이슈 타입은 Epic·Story·Task·Bug·Sub-task뿐이고 "Decision"은 없다.
실무에서 "결정이 필요한 사항"을 다루는 방식은 대략 셋으로 갈린다.

| 방식 | 실무 사례 | 이 프로젝트에 맞는가 |
|---|---|---|
| RFC/ADR 선(先) PR | 이슈 없이 바로 ADR 문서에 PR을 올려 리뷰로 논의 (Rust RFC 등) | 기한 강제·보드 가시성이 없어 약함 |
| Spike (Scrum/XP) | Atlassian 공식 가이드도 "별도 타입이 아니라 Story/Task + `spike` 라벨"로 구현하라고 명시 | Spike는 *기술 조사*에 초점 — 운영진 의사결정엔 결이 다름 |
| **Task + 목적 표시** | 별도 타입을 안 만들고 Task 안에서 "결정 필요"임을 표시하되, 구조화된 입력은 선택 필드로 제공 | **채택** — Jira 4종을 유지하면서 보드 뷰·기한 강제 UX는 살릴 수 있음 |

`2-task.yml`은 **Issue Type을 `Task`로 지정**하고, **라벨은 아예 쓰지 않는다** — 이 레포는
`domain:*` 말고는 라벨을 쓰지 않기로 했다(6절 참조). "결정 필요" 작업은:

1. "작업 종류" 드롭다운에서 **결정 필요**를 선택하고
2. 제목 앞에 **`[DECISION]`**을 붙인다 — "운영진 확인 대기" 뷰가 이 접두어로 필터링한다
3. 선택지·권고안·결정 주체·결정 기한·영향·결정 결과 필드(**전부 선택 입력**)를 채운다

**필수 강제를 일부 포기한 대가다.** 예전 `06-decision.yml`은 선택지·기한·영향을 필수로
강제했지만, 한 템플릿에 여러 작업 종류를 같이 담으면 GitHub Issue Forms는 조건부 필수
필드(드롭다운 값에 따라 다른 필드를 강제)를 지원하지 않는다. 그래서 이 필드들은 전부
선택 입력으로 두고, 마크다운 안내와 필드 설명("결정 필요 항목만")으로 유도하는 쪽을 택했다.
강제력은 약해졌지만 템플릿 파일 하나로 관리 부담이 준다.

| 이전 | 지금 |
|---|---|
| `2-task.yml` (Story 계층별 실작업, 6종→5종 시절) | `3-subtask.yml`로 부활 — 단 Issue Type을 `Subtask`로 정확히 구분 |
| `05-chore.yml` (인프라·문서) | `2-task.yml`(Task)로 흡수 |
| `06-decision.yml` (별도 파일, Issue Type `Task`) | **삭제** — `2-task.yml`의 "작업 종류: 결정 필요" + 선택 필드로 흡수 |
| `type:*` 라벨 5종 | **전부 폐지** — Epic/Story/Task/Bug/Subtask가 실제 Issue Type이라 중복 |
| `P0:must`~`P3:could`, `wont`, `status:*` 라벨 8종 | **전부 폐지** — Projects 필드로 대체 (4절) |

인터뷰·요구사항정의·설계·결정 필요는 전부 Story·Task·Sub-task로 수렴한다.

| 작업 | 어디로 |
|---|---|
| 인터뷰 진행 전체를 묶는 단위 | `2-task` (종류: 인터뷰·자료 수집) |
| 인터뷰 개별 건 | `3-subtask` — 위 Task를 Parent로 지정 |
| 요구사항 정의 | `2-story` — 정의와 구현이 같은 카드에서 진행 |
| 설계 작업 | `2-task` (종류: 문서) |
| 운영진·개발팀 확정 필요 | `2-task` (종류: 결정 필요, 제목 `[DECISION]`) |
| 테스트·UAT | `2-task` 또는 Bug — 별도 카드 종류가 불필요 |
| 실제 코드 구현 | **개발 레포 Sub-task** — `ssccops`에는 만들지 않음 |

---

## 4. Projects 보드 구성

### 4.1 보드 만들기

**조직 수준**(레포 수준이 아님) → **Projects** → **New project** → **Board** 템플릿

이름: `SSCC 운영시스템`

조직 수준으로 만드는 이유: `ssccops`와 개발 레포(`ssccops-server` 등)의 이슈를
**하나의 보드**에 모아야 레포가 나뉘어도 전체 진행 상황이 한눈에 보인다.

### 4.2 필수 커스텀 필드

보드 우측 `⚙️ Settings` → `+ New field`

| 필드명 | 타입 | 옵션 | 용도 |
|---|---|---|---|
| **Status** | Single select | `Backlog` `Todo` `In Progress` `In Review` `Blocked` `Done` | 칸반 컬럼 |
| **Priority** | Single select | `P0` `P1` `P2` `P3` `Won't` | MoSCoW (`Won't` = 이번 범위 제외, 이전 `wont` 라벨 대체) |
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
| **In Progress** | 개발 레포에 Sub-issue 생성 후 작업 중 | PR 올리면 In Review |
| **In Review** | 개발 레포 PR 올라감. 리뷰 대기 | 승인·머지되면 Done |
| **Blocked** | 결정 대기 · 선행 작업 미완 | 막힌 게 풀리면 원래 컬럼으로 |
| **Done** | AC 전부 통과, 관련 Sub-issue 전부 머지 완료 | — |

**WIP 제한**: `In Progress`는 **1인당 2개까지.** 넘어가면 새로 시작하지 말고 끝내는 걸 우선한다.

### 4.4 뷰 5개

보드 하단 `+ New view`

| 뷰 이름 | 레이아웃 | 설정 |
|---|---|---|
| **칸반** | Board | Group by `Status`. 기본 화면 |
| **이번 주** | Board | Filter `iteration:@current`. 스프린트 집중용 |
| **Epic별** | Table | Group by `Epic`. 진행률 확인 |
| **내 작업** | Board | Filter `assignee:@me` |
| **운영진 확인 대기** | Table | 텍스트 필터 `[DECISION]` (제목 검색). **회의 때 이 화면만 띄운다** |

### 4.5 자동화 (Workflows)

보드 `⚙️ Settings` → `Workflows`에서 켠다. 수동 관리를 크게 줄여준다.

| 자동화 | 동작 | 켜는 이유 |
|---|---|---|
| **Item added to project** | Status를 `Backlog`로 | 신규 이슈가 컬럼 없이 뜨는 것 방지 |
| **Item reopened** | Status를 `Todo`로 | 재오픈된 이슈가 Done에 남는 것 방지 |
| **Pull request merged** | Status를 `Done`으로 | 머지 = 완료. 수동 이동 불필요 |
| **Item closed** | Status를 `Done`으로 | |
| **Auto-add to project** | 저장소 신규 이슈 자동 추가 | **가장 중요.** `ssccops`뿐 아니라 **개발 레포마다 각각** 등록해야 그 레포의 Sub-issue도 보드에 뜬다 |

> `Auto-add`는 레포별로 따로 켠다. Projects의 `Workflows` → `Auto-add to project` →
> `ssccops` 추가 → `ssccops-server` 추가 → (개발 레포가 늘 때마다 반복).
> 하나라도 빠뜨리면 그 레포의 Sub-issue가 보드에 안 보인다.

---

## 5. 이슈 등록 절차

템플릿이 채우지 못하는 필드가 있으므로, **등록 직후 보드에서 채우는 습관**이 필요하다.
Story·Task를 더 쪼개야 하면 Sub-task를 만드는 절차가 하나 더 붙는다.

```
1. [ssccops] New issue → 템플릿 선택 → 본문 작성 → Create
                                    ↓
2. 이슈 화면 우측에서:
   - Labels    → domain:OPS
   - Assignees → 담당자
   - Milestone → M4. 개발 (MVP)
   - Type      → Story (Story인 경우)
   - Parent issue → 소속 Epic 연결        ← Sub-issue 계층
                                    ↓
3. 보드로 이동 (Auto-add로 자동 추가됨):
   - Status    → Backlog 또는 Todo
   - Priority  → P0
   - Size      → M
   - Iteration → 이번 주
   - 담당 영역  → 개발자A
                                    ↓
4. 더 쪼개야 하면 Sub-task 생성, 어디에 만들지는 코드 여부로 결정:
   - 코드 구현 → [ssccops-server] 에 Sub-task 생성, Parent를 [ssccops]의 Story/Task로 지정 (cross-repo)
   - 코드가 아님 → [ssccops] 안에 3-subtask.yml로 생성, Parent를 같은 레포의 Story/Task로 지정
```

**2·3단계를 빠뜨리면 카드가 보드에서 안 보이거나 필터에 안 걸린다.**
**4단계를 빠뜨리면 `ssccops`의 Story/Task가 "구현 안 됨" 상태로 방치된다.**

---

## 6. 라벨 vs Projects 필드 — 무엇을 어디에

둘 다 분류 수단이라 헷갈린다. 기준은 **"검색·자동화에 쓰는가"** 이다.

| | 라벨 | Projects 필드 |
|---|---|---|
| 보이는 곳 | 이슈 목록·검색 | 보드에서, 그리고 프로젝트에 추가된 이슈라면 이슈 사이드바에서도 |
| 검색 | `label:domain:OPS` 가능 | 이슈 목록 검색 불가 (보드 필터만 가능) |
| 자동화 | GitHub Actions 조건으로 사용 | 보드 워크플로에서만 |
| 변경 빈도 | 거의 안 바뀜 | 자주 바뀜 |

**이 레포는 라벨을 `domain:*` 하나로만 쓴다.** 예전엔 type·priority·status까지 라벨과 필드에
중복으로 뒀는데, 지금은 전부 한쪽으로 정리했다.

| 항목 | 위치 | 이유 |
|---|---|---|
| Epic/Story/Task/Bug/Subtask | **Issue Type** (라벨 아님) | 뱃지·검색(`type:Bug`)이 이미 되므로 라벨을 따로 안 둔다 |
| `domain:*` | **라벨만** | Projects 필드는 이슈 목록 검색에 안 걸린다. 요구사항 ID 접두어와 1:1로 검색에 실제로 쓰인다 |
| Priority (`P0`~`P3`, `Won't`) | **Projects 필드만** | 예전엔 라벨과 필드 양쪽에 뒀지만, 라벨을 domain 하나로 좁히면서 필드로 일원화했다 |
| Status(칸반 컬럼, `Blocked`/`In Review` 포함) | **Projects 필드만** | 드래그로 바꾸는 값 |
| "결정 필요" | **라벨도 필드도 아님 — 제목 `[DECISION]` 접두어** | Status 필드엔 대응 옵션이 없고, 라벨은 안 쓰기로 했다. 유일한 예외 처리 |
| Size · Iteration | Projects 필드만 | 보드 전용 |

> **Priority가 예전엔 라벨+필드 둘 다였다.** "이슈 목록에서도 우선순위가 보여야 한다"는 이유였는데,
> Priority 필드는 프로젝트에 추가된 이슈의 사이드바에 직접 표시되므로(Auto-add로 항상 추가되니
> 사실상 전부) 굳이 라벨로 중복 관리할 필요가 없다고 판단해 필드로 일원화했다.

---

## 7. 작업 흐름 (이슈 → Sub-task → PR)

**코드인 경우**

```
[ssccops]
📌 #87 [OPS] 업무 상태 전이 관리
    Status: Todo → In Progress          (보드에서 드래그)
        ↓
[ssccops-server]
🔹 #42 [SUB] 상태 전이 API                Parent: ssccops#87 (cross-repo Sub-issue)
        ↓
브랜치  feat/#42-status-transition       (ssccops-server가 이슈 생성 시 자동 생성 — CONTRIBUTING.md)
        ↓
커밋    #42 feat(ops): 상태 전이 검증 로직 추가
        ↓
PR      [#42] 상태 전이 API 구현
        본문: Closes #42
    ↓
머지    Squash and merge (커밋 제목 = PR 제목 그대로) → ssccops-server#42 종료
        ↓
[ssccops]
📌 #87  Sub-task #42 종료 확인 → Status: In Review → Done   (수동, 자동화 아님)
        AC 체크박스 체크 = 완료 근거
```

**코드가 아닌 경우** — 전부 `ssccops` 안에서 끝난다. 브랜치·PR이 없다.

```
[ssccops]
🔧 #11 [TASK] 인터뷰 진행
    Status: Todo → In Progress
        ↓
🔹 #12 [SUB] 총무 인터뷰                 Parent: ssccops#11 (같은 레포 Sub-issue)
    작업 내용 체크 → 이슈 Close
        ↓
🔧 #11  Sub-task 8건 전부 종료 확인 → Status: Done
```

**브랜치 접두어**(`ssccops-server` 기준, `feat`/`fix`/`refactor` 세 가지만 허용 — 이슈 제목
태그로 자동 생성됨): `feat/` `fix/` `refactor/`. 커밋·PR·머지 규칙 전체는 개발 레포의
`CONTRIBUTING.md`에 있다 — `ssccops`는 이를 강제하지 않고 참고만 한다
(`GITHUB_PROJECTS_설계안.md` 7.2절 참조).

---

## 8. 운영 규칙

### 8.1 카드를 만들 때

1. **타입/도메인 태그를 제목 맨 앞에** — `[OPS]`, `[EPIC]`, `[TASK]`, `[SUB]`, `[BUG]`, `[DECISION]`. 순번은 매기지 않는다 — 고유 식별자는 GitHub 이슈 번호(#)
2. **Story의 AC가 10개를 넘으면 쪼갠다** — 한 Sub-task로 닫을 수 없는 크기
3. **Epic은 보드에 올리지 않는다** — 진행률만 Epic별 뷰에서 확인
4. **M4 이후 새 기능 요청은 Story부터** — 바로 Sub-task를 만들지 않는다 (범위 통제)
5. **코드는 반드시 개발 레포에** — `ssccops`에 코드·설정 파일을 올리지 않는다. 코드가 아닌
   Sub-task(인터뷰·자료 취합 등)는 `ssccops`에 만들어도 된다 (0절)

### 8.2 카드를 옮길 때

- `In Progress`는 **1인당 2개까지** — 넘으면 끝내는 걸 우선
- 막히면 **Blocked로 옮기고 이유를 코멘트로** — 조용히 멈춰 있는 카드가 가장 위험
- `In Review`에 3일 이상 있으면 리뷰 재촉

### 8.3 카드를 닫을 때

| 종류 | 닫는 조건 |
|---|---|
| Story | AC 체크박스 **전부** 체크 + 하위 Sub-task **전부** 종료 (레포 무관) |
| Task | 작업 내용 체크박스 전부 체크 (하위 Sub-task가 있으면 그것도 전부 종료) |
| Sub-task | 작업 내용 체크박스 전부 체크 |
| Bug | 재현 절차대로 다시 해서 발생하지 않음 |
| Epic | 하위 이슈 전부 종료 + Epic 완료 조건 충족 |

**닫지 말아야 할 때**: "일단 되니까" (AC 미달) · "나중에 문서 쓸게" (문서가 완료 조건) · 결함을 원인 파악 없이 우회만 함 · 하위 Sub-task가 아직 열려 있는데 상위 Story/Task만 닫음

---

## 9. 적용 순서

- [ ] 1. 템플릿 5종을 `.github/ISSUE_TEMPLATE/` 에 커밋
- [ ] 2. 보드 생성 후 `config.yml` 의 칸반 보드 URL에 프로젝트 번호 추가
      (`/projects` → `/projects/1`)
- [ ] 3. 조직 설정에서 **Issue Types** 확인 — `Epic` `Story` `Task` `Subtask` `Bug` (전부 등록 완료)
      (조직 저장소만 사용 가능. 개인 저장소면 라벨 `type:*` 로 대체)
- [ ] 4. 조직 설정에서 **cross-repo Sub-issues** 가능 여부 확인
- [ ] 5. 라벨 생성 — `scripts/setup-github.sh`
- [ ] 6. Projects 보드 생성(조직 수준) + 커스텀 필드 6개 (4.2절)
- [ ] 7. 뷰 5개 생성 (4.4절)
- [ ] 8. 자동화 켜기 (4.5절) — 특히 `Auto-add to project`를 `ssccops`와 개발 레포 각각에
- [ ] 9. Epic 이슈부터 등록 (RFP의 `EP-*`)
- [ ] 10. Story를 Epic의 Sub-issue로 연결

---

## 부록 A. Jira 기능 대응표

| Jira | GitHub 대응 | 비고 |
|---|---|---|
| Epic | Issue Type `Epic` + Sub-issues | `ssccops`에서만 관리. 최대 8단계 중첩, 부모당 100개 |
| Story / Task / Bug | Issue Type `Story`/`Task`/`Bug` | `ssccops`에서만 생성 |
| **Sub-task** | Issue Type `Subtask` + Sub-issues (레포 내부 또는 cross-repo) | **코드면 개발 레포**(`ssccops-server` 등), **코드가 아니면 `ssccops`**. 어느 쪽이든 Story/Task/Bug를 Parent로 지정 |
| Story Point | Projects `Size` 필드 | 숫자 필드로 만들면 합계 표시 가능 |
| Sprint | Projects `Iteration` 필드 | 기간·휴식 설정 가능 |
| Board | Projects Board 뷰 | 조직 수준 — 모든 레포 통합 |
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
| 리팩터링 | `2-task` 로 충분 |
| Decision (별도 Issue Type이나 별도 파일) | Jira 표준 타입이 아님 — `2-task.yml`의 "작업 종류: 결정 필요"로 흡수, Issue Type은 `Task` (3.1절) |
| 스파이크(기술 조사) | 별도로 만들지 않음 — 필요하면 `2-task`의 "작업 종류"에 문서/조사로 등록. 기술 조사가 아닌 운영 의사결정은 같은 템플릿의 "결정 필요" 종류 사용 |
| 릴리스 | 마일스톤이 그 역할 |
| 보안 취약점 | Private 저장소이므로 Bug(Critical)로 처리 |
| 테스트·UAT | `2-task` 또는 Bug — 별도 카드 종류가 불필요 |

---

**출처**
- [Issue templates: specify a project — community discussion #13382](https://github.com/orgs/community/discussions/13382)
- [Sub-issues Public Preview — community discussion #148714](https://github.com/orgs/community/discussions/148714)
- [Configuring issue templates — GitHub Docs](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository)
- [About Projects — GitHub Docs](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects)
