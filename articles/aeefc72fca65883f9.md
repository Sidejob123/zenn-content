---
title: "Power PlatformのGenerative pagesがDataverse custom APIを呼べるようになった：エージェント設計に何が変わるか（2026年11月）"
emoji: "🔧"
type: "tech"
topics: ["PowerPlatform", "PowerApps", "Dataverse", "MicrosoftCopilot"]
published: true
---
2026年11月のPower Platform月次アップデートで、Generative pages（生成ページ）がDataverse custom APIを呼び出せるようになった（Public Preview）。これは、Power Platform上でAIエージェントをどう設計するかという観点で、設計の選択肢が変わる可能性がある変更だ。

企業のCopilot・Power Platform展開を担当する立場から、今回の変更が設計上何を変えるか、自分の理解を整理した。

出典：https://www.microsoft.com/en-us/power-platform/blog/power-apps/whats-new-in-power-platform-october-2026-feature-update/（2026-11-11）

## 前提：Generative pagesとDataverse custom APIとは何か

まずGenerative pagesを知らない読者向けに整理する。Power Appsには従来、makerが画面を設計するcanvas appsとmodel-driven appsがあった。Generative pagesは、AIがデータモデルを読み取って画面を自動生成するモードで、まだPreviewの機能だ。

Dataverse custom APIは、Power Platformのデータ基盤（Dataverse）にカスタムのサーバー側ロジックを実装する仕組みだ。HTTPを通じて呼び出せる。承認処理、集計、外部サービスへの通知など、テーブルへの単純なCRUD以上の処理を書ける。

従来のGenerative pagesは、主にDataverseテーブルへのデータの表示と更新しかできなかった。今回の変更で、custom APIも呼び出せるようになった。

## 設計上の変化：「AIが生成した画面」からビジネスロジックを呼べる

これまでのGenerative pagesは「データを見て・変える」だけだった。今後は「データを見て・変えて・何かをトリガーする」設計が可能になる。

たとえばこういう設計が考えられる：

- 受注データを一覧する生成ページから、ステータス変更と同時に承認フローのAPIを呼び出す
- 顧客情報を編集した際に、変更通知をTeamsに送るAPIをトリガーする
- フォーム入力の内容を別のDataverseテーブルに記録するロジックを呼び出す

いずれも従来はmodel-driven appsでプラグインを書くか、Power Automateのフローで処理するかという方法が一般的だった。生成ページからcustom APIを直接呼べるようになると、「AIが画面を作って、そこからビジネスロジックも動かす」という一体型の設計がしやすくなる。

## Work IQとの組み合わせで何ができるか

同じアップデートで、Work IQ（Power Appsのmodel-driven appsのデータを自然言語で操作する機能）もPublic Previewになった。Work IQはDataverseのデータへの自然言語アクセスを提供し、Generative pagesはそのデータをAI生成の画面で扱う。

MicrosoftがPower Platform Community Conferenceで「Work, Reimagined」というセッションで示した方向性は、「アプリを介さずにビジネスデータをAIで扱える世界」だと読める。Work IQが「会話でデータを触る入口」であれば、Generative pagesは「AIが構築した画面でデータを触る入口」だ。

この組み合わせが企業展開でどう使えるかはまだ実機で確認していないが、設計の観点から整理すると：

- **Work IQ**：アプリを開かずに日常的なデータ確認や更新に向く。「今日何件の問い合わせが来た？」「この案件のステータスを更新して」といった日常的な操作
- **Generative pages + custom API**：複雑なビジネスロジックを含む画面操作に向く。承認・通知・集計を含むプロセスをAI生成画面から動かす

ただしどちらもPreviewであり、GAのタイミングや機能の変更が入る可能性がある。今すぐ本番設計に組み込むというよりは、概念実証（PoC）で試しながら判断するフェーズだ。

## 既存設計への影響：Power Automateとの棲み分けが変わるかもしれない

ここが一番気になった点だ。現在、企業内のPower Platform展開では「データ操作はPower Apps、自動化・承認・通知はPower Automate」という棲み分けが一般的だ。

今回のGenerative pages + Dataverse custom APIが普及すると、「Power Appsから直接ビジネスロジックを呼べる」設計が増える可能性がある。すると「これはPower Automateで作るべきか、custom APIで書くべきか」という判断が必要な場面が増えてくる。

自分が設計判断で使っている基準を暫定的に整理すると：

- **Power Automateのフロー**：外部サービスとの連携（Teams・メール・SharePoint）が多い場合、条件分岐が複雑な場合、フローの実行履歴を運用担当者が確認したい場合
- **Dataverse custom API**：Power Platform内部で完結する処理、高頻度の呼び出しが必要な場合（Power Automateのフロー実行制限に引っかかりたくない場合）、低レイテンシが求められる場合

ただしこれはあくまで暫定で、Generative pagesのGAがどうなるかによって変わる。PowerPlatform運用の設計判断は、機能がGAになってからが本番だと思っている。

## 展開中の組織が今確認すること

現在Power Platformを展開中の組織が、今月のアップデートを受けて確認しておくといい点を絞ると：

1. **Work IQのPreviewを有効にするか**：tenant adminの設定でPreviewの有効化が必要になる見込み。セキュリティポリシー上、preview機能を本番テナントに有効化するかどうかを確認する
2. **Generative pagesの利用方針**：既にGenerative pagesを試している場合、custom API呼び出しの可否が変わった。試していない場合は今後の方針を決める良いタイミング
3. **SharePointカスタムフォームのテンプレート変更**：新しくPower Appsフォームを作成すると、以前と見た目が変わる（レスポンシブマルチカラム）。ヘルプデスクへの問い合わせが増える可能性があるため、事前に告知を検討する

## この検証の限界

今回の記事は2026-11-11公開の公式ブログを元にしており、実機確認はしていない。機能の挙動・制限・課金への影響については、公式のLearn moreリンクと実機確認が必要だ。

Generative pagesとDataverse custom APIはどちらもPreviewで、機能が変更・廃止される可能性がある。企業の本番設計に組み込む際は、GAのタイミングを確認してから進めることを推奨する。

自分自身もまだ実機で試していない部分が多い。試した結果は別の記事で書く予定。もし既に試している人がいれば、コメントで教えてもらえると参考になる。


CopilotやPower Platform展開の相談を、ココナラで承っています（「Copilot Studio導入」「Power Automate 点検」で検索）。