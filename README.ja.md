# omakase-pr 🍣

[English](./README.md) | **日本語**

**PRテンプレートは、おまかせで。**

GitHub ではPRテンプレートを複数置けますが、PRを作るときに[選ぶ手段がありません](https://github.com/orgs/community/discussions/146146)。結局、バグ修正にも新機能にも同じテンプレートを使うことになりがちです。

omakase-pr は Claude Code と Codex 向けのskillです。ブランチ名やコミット、差分から変更の種類を見分けて、合うテンプレートを選びます。中身を埋めて担当者とラベルを付け、`gh pr create` でPRを作るところまで進めます。選ぶのも書くのも、板前におまかせです。

## 使うとこうなる

```
> omakase

Opened https://github.com/acme/shop/pull/412
Kind: fix, from the branch name fix/login-timeout. Assigned to you, labeled bug.
```

本文は fix 用テンプレートの見出しに沿って埋まります。不具合の内容と原因、修正内容、テストの順です。チェックボックスにチェックを入れるのは根拠があるときだけで、実行していないテストを「実行済み」と書くことはありません。

タイトルは35字程度にまとめ、触ったファイルではなく「何が変わるか」を書きます。本文は1分で読める長さが目安です。差分をなぞるのではなく、要点とレビューで見るべき場所を伝えます。変更の理由がどこにも書かれていなければ、作り話で埋めずに確認してきます。

## テンプレートはそのまま使う

設定なしで使えます。リポジトリにあるテンプレートをそのまま使い、書き換えることはありません。

- **複数ある**（`PULL_REQUEST_TEMPLATE/`）：ファイルごとに1つの種類として扱い、合うものを選びます。
- **1つだけある**：どの種類でもそれを使います。種類は、タイトルの接頭辞とラベルを決めるのに使います。
- **Organization 共通のテンプレートがある**（`.github` リポジトリ）：同じように使います。リポジトリにはコピーしません。
- **ない**：omakase-pr の feature、fix、refactor、docs、chore 用テンプレートを使います。言語は最近のPRやコミットに合わせるので、日本語のチームなら日本語版です。

探す場所は GitHub と同じく、`.github/`、`docs/`、リポジトリのルートです。

## 担当者とラベル

PRの担当者（Assignees）は自分になり、種類に合うラベルが付きます。ラベルはリポジトリにあるものから選び、fix なら `bug`、feature なら `enhancement`、docs なら `documentation` です。ラベルを新しく作ることはありません。fork から出すPRのように、ラベルを付ける権限がないときは付けません。

## 種類の決め方

上から順に見て、最初に決まったものを使います。

1. 指定した種類：`omakase fix`
2. ブランチ名：`fix/login-timeout`
3. コミットの接頭辞：`fix: handle expired session`
4. コミットメッセージの中身：`ログインのタイムアウトを修正`
5. 差分の中身：ドキュメントしか変えていなければ docs

それでも決まらないときだけ、1回確認します。

## インストール

### Claude Code

```
/plugin marketplace add 6igtree/omakase-pr
/plugin install omakase-pr@omakase-pr
```

### Codex

```sh
tmp=$(mktemp -d) && git clone --depth 1 https://github.com/6igtree/omakase-pr.git "$tmp"
mkdir -p ~/.agents/skills && cp -R "$tmp/skills/omakase-pr" ~/.agents/skills/
```

更新するときも、同じコマンドを実行します。

[GitHub CLI](https://cli.github.com/)（`gh`）にログインしている必要があります。

## 使い方

先に変更をコミットして、ブランチを push しておいてください。omakase-pr はPRを作るだけで、コミットや push はしません。

最初は、何も作らずに中身だけ確かめられます。

```
omakase preview
```

種類とタイトル、ラベル、本文が表示されます。「作って」と言えばその内容のままPRを作ります。直してほしいところを伝えれば、直してから作ります。

すぐに作るなら、こうです。

```
omakase
```

必要なことは、普段の言葉で自由に付け足せます。

```
omakase fix をドラフトで
omakase レビュアーは @alice、ラベルは urgent で
```

その場で言ったことは、設定ファイルより優先されます。Claude Code では `/omakase-pr` でも呼び出せます。

## リポジトリごとの設定

`omakase init` で、このリポジトリ向けの設定を `.github/omakase-pr.yml` に保存できます。最近のブランチ名、コミットメッセージ、ラベルから設定案を作り、確認してもらってから書き出します。

```yaml
kinds:
  feature: [feat, feature]
  fix: [fix, bugfix, hotfix]
  refactor: [refactor]
  docs: [docs]
  chore: [chore, ci, build, deps]

assignees: ["@me"]
labels:
  feature: [enhancement]
  fix: [bug]
  docs: [documentation]

# 任意。コメントを外すと、すべてのPRに設定される
# projects: [Roadmap]
# milestone: v1.2
```

リポジトリにテンプレートがなければ、omakase-pr のテンプレートも `.github/PULL_REQUEST_TEMPLATE/` に書き出します。チームに合わせて直して使えます。どちらもコミットはしません。

## ライセンス

MIT
