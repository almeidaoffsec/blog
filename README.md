# blog — almeidaoffsec

Blog pessoal servido em `blog.almeidaoffsec.com`. Jekyll 4 + GitHub Actions.

## Publicar um post novo

```bash
mkdir _blog/slug-do-post
# criar index.md com o front matter abaixo
# copiar imagens para o mesmo diretório
git add _blog/slug-do-post/
git commit -m "Add: post slug-do-post"
git push
# GitHub Actions builda e publica automaticamente
```

## Front matter obrigatório

```yaml
---
layout: post
title: "Título do post"
permalink: /slug-do-post/
date: YYYY-MM-DD
category: "jornada"   # jornada | ferramentas | labs | segurança
tags: ["Tag1", "Tag2"]
description: "Frase curta sobre o post."
---
```

## Configuração inicial (uma vez)

Settings → Pages → Source → **GitHub Actions**

Criar registro CNAME no Cloudflare: `blog` → `almeidaoffsec.github.io` (DNS only).
