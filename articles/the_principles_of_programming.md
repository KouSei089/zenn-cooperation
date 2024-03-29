---
title: "The Principles of Programming 要約"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# プリンシプルオブプログラミング 要約

## 第1章 前提
### プログラミングに銀の弾丸は存在しない
- ソフトウェアの本質には困難性がある.（具体的には以下）
  - 複雑性: ソフトウェアの規模拡大と共に依存関係が非線形に増大する
  - 同調性: ソフトウェアを実世界に寄せる.実世界は複雑である
  - 可変性: ソフトウェアそのものが変化していく
  - 不可視性: 全てのプロセスや設計を可視化することができない
> 結論: 偶有的な部分は容易に変更できるので改善する

### コードは設計書である
- 上流工程で行うことが設計ではなく、結局最終的なアウトプットはコードである
- 「詳細設計」「プログラミング」「テスト」「デバッグ」これら全てが設計である
- 改善対象もコードである
- 上記のことから分業的になりすぎるのは、本質的に良い設計でなくなる可能性がある

### コードは必ず変更される
- リーダブルなコードを書く

## 第2章 原則

### KISS
- `Keep It Short and Simple`(簡潔かつ単純にしておけ)

### DRY
- `Don't Repeat Yourself.`(繰り返すな)

### YAGNI
- `You Aren't Going to Need It.`(それはきっと必要にならない)

### PIE
- `Programming Intently and Expressively.`(意図を表現してプログラミングせよ)
- コードを書くときは書きやすさよりの読みやすさを重要視する

### SLAP
- `Single Level of Abstraction Principle.`(抽象化レベルの統一)
- コードの抽象レベルを上から下に向かって統一すると書籍の目次のようになり、見やすくなる

### OCP
- `Open-Closed Principle`(オープン・クローズドの原則)

### 名前重要
- `Naming is important.`