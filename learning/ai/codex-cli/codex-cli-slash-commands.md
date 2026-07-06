# Codex CLI のスラッシュコマンド

Codex CLI の TUI では、入力欄で `/` を打つとスラッシュコマンドの候補を開ける。モデル変更、権限変更、差分確認、会話の整理などを、ターミナルから離れずに実行するためのショートカットとして使う。

タスク実行中でも、スラッシュコマンドを入力して `Tab` を押すと次のターンにキューできる。キューされたコマンドは、現在のターンが終わってから解釈される。

## よく使うコマンド

| コマンド | 使いどころ |
| --- | --- |
| `/status` | 現在のモデル、権限、サンドボックス、トークン使用量、コンテキスト残量を確認する。 |
| `/model` | 使用するモデルや、対応している場合は reasoning effort を変更する。 |
| `/permissions` | Codex が確認なしに実行できる操作の範囲を変える。Auto / Read Only などの切り替えに使う。 |
| `/diff` | Git 管理外ファイルを含めて、現在の作業ツリー差分を確認する。 |
| `/review` | 現在の作業ツリーを Codex にレビューさせる。 |
| `/compact` | 長くなった会話を要約して、コンテキストを節約する。 |
| `/new` | CLI を終了せずに、新しい会話を開始する。 |
| `/resume` | 保存済みの過去セッションを再開する。 |
| `/quit` / `/exit` | Codex CLI を終了する。 |

## セッションと会話の管理

| コマンド | 説明 |
| --- | --- |
| `/clear` | 画面をクリアし、同じ CLI セッション内で新しいチャットを始める。`Ctrl+L` は表示だけをクリアする点が違う。 |
| `/archive` | 現在のセッションをアーカイブして終了する。 transcript は削除されない。 |
| `/delete` | 現在のセッションを完全に削除して終了する。 transcript も削除されるので注意する。 |
| `/fork` | 現在の会話を分岐して、新しいスレッドとして続ける。別案を試したいときに使う。 |
| `/side` / `/btw` | メインの transcript を乱さずに、一時的な横道の質問をする。 |
| `/copy` | 直近の Codex 応答をコピーする。`Ctrl+O` でも同じ操作ができる。 |
| `/raw` | raw scrollback 表示を切り替える。長い出力を選択・コピーしやすくする用途。 |

## 実行方針の変更

| コマンド | 説明 |
| --- | --- |
| `/plan` | Plan mode に切り替える。実装前に手順を出してほしいときに使う。インラインで `/plan 依頼内容` のようにも書ける。 |
| `/goal` | 長めの作業目標を設定・表示・一時停止・再開・削除する。例: `/goal Finish the migration and keep tests green` |
| `/fast` | 対応モデルで Fast service tier の on/off/status を切り替える。モデルカタログが対応していない場合は表示されない。 |
| `/personality` | 応答スタイルを変更する。`friendly`、`pragmatic`、`none` などを選べる。 |
| `/approve` | auto review に拒否された直近の操作を、1 回だけ再試行するために承認する。 |

## 開発コンテキストを渡す

| コマンド | 説明 |
| --- | --- |
| `/ide` | IDE で開いているファイル、選択範囲などの文脈を次のプロンプトに含める。 |
| `/mention` | 特定のファイルやフォルダを会話に添付する。次に読んでほしい対象を明示するときに使う。 |
| `/init` | 現在のディレクトリに `AGENTS.md` の雛形を生成する。リポジトリ固有の永続指示を書く入口。 |
| `/sandbox-add-read-dir` | Windows で、現在の readable roots 外にある追加ディレクトリの読み取りを許可する。 |

## 拡張機能・外部連携

| コマンド | 説明 |
| --- | --- |
| `/skills` | 利用可能な skill を選んで、次の依頼に適用する。 |
| `/plugins` | インストール済み、または発見可能な plugin を確認・管理する。 |
| `/apps` | apps/connectors を探して、プロンプトに `$app-slug` として挿入する。 |
| `/mcp` | 設定済みの MCP tools を一覧表示する。`verbose` を付けるとサーバー詳細も確認できる。 |
| `/hooks` | lifecycle hook を確認・管理する。新規または変更された hook の trust、無効化などに使う。 |
| `/import` | Claude Code の設定、プロジェクトファイル、最近のチャットなど、対応している外部エージェント資産を Codex に取り込む。 |

## TUI と表示設定

| コマンド | 説明 |
| --- | --- |
| `/keymap` | TUI のショートカット割り当てを確認・永続化する。 |
| `/vim` | 入力欄の Vim mode を切り替える。 |
| `/theme` | シンタックスハイライトのテーマを選ぶ。 |
| `/statusline` | footer に表示する項目を選び、順序を変更して `config.toml` に保存する。 |
| `/title` | ターミナルのウィンドウ/タブタイトルに出す項目を設定する。 |

## 診断・アカウント・設定

| コマンド | 説明 |
| --- | --- |
| `/usage` | ChatGPT token 使用量や rate limit reset を確認する。 |
| `/debug-config` | config のレイヤー、requirements、実験的なネットワーク制約などの診断を出す。 |
| `/experimental` | experimental features を切り替える。必要に応じて Codex の再起動が求められる。 |
| `/memories` | memory の注入や生成を有効/無効にする。 |
| `/feedback` | Codex maintainers にログや診断情報を送る。 |
| `/logout` | ローカルの認証情報を消してサインアウトする。共有マシンでは特に重要。 |

## バックグラウンド処理

| コマンド | 説明 |
| --- | --- |
| `/ps` | experimental な background terminal と最近の出力を確認する。 |
| `/stop` | 現在のセッションが開始した background terminal をすべて停止する。 |
| `/agent` | spawn された subagent の thread を確認・切り替える。 |

## 使い分けの目安

- 状況確認なら、まず `/status` と `/diff`。
- 作業前に方針を固めたいなら `/plan`。
- 長い作業を継続的に追わせたいなら `/goal`。
- モデルや応答の性格を変えたいなら `/model`、`/fast`、`/personality`。
- 追加情報を渡したいなら `/mention`、`/ide`。
- 会話が長くなってきたら `/compact`。
- 別案を試すなら `/fork`、完全に切り替えるなら `/new`。

## 注意点

- `/quit` と `/exit` は同じく CLI を終了する。未保存・未コミットの重要な作業がないか確認してから使う。
- `/delete` は transcript を削除する。単に一覧から隠したいだけなら `/archive` を使う。
- `/permissions` は作業の安全性に直結する。信頼できないリポジトリや不明なコマンドを扱うときは緩めすぎない。
- `/approve` は、auto review が拒否した操作を再試行したい場合だけ使う。

## 出典

- OpenAI Codex Manual: `Slash commands in Codex CLI`
- 取得日: 2026-07-06
