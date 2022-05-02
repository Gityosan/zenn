---
title: "apollo-serverをfargateに乗っけるまでのメモ"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Apollo","AWS","PostgreSQL","Docker","GraphQL"]
published: false # 公開設定（falseにすると下書き）
---

:::details ライブラリ群
  - apollo-server＠^3.6.7
  - graphql＠^16.3.0
  - prisma＠^3.12.0
  - typescript対応
      - ts-node＠^10.7.0
      - ts-node-dev@^1.1.8
          - ts-nodeとnode-devの合体版のよう。ts-nodeを内包しているが、慣例でどちらも明示的に入れるみたい。
      - typescript@^4.6.3
  - linter等
      
      ```json
      {
        "eslint": "^8.13.0",
        "eslint-config-prettier": "^8.5.0",
        "prettier": "^2.6.2"
      }
      ```
      
  - typescript for eslint
      - @typescript-eslint/eslint-plugin
          - extendsにplugin:@typescript-eslint/recommendedと書いて使う
      - @typescript-eslint/parser
          - eslintでtypescriptを解析するためのライブラリ
:::