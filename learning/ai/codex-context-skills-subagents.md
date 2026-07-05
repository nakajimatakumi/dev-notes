# Codex に事前知識を渡す方法と Skills / Subagents の使い方

Codex には、毎回チャットで説明しなくても文脈や作業ルールを渡す方法がいくつかある。用途ごとに置き場所と効き方が違うので、まずは次の順で考えると整理しやすい。

| 目的                  | 使うもの               | 置き場所の例                                      |
| ------------------- | ------------------ | ------------------------------------------- |
| リポジトリの恒久ルールを渡す      | `AGENTS.md`        | repo root、またはサブディレクトリ                       |
| 自分個人の好みや共通ルールを渡す    | global `AGENTS.md` | `~/.codex/AGENTS.md`                        |
| 繰り返し使う作業手順を部品化する    | Skill              | `.agents/skills/`、`$HOME/.agents/skills/`   |
| 外部サービスや社内情報に接続する    | MCP                | `~/.codex/config.toml`、`.codex/config.toml` |
| 過去スレッドの安定した文脈を再利用する | Memories           | `~/.codex/memories/`                        |
| 調査やレビューを並列化する       | Subagents          | prompt、または `.codex/agents/`                 |

## AGENTS.md

`AGENTS.md` は Codex が作業前に読む永続的な指示ファイル。ビルド・テストコマンド、レビュー観点、ディレクトリ構成、命名規則など「この repo では毎回守ってほしいこと」を書く。

Codex は起動時に instruction chain を作る。大まかな順序は次の通り。

1. `~/.codex/AGENTS.override.md` または `~/.codex/AGENTS.md`
2. repo root から current working directory までの `AGENTS.override.md` / `AGENTS.md`
3. より深いディレクトリの指示ほど後に入り、前の指示を上書きしやすい

使い方の目安:

- repo 全体のルールは root の `AGENTS.md`
- 特定領域だけのルールは `learning/ai/AGENTS.md` のように近い場所
- 一時的に全体を上書きしたい場合は `AGENTS.override.md`
- 毎回チャットで説明しているルールは `AGENTS.md` に移す

書くべき内容は少なく保つ。長すぎると重要なルールが埋もれるので、繰り返し発生するミスやレビュー指摘だけを追加する。

## Memories

Memories は、過去スレッドから安定した好み・作業パターン・既知の注意点を Codex が再利用する仕組み。デフォルトでは無効の地域や環境があるため、使う場合は Codex 設定または `~/.codex/config.toml` で有効化する。

```toml
[features]
memories = true
```

ただし、チームで必ず守るルールは Memories ではなく `AGENTS.md` に置く。Memories は個人の補助記憶であり、必須ルールの唯一の保存場所にしない。

CLI/TUI では `/memories` から、現在のスレッドで memory を使うか、今後の memory 生成に使うかを制御できる。

## Skills

Skill は、Codex に再利用可能な作業手順や専門知識を持たせる仕組み。`SKILL.md` と、必要に応じて `scripts/`、`references/`、assets を含むディレクトリとして作る。

最小構成:

```md
---
name: commit-helper
description: Use when organizing changed files into focused commits.
---

1. Check `git status --short`.
2. Group files by purpose.
3. Stage only related files together.
4. Write a concise commit message.
```

保存場所:

- repo 用: `.agents/skills/<skill-name>/SKILL.md`
- 個人用: `$HOME/.agents/skills/<skill-name>/SKILL.md`
- 配布用: plugin として package 化

Codex は最初から全 Skill 本文を読むわけではない。まず `name` と `description` などのメタデータだけを見て、必要だと判断したときに `SKILL.md` を読む。これを progressive disclosure と呼ぶ。

起動方法:

- 明示的に使う: `/skills`、または prompt で `$skill-name`
- 暗黙的に使う: 依頼内容が `description` に合うと Codex が選ぶ

`description` は重要。何に使う Skill か、いつ使わないかを短く具体的に書く。

## Custom Prompts は原則使わない

Custom prompts は Markdown を slash command 化する古い仕組みだが、現在は非推奨。再利用可能な instructions を作りたい場合は Skills を使う。

Custom prompts は明示実行が必要で、基本的に `~/.codex/prompts/` に置くローカル機能。repo 共有や暗黙起動が必要なら Skill のほうが合っている。

## MCP

MCP は Codex を外部ツールや外部コンテキストに接続する仕組み。GitHub、Linear、Figma、社内ドキュメント、ブラウザ操作など、ローカル repo 外の情報や操作が必要なときに使う。

設定は主に `~/.codex/config.toml` に書く。信頼済み project では `.codex/config.toml` に置くこともできる。

CLI では次で確認できる。

```bash
codex mcp --help
```

TUI では `/mcp` で現在使える MCP tools を確認する。

Skill と MCP は組み合わせると強い。Skill が「どう作業するか」を定義し、MCP が「どの外部情報や操作を使えるか」を提供する。

## Subagents

Subagents は、Codex が専門化した agent を並列に動かし、結果をまとめる仕組み。大きな調査、レビュー、ログ分析、テスト観点の分担など、並列化しやすい作業に向いている。

基本方針:

- Codex は勝手に subagent を起動しない
- 使いたいときは prompt で明示する
- subagent ごとに model/tool work が発生するため token 消費は増える
- 書き込み中心の並列作業は衝突しやすいので注意する

例:

```text
このブランチを並列 subagents でレビューして。
1つ目はセキュリティ、2つ目はバグ、3つ目はテスト不足を見る。
全員の結果を待ってから、重要度順にファイル参照付きでまとめて。
```

CLI では `/agent` で active agent thread を切り替えたり、進行中の subagent を確認できる。

## Custom Agents

Subagents の役割を固定したい場合は custom agent を作れる。個人用は `~/.codex/agents/`、project 用は `.codex/agents/` に TOML ファイルを置く。

必須項目:

```toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, and missing tests."
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
"""
```

必要に応じて `model`、`model_reasoning_effort`、`sandbox_mode`、`mcp_servers`、`skills.config` なども指定できる。省略した項目は親セッションから継承される。

組み込み agent には `default`、`worker`、`explorer` がある。独自 agent の `name` が組み込み名と一致すると、独自設定が優先される。

## 使い分けの実践パターン

1. まず `AGENTS.md` に repo の基本ルールを書く。
2. 毎回やる作業手順が出てきたら Skill に分離する。
3. 外部サービスや社内情報が必要なら MCP を足す。
4. 調査・レビュー・要約を並列化したくなったら Subagents を使う。
5. よく使う専門役割が固まったら Custom Agent を作る。

この順序にすると、チャットで毎回説明する量を減らしつつ、必要なときだけ大きな文脈や専門手順を読み込ませられる。

## 出典

- OpenAI Codex Manual: `Customization`
- OpenAI Codex Manual: `Custom instructions with AGENTS.md`
- OpenAI Codex Manual: `Agent Skills`
- OpenAI Codex Manual: `Memories`
- OpenAI Codex Manual: `Model Context Protocol`
- OpenAI Codex Manual: `Subagents`
- 取得日: 2026-07-06
