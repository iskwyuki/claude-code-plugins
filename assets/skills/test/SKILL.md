---
description: テスト実行＋結果解析
trigger: "when the user asks to run tests"
disable-model-invocation: true
---

# テスト実行スキル

プロジェクトのテストを実行し、結果を分析する。

## 引数

- 引数なし → 全テスト実行
- ファイルパス → 指定ファイル/ディレクトリのテストのみ実行
- `--failed` → 前回失敗したテストのみ再実行

## 手順

1. プロジェクトのテストフレームワークを検出する

| ファイル | フレームワーク | 実行コマンド |
|---------|-------------|------------|
| package.json (jest/vitest) | Jest / Vitest | `npm test` / `npx vitest` |
| pytest.ini / pyproject.toml | pytest | `pytest` |
| Cargo.toml | cargo test | `cargo test` |
| go.mod | go test | `go test ./...` |

2. テストを実行する
3. 結果を以下の形式で報告する

### 報告形式

```
## テスト結果

- 合計: X件
- 成功: X件
- 失敗: X件
- スキップ: X件

### 失敗テスト詳細
（失敗がある場合のみ）

#### テスト名
- ファイル: path/to/test
- エラー内容: ...
- 推定原因: ...
- 修正案: ...
```

4. 失敗テストがある場合、修正するか確認する

## 注意事項

- テスト環境のセットアップが必要な場合は先に確認する
- テスト実行に時間がかかる場合は、変更に関連するテストのみの実行を提案する
- CI環境との差異に注意する（環境変数、DB接続等）
