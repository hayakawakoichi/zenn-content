---
title: "Node.js 環境変数管理の改善: dotenv から --env-file オプションへ変更する際の注意点"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nodejs", "環境変数", "dotenv"]
published: true
---

## はじめに

Node.js のバージョンアップに伴う環境変数の取り扱いについて、実際に遭遇した課題と解決策を共有します。

## 背景

私が所属するプロジェクトでは、最近 Node.js を最新バージョン（v22）にアップデートしました。
この際、これまで環境変数の読み込みに使用していた `dotenv` パッケージ（および `dotenv -e` コマンド）を、Node.js 組み込みの `--env-file` オプションに変更しました。
`--env-file` は、Node.js 20.6.0 以降で利用可能なオプションで、これにより外部ライブラリを使用せずに .env ファイルから環境変数を直接読み込めるようになりました。

変更前後のコードの例を以下に示します。

```diff
{
    "scripts": {
-       "dev": "dotenv -e .env -- node src/index.js",
+       "dev": "node --env-file=.env src/index.js",
    }
}
```

## 直面した課題

`--env-file` オプションへの移行自体はシンプルですが、実運用環境にデプロイした際に問題が発生しました。
`--env-file` オプションは、**指定したファイルが存在しない場合にエラーを発生させる**という仕様になっています。

環境変数の管理方法として、下記のような構成を取る場合、`--env-file` オプションを指定するとクラウド環境でのビルド時にエラーとなってしまいます。

1. 開発者のローカル環境
   - `.env` ファイルを作成し、その中で環境変数を管理

2. 本番環境（Vercel, Cloudflare Pages, etc.）
   - `.env` ファイルは使用せず、プラットフォームの管理画面上で環境変数を設定
   - `.env` ファイルは `.vercelignore` や `.gitignore` に含めている

## 解決策: --env-file-if-exist オプションの活用

Node.js には `--env-file-if-exist` というオプションも用意されています。
このオプションは名前が示す通り、**指定したファイルが存在しない場合でもエラーを発生させず、処理を続行する**という仕様になっています。

```json
// package.json の例
{
  "scripts": {
    "dev": "node --env-file-if-exist=.env src/index.js"
  }
}
```

このオプションを使用すると、

1. ローカル環境では `.env` ファイルから環境変数を読み込む
2. Vercel などのクラウド環境では、ファイルがなくてもエラーにならず、プラットフォーム側で設定された環境変数を使用する

という動作が可能になります。(`dotenv -e` オプションと同じ挙動になります)

## まとめ

Node.js の `--env-file` オプションは .env ファイルが存在しないとエラーになるため、ローカルとクラウドの環境を考慮する場合は `--env-file-if-exist` を利用すると、良さそうです。

## 参考

> In case you want to optionally read from a .env file, it's possible to avoid throwing an error if the file is missing using the --env-file-if-exists flag.

https://nodejs.org/en/learn/command-line/how-to-read-environment-variables-from-nodejs
