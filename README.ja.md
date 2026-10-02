# omakase-pr 🍣

[English](./README.md) | **日本語**

**PRテンプレートは、おまかせで。**

GitHub ではPRテンプレートを複数置けますが、PRを作るときに[選ぶ手段がありません](https://github.com/orgs/community/discussions/146146)。結局、バグ修正にも新機能にも同じテンプレートを使うことになりがちです。

omakase-pr は Claude Code と Codex 向けのskillです。ブランチ名やコミット、差分から変更の種類を見分けて、合うテンプレートを選びます。中身を埋めたら、`gh pr create` でPRを作るところまで進めます。選ぶのも書くのも、板前におまかせです。

## 使うとこうなる

```
> omakase

Opened https://github.com/acme/shop/pull/412
Kind: fix, from the branch name fix/login-timeout.
```

本文は `.github/PULL_REQUEST_TEMPLATE/fix.md` の見出しに沿って埋まります。不具合の内容と原因、修正内容、テストの順です。チェックボックスにチェックを入れるのは根拠があるときだけで、実行していないテストを「実行済み」と書くことはありません。

## 導入時の設定

リポジトリで初めて使うと、最近のブランチ名とコミットメッセージから、接頭辞の設定を提案します。

```yaml
# .github/omakase-pr.yml
kinds:
  feature: [feat, feature]
  fix: [fix, bugfix, hotfix]
  refactor: [refactor]
  docs: [docs]
  chore: [chore, ci, build, deps]

# 任意。コメントを外すと、すべてのPRに設定される
# assignees: ["@me"]
# labels:
#   feature: [enhancement]
#   fix: [bug]
# projects: [Roadmap]
# milestone: v1.2
```

担当者（Assignees）、種類ごとのラベル、プロジェクト、マイルストーンは空のままです。使いたいものだけコメントを外せば、すべてのPRに設定されます。

続けて、`.github/PULL_REQUEST_TEMPLATE/` に種類ごとのテンプレートを作ります。すでにあるテンプレートの見出しは、チームの書き方として全種類に引き継がれます。言語は最近のPRやコミットに合わせるので、日本語のチームなら日本語版です。設定ファイルもテンプレートも、コミットする前に中身を確認できます。

## 種類の決め方

上から順に見て、最初に決まったものを使います。

1. 指定した種類：`omakase fix`
2. ブランチ名：`fix/login-timeout`
3. コミットメッセージ：`fix: handle expired session`
4. 差分の中身：ドキュメントしか変えていなければ docs

それでも決まらないときだけ、1回確認します。

## インストール

### Claude Code

```
/plugin marketplace add 6igtree/omakase-pr
/plugin install omakase-pr@omakase-pr
```

### Codex

```sh
git clone https://github.com/6igtree/omakase-pr.git /tmp/omakase-pr
mkdir -p ~/.agents/skills && cp -r /tmp/omakase-pr/skills/omakase-pr ~/.agents/skills/
```

[GitHub CLI](https://cli.github.com/)（`gh`）にログインしている必要があります。

## 使い方

最初は、何も作らずに中身だけ確かめられます。

```
omakase preview
```

種類とタイトル、本文が表示されるだけで、push もしません。内容がよければ、そのままPRを作ります。

```
omakase
```

必要なことは、普段の言葉で自由に付け足せます。

```
omakase fix をドラフトで
omakase レビュアーは @alice、ラベルは urgent で
```

その場で言ったことは、設定ファイルより優先されます。

`omakase init` で導入時の設定をやり直せます。Claude Code では `/omakase-pr` でも呼び出せます。

## ライセンス

MIT
