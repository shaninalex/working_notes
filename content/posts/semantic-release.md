---
title: "Semantic Release"
date: 2023-12-10T14:03:17+02:00
draft: false
tags: ['git', 'administration']
---

I was messing around with semantic releases for a while and came to the solution that I will replace in this article. What a twist! :)

Basically, my solution was this:

```yaml
name: Create New Tag

on:
  push:
    branches:
      - master

jobs:
  tag:
    runs-on: ubuntu-22.04
    permissions:
      contents: write
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: '0'
    - name: Bump version and push tag
      uses: anothrNick/github-tag-action@1.66.0
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        WITH_V: true
        BRANCH_HISTORY: full
        INITIAL_VERSION: 1.0.0
```

And it's working fine. But for some reason, and it's probably me, all tags have only "0" as the patch number. Like: 1.12.0, 1.13.0, 1.14.0... Whatever I name my PRs and commits - it's always zero. This is strange, but we can live with it.

### Replacement

Surfing around GitHub, I started noticing that many repositories have this file - .releaserc.json. Interesting. A little bit more googling and reading the documentation, I came to another solution:

.releaserc.json
```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    [
      "@semantic-release/github",
      {
        "successComment": false
      }
    ]
  ]
}
```

workflow.yml
```yml
name: ChatApp Workflow

on:
  push:
    branches:
      - master

jobs:
  release:
    name: release
    runs-on: ubuntu-20.04
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: 18
      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: npx semantic-release
```

> Note! This action requires `permissions: contents: write` to be able to push new tags.

This action and `.releaserc.json` automatically create a new tag and push a new release. The previous solution required manually creating a new release. Also, patch numbers in the release version appear correctly again :)
