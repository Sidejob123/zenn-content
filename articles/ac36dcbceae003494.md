---
title: "Claude APIをSaaSに組み込む際のコスト設計 — 本番運用で気づいたこと"
emoji: "💰"
type: "tech"
topics: ["claude", "SaaS", "API", "コスト設計", "個人開発"]
published: true
---
個人でSaaS（hukugyou.tech）を開発・運営しながら、Claude APIを本番環境で使ってきた。設計段階で考慮しておきたかったことをまとめる。

## コスト設計の前提：「想定外の使い方」は必ず起きる

SaaSにLLM APIを組み込む場合、「ユーザーが自分と同じような使い方をする」という前提は崩れやすい。想定より長いプロンプトを入れてくる、ループして何十回もリクエストする、など。コスト設計は「平均的な使い方」ではなく「外れ値の使い方」を基準に考えると、後で驚かずに済む。

## 入力トークンのコントロール

Claude APIはinput/outputそれぞれ課金される。input側を削れる余地は大きい。

**効く施策**

- システムプロンプトをキャッシュ対象にする（cache_control）
  - 変わらない部分（役割定義、ルール等）をプロンプトの先頭に置く
  - ヒット時はinput料金が大幅に下がる（Anthropicの場合90%削減）
- ユーザー入力の前処理：不要な空白・改行・重複を除去してからAPIに渡す
- コンテキストウィンドウに詰め込みすぎない：関連性の低い履歴は切り捨てる

**実装例（Node.js）**

```javascript
const response = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  system: [
    {
      type: "text",
      text: SYSTEM_PROMPT_STATIC,
      cache_control: { type: "ephemeral" }
    }
  ],
  messages: conversationHistory
});
```

## レート制限とリトライ設計

個人SaaSの場合、突発的なトラフィックで429（レート制限）を受けることがある。リトライ設計を入れていないと、ユーザーがエラー画面を見て離脱する。

```javascript
async function callWithRetry(params, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await client.messages.create(params);
    } catch (err) {
      if (err.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(r => setTimeout(r, delay));
      } else throw err;
    }
  }
}
```

## 利用上限の設計

SaaSにAPIを組み込む場合、ユーザー1人あたりの上限（1日N回、1リクエストMトークンまで）を設けないと、少数のヘビーユーザーがコストの大半を占める状態になりやすい。上限設計は「ユーザーを制限する」という視点より、「サービスの持続可能性を保つ」という視点で設計する方が、UIへの組み込み方も変わってくる。

## モニタリング

Anthropicのダッシュボードはトークン使用量を確認できるが、「どのユーザーが何を呼んでいるか」はアプリ側でログを持たないとわからない。最低限のログ設計：リクエスト時刻、ユーザーID、input/outputトークン数、モデル名。これがあるだけでコスト異常の原因調査がかなり楽になる。

Claude APIのコスト設計で気をつけていることはnoteにも書いている：[Claude APIをSaaSに組み込む時に考えたコスト設計](https://note.com/light_tern636/n/n387ce9d097e2)