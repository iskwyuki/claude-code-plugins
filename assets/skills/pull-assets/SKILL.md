---
name: pull-assets
description: 配信元リポジトリ (claude-code-plugins) の asset をプロジェクトの .claude/ へ同期する。assets/ 配下のディレクトリを動的走査するため、将来 hooks や commands が追加されても skill 本体の変更なしで対応する。
---

# pull-assets

Plugin キャッシュ `~/.claude/plugins/claude-code-plugins/assets/` の内容をプロジェクトの `.claude/` にコピーする skill。2 回目以降の継続運用で使う。

## 使い方

- `/pull-assets` - assets 配下の全ディレクトリを同期
- `/pull-assets --dry-run` - 差分表示のみで実変更なし
- `/pull-assets --only=<dir>` - 特定ディレクトリだけ同期（例: `--only=skills`）

## 手順

### Step 1: 引数パース

ユーザー入力から以下を取得:
- `--dry-run` フラグ
- `--only=<dir>` の値（存在すればフィルタとして使用）

### Step 2: 配信元キャッシュの存在確認

```
test -d ~/.claude/plugins/claude-code-plugins/assets || echo "MISSING"
```

`MISSING` の場合は以下を案内して終了:
1. `/plugin marketplace update` で最新化
2. `/plugin install claude-code-plugins@iskwyuki` で再 install

### Step 3: 配布対象ディレクトリの動的走査

```
ls -1 ~/.claude/plugins/claude-code-plugins/assets/
```

出力された各ディレクトリを配布対象として扱う。`--only=<dir>` 指定時はそのディレクトリだけに絞る。

**重要**: ハードコードで skills/agents のみを対象にしないこと。assets 直下にあるものすべてを動的に検出する。

### Step 4: 各ディレクトリの差分表示

対象ディレクトリごとに rsync の dry-run で差分を表示する。

```
rsync -avn ~/.claude/plugins/claude-code-plugins/assets/<type>/ ./.claude/<type>/
```

出力を整形し、以下を区別して提示:
- 新規追加されるファイル
- 上書きされる既存ファイル

`--dry-run` 指定時はここで終了。

### Step 5: 同期の実行

AskUserQuestion で「同期してよいか」を確認してから実コピーする。

```
rsync -av ~/.claude/plugins/claude-code-plugins/assets/<type>/ ./.claude/<type>/
```

- `--delete` は使わない（プロジェクト固有の skill を誤って削除しないため）
- コピー先ディレクトリがなければ作成される

### Step 6: 変更の報告

```
git status -- .claude/
```

変更されたファイルの一覧を表示し、`git add .claude/ && git commit -m "chore: claude-code-plugins 同期"` を案内する。

## 衝突時の取扱い

配信元 asset とプロジェクト固有 skill のディレクトリ名が同じ場合、配信元側が上書きする。プロジェクト固有で独自機能を持たせたい場合は、**ディレクトリ名を配信元と重複させない命名**にすること（例: `/review` は配信元、`/review-custom` はプロジェクト固有）。
