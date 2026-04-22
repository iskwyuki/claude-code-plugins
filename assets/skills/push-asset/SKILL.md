---
name: push-asset
description: プロジェクトの .claude/<type>/<name>/ を配信元リポジトリ (claude-code-plugins) の assets/ にコピーし、他リポジトリからも利用できるようにする。「この skill を他リポジトリでも使いたい」導線。
---

# push-asset

プロジェクトで作成・改善した asset を配信元に昇格させる skill。
pull-assets と対になる、逆方向の同期。

## 使い方

- `/push-asset <type> <name>` - 例: `/push-asset skills deploy-check`
- `/push-asset <type> <name> --dry-run` - 差分表示のみ
- `/push-asset <type> <name> --auto-commit` - 配信元での commit & push まで自動実行

`<type>` は skills / agents / hooks / commands / mcp など、プロジェクトの `.claude/` 直下にあるディレクトリ名。

## 手順

### Step 1: 引数パース

- `<type>`, `<name>` を必須引数として取得
- `--dry-run`, `--auto-commit` フラグを解釈

### Step 2: プロジェクト側の対象確認

```
test -e .claude/<type>/<name> || echo "MISSING"
```

ディレクトリまたはファイル（例: `agents/foo.md`）の存在を確認。
見つからない場合はエラーで終了。

### Step 3: 配信元リポジトリのローカルパス解決

優先順位:
1. 環境変数 `$CLAUDE_PLUGINS_REPO`
2. `~/dev/claude-code-plugins`
3. `~/dev/claude-code-template`（リネーム前の後方互換）

最初に存在するものを採用。見つからなければ以下を案内して終了:

```
git clone https://github.com/iskwyuki/claude-code-plugins.git ~/dev/claude-code-plugins
```

### Step 4: 差分表示

配信元 `<repo>/assets/<type>/<name>` の存在確認:
- 未存在: 「新規追加」として表示
- 存在: `diff -ruN <repo>/assets/<type>/<name> .claude/<type>/<name>` で差分表示

`--dry-run` 指定時はここで終了。

### Step 5: 実コピー

AskUserQuestion で「配信元に push してよいか」を確認してから実コピーする。

```
rsync -av --delete .claude/<type>/<name>/ <repo>/assets/<type>/<name>/
```

単一ファイル（agents/foo.md 等）の場合は `cp -f`。

### Step 6: 配信元での commit & push

`--auto-commit` 指定時のみ自動実行:

```
cd <repo>
git add assets/<type>/<name>
git commit -m "feat(<type>): add <name>"
git push
```

指定がなければ以下を案内して終了:

```
cd <repo>
git add assets/<type>/<name>
git commit -m "feat(<type>): add <name>"
git push
```

### Step 7: 他プロジェクトへの反映手順を案内

以下を他プロジェクトのルートで実行すれば最新の asset が届く、と伝える:

```
/plugin marketplace update
/pull-assets
git add .claude/ && git commit -m "chore: claude-code-plugins 同期"
```

## 注意事項

- 配信元に push する asset は **技術スタックに依存しない汎用性** を持つこと。プロジェクト固有の npm スクリプトや特定フレームワーク向けの実装を含む場合は、プロジェクト側に残す判断を検討する
- 既存 asset を上書きする場合、他プロジェクトに影響するため差分を慎重に確認すること
