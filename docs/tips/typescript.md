---
layout: default
title: TypeScript
nav_order: 2
parent: Tips
last_modified_date: 2024-07-23
---

# Package Manager

I use `yarn berry` package manager.

It's because it was my first package manager when I started learning typescript.

And then, when i googled about it, I found that it's better than other package manager.

So, I use it.

## Install

```bash
$ npm install -g yarn
$ yarn init
$ yarn set version berry
```

## Use

```bash
$ npm install -g yarn
$ yarn add ?
$ yarn add -D @types/?
```

- For vim lsp no error

```
$ yarn dlx @yarnpkg/sdks vim
```
