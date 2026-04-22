---
description: オープン中のGitHub Issueをカテゴリ・概要付きで一覧表示する
trigger: "when the user asks to see open issues, TODOs, or remaining tasks"
disable-model-invocation: true
---

# TODOスキル

オープン中のIssueを取得し、カテゴリ別に整理して表示する。

## 手順

1. `gh issue list --state open --limit 50 --json number,title,labels,body,assignees,milestone,createdAt` でオープン中のIssueを取得する
2. ラベルをカテゴリとして分類する（ラベルなしは「未分類」）
3. 以下の形式で表示する

### 表示形式

```
## TODO一覧（オープンIssue: N件）

### 🐛 bug
- #12 ログイン画面でエラーが発生する — ログインボタン押下時に500エラー
- #8  メール送信が失敗する — SMTPタイムアウト

### ✨ enhancement
- #15 ダッシュボードにグラフ追加 — 月別売上の棒グラフを表示

### 未分類
- #3  READMEの更新 — セットアップ手順を最新化
```

各Issueの表示項目:
- `#番号` Issue番号
- タイトル
- `—` の後にbodyの先頭1-2文を概要として表示（長い場合は50文字で切る）

4. 担当者やマイルストーンが設定されている場合は補足情報として表示する

## 注意事項

- Issueが0件の場合は「オープン中のIssueはありません」と表示する
- ラベルが複数ある場合は最初のラベルで分類する
