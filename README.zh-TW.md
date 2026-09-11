# Agent Runs Template

[English](README.md) | [繁體中文](README.zh-TW.md)

一個可重複使用、與 Agent provider 無關的模板，用來保存 AI coding agent 的 durable execution state。

這個 repository 預期作為 **GitHub Template Repository** 使用。真正的執行資料應建立在由此模板產生的獨立 instance repo 中；如果內容可能包含內部工程資訊，instance repo 應依資料敏感度設為 private。

> Public template 規則：本 repo 內所有 example 都必須是 synthetic 且已去識別化。不可使用真實公司名、產品名、repository 名稱、Issue / PR 編號、內部 file path、customer data、credential 或其他內部識別資訊。

## 目標

提供 Claude Code、Codex、OpenCode 與未來其他 coding-agent runtime 一套共同的 state / logging contract，讓外部 orchestrator（例如 ChatGPT）可以回答：

- 現在正在做什麼？
- 哪些 execution streams 正在進行？
- 哪裡被 blocked？
- 哪些事情需要 human attention？
- 有什麼 evidence 可以證明真的完成？
- 新的 agent / session 能不能安全接手？

核心概念是把 durable engineering state 與高噪音 runtime detail 分開。

## 核心模型

```text
Project
  ↓
Work Item
  ↓
Stream
  ↓
Run
```

- **Work Item**：真正的 engineering goal，通常會連到 GitHub Issue 或其他 canonical tracker item。
- **Stream**：Work Item 裡的一條 execution lane，例如 `api`、`web`、`migration`、`infra`、`e2e`。
- **Run**：單次 agent runtime execution / session。Run 可以被替換，但 Work Item 與 Stream 應該是 durable 的。

`Run != Task`，`Session != Work Item`。

## 資訊分層

```text
Issue       = What / Why
State       = Where we are
Checkpoint  = What the next agent needs
Events      = What actually happened
PR + CI     = Evidence
```

### Control Plane

Canonical engineering Issue 應保存：

- requirement
- scope
- Acceptance Criteria
- 重要決策
- 需要人處理的 blocker
- PR milestone
- verified completion

Issue **不應該**變成 raw agent transcript 或每一次 tool call 的 activity log。

### State

State file 回答「現在在哪裡」。分三層：

```text
work-item/index.json
stream/state.json
run/state.json
```

Orchestrator 應該從最高層開始讀，只在需要時才往下 drill down。

### Checkpoint

Checkpoint 是 compact handoff artifact，目的是讓 fresh agent 或 compact 後的新 session 不需要重播整段 transcript，也能安全接手。

### Events

`events.jsonl` 保存 structured runtime history，用於 audit / debugging / retrospective。

它不是預設 human interface，也不應整份投影到 GitHub Issue。

## Repository 結構

```text
<org-or-scope>/
  <project>/
    work-items/
      <work-item-id>/
        index.json
        checkpoint.md
        streams/
          <stream>/
            state.json
            runs/
              <run-id>/
                state.json
                checkpoint.md
                events.jsonl

schema/
examples/
```

完整 synthetic example 請看：

```text
examples/acme/example-app/work-items/WORK-123/
```

## ChatGPT / Orchestrator 讀取策略

使用 progressive drill-down：

```text
Canonical Issue
    ↓
Work Item index.json
    ↓
Stream state.json
    ↓
Run state.json
    ↓
Checkpoint
    ↓
events.jsonl
```

原則是：**只讀回答問題所需要的最小層級。**

例如：

- 「現在做到哪？」→ Issue + `index.json`
- 「為什麼 E2E blocked？」→ 再讀 `streams/e2e/state.json`
- 「失敗前到底發生什麼？」→ 再讀 active run 與 `events.jsonl`

## Runtime Responsibility

Logging 應主要是 **runtime / hook / adapter responsibility**，而不是依賴 model memory。

Provider adapter 可以把 session start、tool use、stop、compact、session end 等 lifecycle event 轉成統一的 normalized state / event protocol。

Agent 不應被要求每個 tool call 後都額外產生自然語言摘要。

## Security / Privacy

- 永遠不要記錄 credential、secret value、auth token、private key、session cookie。
- Raw transcript 預設不應 push 到 durable storage。
- 能記 metadata 就不要記 sensitive payload。
- 必要時先 redact tool output 再保存。
- Public template repo 只能包含 **synthetic examples**。
- 真正 execution repo 的 visibility 應依資料敏感度決定。

安全 event 範例：

```json
{
  "event": "credential_used",
  "credential": "EXAMPLE_API_KEY",
  "value": "REDACTED"
}
```

## Completion Semantics

Agent 說「done」不能直接視為完成證據。

Completion 應該有對應 evidence，例如：

- automated tests
- CI checks
- read-back verification
- deployment verification
- measurable Acceptance Criteria

Acceptance Criteria 應各自有明確 verification state。

## Rollover / Repository Growth

當 execution repository 變得太大或操作成本太高時：

1. 保留舊 repo 作為 immutable historical evidence。
2. 從這個 template 建立新的 execution repo。
3. 沿用相同 schema / version contract。
4. 不要只是為了方便，就回頭重寫或壓縮舊歷史。

Template repo 本身只作為 reusable source，不保存 production execution logs。

## 初始 PoC 範圍

第一版可以只做：

- 一個 GitHub-backed execution repository
- 一種 coding-agent runtime
- runtime hooks / adapters
- Work Item / Stream / Run state
- checkpoint
- structured events
- orchestrator read-back

在 state model 還沒證明有價值以前，不急著做 dashboard、database、real-time streaming 或複雜 RBAC。

## Implementation Acceptance Criteria

- [ ] Work Item、Stream、Run schema 有文件且 machine-readable。
- [ ] 新 Run 可以接到既有 Stream，不會失去 durable state。
- [ ] 不需要讀完整 event log 就能知道 current state。
- [ ] Human-required blocker 與 agent-actionable blocker 可以區分。
- [ ] Completion 需要 evidence，而不是只有 agent status claim。
- [ ] Raw runtime detail 不污染 canonical engineering Issue。
- [ ] Public example 全部使用 synthetic / de-identified data。
- [ ] Fresh orchestrator 可以透過 repository files 找到相關 state。

## Schema

請看：

- `schema/work-item.schema.json`
- `schema/stream-state.schema.json`
- `schema/run-state.schema.json`
- `schema/event.schema.json`

## Example

請看：

```text
examples/acme/example-app/work-items/WORK-123/
```

這個 example 刻意使用完全虛構的名稱與 identifiers。