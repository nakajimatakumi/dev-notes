# 個人開発で設計書をGit管理する

このページでは、個人開発プロジェクトのコードと設計書を同じGitリポジトリで管理する方法を整理する。設計書はMarkdownで作成し、GitHubやGitLabでは通常のドキュメントとして、Obsidianではリンク付きのWikiのように閲覧する。

この運用は一般に「モノレポ」よりも「Docs as Code」に近い。設計書をコードと同じ変更履歴、ブランチ、Pull Requestで扱い、実装変更と設計変更のずれを減らすことが目的である。

## この方法が向いている場合

- 設計書と実装を同じタイミングで更新したい。
- 変更理由と差分をGitで追跡したい。
- Markdownで表現できる設計書が中心である。
- GitHubやGitLab上でも設計書を読めるようにしたい。
- 将来、複数人でレビューする可能性がある。

非開発者による同時編集、細かな閲覧権限、PowerPointやExcel中心の資料管理が必要な場合は、NotionやConfluenceなどとの併用を検討する。

## 推奨ディレクトリ構成

```text
project/
├─ src/
├─ tests/
├─ docs/
│  ├─ README.md
│  ├─ requirements/
│  ├─ architecture/
│  ├─ features/
│  ├─ adr/
│  ├─ api/
│  └─ diagrams/
├─ .github/
│  ├─ CODEOWNERS
│  └─ pull_request_template.md
└─ README.md
```

| ディレクトリ | 管理する内容 |
| --- | --- |
| `docs/requirements/` | 目的、対象範囲、機能要件、非機能要件、受入条件 |
| `docs/architecture/` | システム構成、コンポーネント分割、データフロー |
| `docs/features/` | 機能単位の画面、処理、状態、例外 |
| `docs/adr/` | 技術選定や設計判断と、その理由 |
| `docs/api/` | APIの入出力、エラー、認証・認可 |
| `docs/diagrams/` | Mermaidなどのテキスト図と必要な画像 |

`docs/README.md`を設計書の入口にする。利用者がディレクトリを直接探索しなくても、要件、構成、機能設計、判断履歴へ移動できる索引を用意する。

## MarkdownリンクとObsidian

設計書同士のリンクには、標準Markdownの相対リンクを使用する。

```md
[認証機能の設計](./features/authentication.md)
[認証方式](./features/authentication.md#認証方式)
```

標準Markdownリンクなら、ObsidianだけでなくGitHub、GitLab、VS Code、Markdownビューアーでもリンクとして利用できる。

Obsidian固有のWikilinkも利用できるが、他の環境ではリンクとして解釈されない場合がある。

```md
[[features/authentication]]
```

相互運用性を優先する場合は、Obsidianの「設定」→「ファイルとリンク」→「Wikilinkを使用」を無効にする。リンク補完などの編集支援を使いながら、標準Markdownリンクを生成できる。

次の機能はMarkdown自体ではなく、Obsidianが提供する。

- バックリンク一覧
- グラフビュー
- リンク先のプレビュー
- ファイル名変更時のリンク更新
- 未作成ページへのリンクからのファイル作成

そのため、設計書の保存形式は標準Markdownにし、Obsidianは閲覧・編集を便利にする任意のクライアントとして扱う。

## 基本的なブランチ運用

設計書用とコード用の長期ブランチを分けず、`main`を唯一の正本にする。

```text
main
├─ feature/login
├─ docs/login-design
└─ fix/login-error
```

| ブランチ例 | 用途 |
| --- | --- |
| `feature/login` | 機能実装。必要なら関連設計書も同じブランチで変更する |
| `docs/login-design` | 設計書だけを追加・修正する |
| `fix/login-error` | 不具合修正。仕様や設計の訂正があれば設計書も変更する |

ブランチ名は作業内容を伝えるための規約であり、設計書の編集権限を保証するものではない。

実装に関係する設計変更は、できるだけコードと同じPull Requestに含める。機能のリリースと設計書の更新時期が揃い、レビュー担当者もコードと説明の整合性を確認できる。

## Pull Requestでの保護

複数人で管理する場合は、`main`への直接pushを禁止してPull Requestを必須にする。そのうえで、変更されたパスごとに承認者を設定する。

```text
# .github/CODEOWNERS

/docs/  @example/design-team
/src/   @example/development-team
```

`CODEOWNERS`を設定すると、変更されたファイルの担当者へレビューを自動依頼できる。ブランチ保護でCode Ownerのレビューを必須にすれば、担当者の承認なしでのマージを防げる。

設計担当と開発担当の両方から必ず承認を得たい場合は、RulesetのRequired reviewersでチームごとのファイルパターンと必要承認数を設定する。

| 変更内容 | 必要な確認 |
| --- | --- |
| `docs/**`だけを変更 | 設計担当者の承認 |
| `src/**`だけを変更 | 開発担当者の承認 |
| `docs/**`と`src/**`を変更 | 設計と開発の両方の確認 |

`CODEOWNERS`はファイルの編集を禁止する機能ではない。誰でも変更案を作成できるが、担当者が承認しなければ正本へマージできない、というレビュー制御である。

## 強い編集制限が必要な場合

利用可能なリポジトリ種別やプランでは、GitHubのPush Rulesetを使って、指定したファイルパスを含むpushを拒否できる。バイパス対象を設計担当チームに限定すれば、設計担当者以外による`docs/**`の変更を強く制限できる。

ただし、この制限は開発者が実装と一緒に設計書を修正することも妨げる。通常は、編集自体を禁止するより、Pull Requestと必須レビューで正本への反映を制御するほうが扱いやすい。

## 派生元ブランチによる制限を避ける理由

「設計用ブランチから派生したブランチだけが`docs/**`を変更できる」という条件は、GitHubの一般的なブランチ保護では直接扱いにくい。

GitHub Actionsでブランチ名、変更パス、コミット履歴を検査し、必須Status Checkにすることはできる。しかし、長期的なコード用ブランチと設計用ブランチを持つと、次の問題が生じる。

- 設計と実装のどちらが最新か分かりにくくなる。
- 両ブランチ間のマージ作業が増える。
- 同じ機能のコードと設計書が別々のPull Requestになりやすい。
- ブランチ名や派生元の検査ルールが複雑になる。

そのため、`main`を正本とし、変更パスに応じてレビュー条件を変える。

## Pull Requestの確認項目

Pull Requestテンプレートには次の項目を用意する。

```md
## 設計書

- [ ] 設計書の変更は不要
- [ ] `docs/`を更新した
- [ ] ADRを追加または更新した
- [ ] コードと設計書の内容が一致している
```

レビューでは次を確認する。

- 実装変更によって既存設計書が古くなっていないか。
- 設計書に目的、前提、対象外、例外が書かれているか。
- コード、API、テストから関連設計書へたどれるか。
- 設計判断の理由をADRに残す必要がないか。
- リンク切れや移動後の参照漏れがないか。

## 個人開発での最小構成

最初から細かいルールをすべて導入する必要はない。個人開発では次の構成から始める。

1. `docs/README.md`を作成する。
2. `docs/architecture/`、`docs/features/`、`docs/adr/`を作成する。
3. 標準Markdownの相対リンクを使用する。
4. 実装変更時に関連設計書の更新要否を確認する。
5. 必要になった段階でPull Request、CODEOWNERS、Rulesetを追加する。

Obsidianを使用する場合は、リポジトリ全体または`docs/`をVaultとして開く。Obsidianがない環境でも読めることを前提にしておけば、特定ツールへの依存を抑えながらWikiのような閲覧性を得られる。

## 関連ノート

- [要件定義書の書き方](../requirements-definition/requirements-definition-guide.md)

## 参考資料

確認日: 2026-07-31

- [Obsidian Help: Internal links](https://obsidian.md/help/links)
- [GitHub Docs: Available rules for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [GitHub Docs: About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitLab Docs: Documentation workflows](https://docs.gitlab.com/development/documentation/workflow/)
