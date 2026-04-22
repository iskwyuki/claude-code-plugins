---
name: bootstrap
description: claude-code-plugins の初回セットアップ skill。Plugin install 直後に /claude-code-plugins:bootstrap で呼び出し、SETUP.md を提示してからプロジェクトの .claude/ に全 asset を初回展開する。
---

# bootstrap

Plugin `claude-code-plugins` を install した直後に 1 度だけ呼び出す踏み台 skill。
Plugin skill は namespaced で呼ぶ必要があるが、bootstrap によってプロジェクトに展開された `pull-assets` / `push-asset` は短縮名で使える。

## 手順

### Step 1: SETUP.md を提示

Plugin キャッシュ配下の SETUP.md を読み込んでユーザーに概要を伝える。

```
cat ~/.claude/plugins/claude-code-plugins/SETUP.md
```

主要項目を抜粋して表示（初回セットアップの流れ、短縮名 skill の一覧、環境変数）。

### Step 2: 配信元キャッシュの存在確認

```
test -d ~/.claude/plugins/claude-code-plugins/assets || echo "MISSING"
```

`MISSING` の場合はユーザーに `/plugin install claude-code-plugins@iskwyuki` の実行を促して終了。

### Step 3: assets 配下のディレクトリを動的走査

配布対象を決め打ちせず、`assets/` 直下に存在するディレクトリを列挙する。

```
ls -1 ~/.claude/plugins/claude-code-plugins/assets/
```

出力された各ディレクトリ名 `<type>` を後続の同期対象とする。現時点では `skills`, `agents` が存在するが、将来 `hooks`, `commands`, `mcp` などが増えたときも自動で対象に加わる。

### Step 4: 差分の事前提示

プロジェクトルートの `.claude/<type>/` と配信元 `~/.claude/plugins/claude-code-plugins/assets/<type>/` の差分を表示する。

```
rsync -avn ~/.claude/plugins/claude-code-plugins/assets/<type>/ ./.claude/<type>/
```

新規追加 / 上書き対象を整理してユーザーに見せる。

### Step 5: 同期の実行

AskUserQuestion で「このまま同期してよいか」を確認してから実コピーする。削除はしない（プロジェクト固有ファイルを守る）。

```
rsync -av ~/.claude/plugins/claude-code-plugins/assets/<type>/ ./.claude/<type>/
```

### Step 6: 完了報告

- `.claude/` 配下の変更を `git status -- .claude/` で確認させる
- `git add .claude/ && git commit -m "chore: claude-code-plugins 初回同期"` を案内
- 以降は `/pull-assets` と `/push-asset` が短縮名で利用可能になる旨を伝える
