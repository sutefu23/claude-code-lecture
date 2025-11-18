---
marp: true
theme: kouen
class: lead
paginate: true
backgroundColor: white
---

# はじめての Claude Code

---
<style scoped>
session {
  font-size: 24px;
}
p{
  line-height: 1em;
}
</style>

# インストール

2025年11月より次で説明する各OSに対応したネイティブインストールが推奨

Homebrew (macOS, Linux):
```bash
brew install --cask claude-code
```

macOS, Linux, WSL:
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Windows PowerShell:
```bash
irm https://claude.ai/install.ps1 | iex
```

Windows コマンドプロンプト:
```bash
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

---

# 料金体系２つ

- サブスクリプションプラン

  Pro (20ドル／月)、Max5x（100ドル／月）、 Max 20x（200ドル／月）


- 従量課金プラン

  **Sonnet 4.5**：

  入力100万トークンあたり$3、出力100万トークンあたり$15

  **Opus 4.1**：

  入力100万トークンあたり$15、出力100万トークンあたり$75


<i>Claude.aiアカウントとClaude Consoleアカウントは別物なので注意。</i>

---

# 立ち上げ

```
claude
```

```
> /init
```

CLAUDE.mdが生成される。
これを「メモリ」といいセッション単位でしかやりとりを保持しないClaudeの引き継ぎデータとして機能する。

ここには技術要件やコード全体の仕様などが記載される。プログラマーにとってのREADMEのようなもの。

必ずコンテキストに含まれる。

---

# 設定ファイル

`.claude/settings.local.json`
ここにファイルの編集許可、実行できるツールの設定
対話モードで回答すると自動で入ります。

---

# Claude Codeの4つのモード

Claude Codeには4つの動作モードがあります。

| モード | 説明 |
| ---- | ---- |
| default| 	デフォルトの動作。各ツールの初回使用時に権限の許可を求める |
| acceptEdits| 	セッション中のファイル編集権限を自動的に受け入れる |
| plan	| Claudeが分析と計画策定のみ行う。ファイルの変更やコマンドの実行は行わない。 |
| bypassPermissions	| すべての権限プロンプトをスキップする（安全な環境が必要） |


---

# MCPサーバーとは

- MCPはLLMと外部ツールがやり取りするための通信規格

- MCPサーバーはその通信規格を使って外部ツールやデータへのアクセスを提供
結果的に拡張機能として様々な能力をClaudeに与える

Goolg DriveやGitHub、Jira、Notionなどの読込、書き込みなど、外部サービスと連携可能

---

# MCPサーバーの設定

```
claude mcp add <MCPサーバーの名前> <MCPサーバーのURLや立ち上げコマンド>
```

---

# おすすめMCPサーバー1

## Context7 MCPサーバー

Context7は各ライブラリやフレームワーク、APIドキュメントなどの技術ドキュメントをClaudeに提供し、コード生成や修正、レビューの精度を向上させる。

## - **インストール方法**
```bash
claude mcp add context7 -s user -- npx -y @upstash/context7-mcp
```

```bash
> TODOアプリをNext.jsのApp Routerを使って作ってください。データはローカルに保存してください。 use context7
```

---

# おすすめMCPサーバー2

## Playground MCPサーバー

フロントの表示確認、エラー確認、動作確認など、Claude Codeに目を付ける事ができる。
「PlaywrightMCPサーバーで確認して、直るまで修正を繰り返して」と伝えると効果的。

Consoleエラーなども取得可能。

## - **インストール方法**
```bash
claude mcp add Playwright npx @Playwright/mcp@latest
```

最近はChrome MCPサーバーも人気。ログインセッションなども引き継げる。

---

# おすすめMCPサーバー3

## Serena
SerenaはIDEと同じようにLSP（Language Server Protocol）を提供することで（脚注）、Claudeに対してより深いコード理解をもたらす。

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

```bash
claude mcp add serena \
  -- uvx --from git+https://github.com/oraios/serena serena-mcp-server \
  --context ide-assistant --project $(pwd)
```

---
<!--_class: lead -->

# インフラを任せるとClaude Codeは最強

ghコマンド、gcpコマンド、awsコマンドなどを使ってインフラ構築や管理をClaude Codeに任せることができる。

---

# 並列実行

CLIの特性を活かして、複数のClaude Codeセッションを並列で実行することも可能。

Git Worktreeを使って、異なるタスクを同時に進行できる。

## Git Worktreeとは
Git Worktreeを使うと追加のcloneをせずに、同じ.gitを共有して複数の作業環境を持てるようになる。
`git worktree add -b ./feature-x feature-x`などのコマンドで特定のディレクトリに特定のブランチをチェックアウト可能。

---

## 平行処理のコツ

並行処理なのでどうしてもコードにコンフリクトが生じることがある。

違うセッションのClaudeは別人格のようなものなので、なるべくお互いの作業を被らせないことが基本的なコツ。

そのためフロントエンド、バックエンド、テスト、インフラ、などそれぞれ担当する作業を分けた方が良い。
たとえば一人にはフロントエンドをやらせる、一人にはバックエンドをやらせる、など。
特に立ち上がり時などは力を発揮する。