# Suzusky homepage

ポートフォリオ 兼 Top Page用のリポジトリ

# 開発

`docker-compose`　を使う。

```
$ docker-compose up -d
$ docker container exec -it hugo_app /bin/sh
```

# 新規投稿

コマンドで雛形作成。

```
> hugo new posts/HOGE.md
```

書き終わったら `draft: true` を消す(はず)。

利用してるテーマの拡張機能？で以下のやつ追加すると良い。

```
categories: ["HOGE"]
tags: ["HOGE", "FUGA"]
description: "HOGE"
```

# 実行確認

コンテナ内で `hugo server` を叩くことで `https://localhost:1313` にホストされる。
`-D` で下書きも反映される。
細かいあれはHUGOのドキュメントで確認してください。

```
> hugo server -D --bind="0.0.0.0"
```
