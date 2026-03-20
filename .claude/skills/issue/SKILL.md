---
description: GitHub Issueの作成・更新・クローズ
trigger: "when the user asks to create, update, or close a GitHub issue"
disable-model-invocation: true
---

# Issue管理スキル

GitHub CLIを使用してIssueを操作する。

## 引数

- 引数なし → 操作を選択（作成 / 更新 / クローズ）
- `create` または `作成` → 新規作成
- `close #番号` → 指定Issueをクローズ
- `#番号` → 指定Issueの更新

## 作成フロー

1. ユーザーにIssueの内容をヒアリングする（タイトル・詳細）
2. 以下のテンプレートでIssueを作成する

```markdown
## 概要
{概要}

## 詳細
{詳細・背景}

## 完了条件
- [ ] {条件1}
- [ ] {条件2}
```

3. ラベルの候補を提示し、選択してもらう
4. `gh issue create` で作成する

## 更新フロー

1. `gh issue view {番号}` で現在の状態を確認
2. ユーザーに変更内容を確認
3. `gh issue edit` で更新する

## クローズフロー

1. `gh issue view {番号}` で内容を確認
2. クローズ理由をコメントとして追加
3. `gh issue close` でクローズする

## 注意事項

- Issue作成前に、重複がないか `gh issue list` で確認する
- ラベルが未作成の場合は作成を提案する
