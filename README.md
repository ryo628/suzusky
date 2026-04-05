# Suzusky homepage

ポートフォリオ 兼 Top Page用のリポジトリ

## Local環境

Docker Compose を使用。

```bash
docker compose up -d
```

http://localhost:1313 でプレビュー可能（下書きも表示）。

## 新規投稿

```bash
docker compose exec hugo_app hugo new posts/my-new-post.md
```

書き終わったら `draft: true` を削除。

### Front matter 例

```yaml
---
title: "記事タイトル"
date: 2026-04-05
draft: false
categories: ["Blog"]
tags: ["Hugo", "Web"]
description: "記事の説明"
---
```

