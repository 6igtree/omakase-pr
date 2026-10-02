# omakase-pr 🍣

[English](./README.md) | **日本語**

**PRテンプレートは、おまかせで。**

GitHub ではPRテンプレートを複数置けますが、PRを作るときに[選ぶ手段がありません](https://github.com/orgs/community/discussions/146146)。そのため、多くのチームはどの変更にもしっくりこないテンプレートを1つだけ使っています。

omakase-pr は Claude Code と Codex 向けのskillです。ブランチ名、コミット、差分を読んで変更の種類に合うテンプレートを選び、中身を埋めて `gh pr create` でPRを作ります。選ぶのも書くのも、板前におまかせです。

## 使うとこうなる

```
> omakase

Opened https://github.com/acme/shop/pull/412
Kind: fix, from the branch name fix/login-timeout.
```

本文は `.github/PULL_REQUEST_TEMPLATE/fix.md` に沿って、バグの内容、原因、修正、テストの方法を埋めます。チェックボックスにチェックを入れるのは、根拠があるときだけです。実行していないテストを「実行済み」と書くことはありません。

## 導入時の設定

リポジトリで初めて使うと、最近のブランチ名とコミットメッセージから、接頭辞の設定を提案します。

```yaml
# .github/omakase-pr.yml
feature: [feat, feature]
fix: [fix, bugfix, hotfix]
refactor: [refactor]
docs: [docs]
chore: [chore, ci, build, deps]
```

続けて、`.github/PULL_REQUEST_TEMPLATE/` に種類ごとのテンプレートを作ります。すでにテンプレートがあれば、その見出しをチームの書き方として全種類に引き継ぎます。どちらも、コミットする前に内容を確認できます。

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

| 言うこと | 動き |
| --- | --- |
| `omakase` | 合うテンプレートでPRを作る。初回は設定から始める |
| `omakase fix` | 種類を指定してPRを作る |
| `omakase draft` | ドラフトで作る。レビュアーやラベルの指定もできる |
| `omakase-pr preview` | push も作成もせず、種類、タイトル、本文だけを見せる |
| `omakase-pr init` | 導入時の設定をやり直す |

## 関連

- [senpai](https://github.com/6igtree/senpai)：エージェントと書くほど、エンジニアとして成長する。
- [nit](https://github.com/6igtree/nit)：エージェントが、英語で話す同僚になる。

## ライセンス

MIT
