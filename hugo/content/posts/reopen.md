---
title: "ホームページリニューアル"
date: 2020-07-25T23:52:36Z
categories: ["Blog"]
tags: ["blog", "hobby"]
description: "suzusky.netのリニューアルをした話"
---

# これはなに

今までポートフォリオサイトとして作成していたサイトをリニューアルしました。

今まではお手製のHTML/CSSだったので、面倒になってきたためHugoへと移行しました。

# やったこと

- 開発用のDockerfileとか作る
- Hugoへの移行
- GitHub Actionsの設定

# Docker開発環境の作成

公式によるイメージが落ちていないようなので `golang:1.14.6-alpine3.11` を使った。

これは新しめのバージョンでモジュールを使う場合は、Go言語を含んだ方がいい(要出典)という情報を見たので Alpine Linuxのイメージを採用した。

```Dockerfile
FROM golang:1.14.6-alpine3.11 AS hugo_builder
```

イメージにHugo に必要なものをゴニョゴニョ持ってこさせる。
(Hugoの依存等わからなかったので半ばコピペだが…)

```Dockerfile
WORKDIR /hugo
SHELL ["/bin/ash", "-eo", "pipefail", "-c"]
RUN apk add --no-cache --virtual .build-deps wget && \
    apk add --no-cache \
    git \
    ca-certificates \
    libc6-compat \
    libstdc++ && \
    wget --quiet "${HUGO_URL}" && \
    wget --quiet "${HUGO_CHECKSUM_URL}" && \
    grep "${HUGO_NAME}.tar.gz" "./hugo_${HUGO_VERSION}_checksums.txt" | sha256sum -c - && \
    tar -zxvf "${HUGO_NAME}.tar.gz" && \
    mv ./hugo /usr/bin/hugo && \
    apk del .build-deps && \
    rm -rf /hugo
```

この `Dockerfile` を指定して、 `port: 1313` を掴んで、Hugo用の `./hugo` をマウントさせるように `docker-compose.yml` を作成した。

雰囲気で普段から Dockerを使ってるので、 `image:` 指定しないやり方が新鮮に感じた。

```yaml
version: '3'
services:
  hugo_app:
    container_name: hugo_app
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 1313:1313
    tty: true
    volumes:
      - ./hugo:/hugo
```

# Hugoへの移行

これはめっちゃ楽だった。

```cmd
> hugo new site
```

これで雛形が自動生成される。

その後お好きなテーマを [ここ](https://themes.gohugo.io/) から探す。

これを `theme/` いかに配置すればいいのだが、インターネットに転がってるように単純にgit cloneするとGitHub Actionsの時にこけるからちゃんとサブモジュールにしましょう。

```
hugo/theme > git submodule add YOUR_THEME.git
```

この後、Hugoの設定ファイルである `config.toml` でテーマを変更する。

ついでに言語やベースURLも変更すると良いだろう

```yaml
baseURL = "http://hoge.fuga/"
languageCode = "ja-ja"
title = "TITLE_NAME"
theme = "THEMA_NAME"
```

# GitHub Actionsの設定 CI編

これは `.github/workflow` 直下に設定yaml作ればいい感じにしてくれる。

これもイメージが必要なのだが、面倒なので人様が作成してるのを拝借する。

今回は、[peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo)を使った。

ちなみに GitHub Actionのyamlもサンプルが公開されているのでそれを用いれば良い。

ただ、ビルドオプションに `--minify` がついているがこのままだとテーマによっては怒られが発生するので、自分は外した。

# GitHub Actionsの設定 Deploy編

自分はさくらインターネットのレンタルサーバを利用しているため、masterへのcommit(本当はPRのmergeのみにブロックしたい)をトリガーに自動でデプロイしてくれるように設定を行う。

と言っても `hugo` コマンドで `public/` 直下に静的ファイルが生成されるのでそれらを全部サーバーへとrsyncで送るだけで済む。
便利。令和最高。(GitHub Actionsは平成から存在するが)

が問題なのは sshを通したり、デプロイ用のユーザーが見えないように保護しないといけないことである。

がこれも簡単で、リポジトリ設定の中に `Secrets` というのがあり、定数定義ができる。

これに秘密鍵から全部入れ込んで仕舞えば問題ない。

```yaml
      - name: Generate ssh key
        run: echo "$SSH_PRIVATE_KEY" > key && chmod 600 key
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          
      - name: Deploy
        run: rsync -rlptgoD --delete --exclude ".git/" -e "ssh -i ./key -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null " hugo/public/ $SSH_USER@$SSH_HOST:$DEPLOY_PATH
        env:
          SSH_USER: ${{ secrets.SSH_USER }}
          DEPLOY_PATH: ${{ secrets.DEPLOY_PATH }}
          SSH_HOST: ${{ secrets.SSH_HOST }}
```

`secrets.HOGE` で設定してある文字列を参照できる。これを `env:` で再定義し、runコマンドで参照する。

また自分は Hugo階層を1つ組んだのでちゃんとパスは考えないといけない。(1敗)

恐らく全てリポジトリのカレントで実行する感じになってる。

# まとめ

数時間で大体できた。
インターネットは偉大。

自分で書けないCSSを触らないで済むのが嬉しい。
うまくmasterブランチの保護設定ができたら、public リポジトリに変更しないなぁ…
