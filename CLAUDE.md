# CLAUDE.md

Claude がこのリポジトリで作業するときに必ず従う指示書。詳細は `docs/project-plan/01.md` を参照すること。

## このリポジトリの目的

- 「読んだ本と感想」を CRUD できる Flask アプリを題材にした、**開発工程（issue・PR・ブランチ戦略）の見本**。
- オーナーは実務経験のない学習者で、ポートフォリオ作成のために工程を学んでいる。issue・PR・コミット・作業履歴は**後から読んで学べる教材**として丁寧に書く。

## 作業の進め方（最重要）

1. 作業を始める前にプランを提示し、オーナーと合意する。
2. **プランが決まったら、オーナーの許可を待たずに最後まで自動で進める。** ファイル作成、コミット、push、issue 作成、PR 作成、マージまでを含む。
3. 止まって確認するのは次のときだけ。
   - プランの範囲を超える変更が必要になったとき
   - 方針に関わる判断（技術選定、仕様の解釈など）が必要になったとき
   - データ削除や本番環境（GCP）の破壊的な操作など、元に戻せない操作
4. **作業のたびに `docs/histories/NN.md` を連番で作る**（後述）。

## ブランチ戦略（3段構成）

| ブランチ | 切る元 | マージ先 | マージ方法 |
| --- | --- | --- | --- |
| `main` | — | — | — |
| `feature/<機能>` | `main` | `main` | Merge commit |
| `dev/<機能>/<issue番号>-<作業>` | `feature/<機能>` | `feature/<機能>` | Squash merge |
| `hotfix/<issue番号>-<内容>` | `main` | `main` | Squash merge |
| `spike/<issue番号>-<内容>` | どこからでも | マージしない | — |

- `main` に直接コミットしない。必ず PR を経由する。
- **ブランチは削除しない**（マージ済みも残す）。マージ済みのブランチには追加でコミットしない。
- push 済みの `main` と `feature/*` では rebase・force-push・amend を禁止する。同期はマージで行う。
- ブランチ名は英小文字とハイフンのみ。`dev/` `hotfix/` `spike/` には issue 番号を入れる。

## Issue

- 種類: Epic（`feature/*` と1対1）/ Task（`dev/*` と1対1）/ Bug / Investigation（`spike/*`）。
- 本文は「背景 / 目的 / 完了条件（チェックリスト）/ ブランチ / 補足」で書く。
- Task は Epic の sub-issue にする。
- `Closes #n` は main へのマージでしか動かない。そのため次のように運用する。
  - `dev/*` → `feature/*` の PR: 本文に `Refs #n` と書き、マージ後に Task issue を**手動でクローズ**する（completed）。
  - `feature/*` → `main` の PR: 本文に `Closes #<Epic>` と書く。
- open のまま放置しない。対応しないものは理由を書いて not planned でクローズする。

## Pull Request

- タイトルは Conventional Commits 形式の英語。例: `feat(book): add create form`。
- 本文は日本語で「概要 / 関連 issue / 変更内容 / 動作確認 / レビュー観点」。
- base ブランチを必ず確認する（`dev/*` なら `feature/*`）。
- 差分は300行程度まで（lock ファイルと自動生成されたマイグレーションは除く）。
- CI がある場合、赤いままマージしない。

## コミット

- **コミットメッセージは件名も本文も必ず英語。**
- Conventional Commits: `feat` / `fix` / `docs` / `test` / `refactor` / `chore` / `ci` / `build`。
- 件名は命令形・小文字始まり・ピリオドなし・50文字程度。1コミットに1つの意図。

## ドキュメント

- issue・PR・ドキュメントは日本語で書く。
- トピック別ドキュメント: `docs/<トピック名>/NN.md`（トピック名は英小文字のケバブケース）。改訂や続編は既存ファイルを書き換えず、次の番号で追加する。
- 作業履歴: `docs/histories/NN.md`。作業のたびに1つ作り、同じ作業ブランチでコミットする。書く内容:
  - 日付、関連 issue / PR / ブランチ、作成・変更ファイル
  - 依頼内容、調査したこと、決定事項（質問と回答）、判断の理由
  - 未完了事項と次の作業

## 技術スタック

Python `==3.15.*`（uv `==0.13.*`、uv_build `==0.13.*`）/ Flask / Jinja2 + Bootstrap 5（CDN、Node なし）/ Flask-SQLAlchemy / Flask-Migrate / psycopg 3 / Flask-WTF / gunicorn / PostgreSQL / Docker Compose / GCP（Cloud Run, Cloud SQL, Artifact Registry, Secret Manager）。

開発ツール: Ruff `==0.17.*` / mypy / pytest / pre-commit（**ローカルフックのみ**、`uv run` で実行）/ just。

## コマンド（導入後）

```bash
just verify     # ruff + mypy + pytest
just modify     # ruff --fix + format
just up         # docker compose up --build
```

コードを変更したら、コミットの前に `just verify` を通す。
