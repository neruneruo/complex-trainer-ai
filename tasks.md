# 実装計画: ファザコン・マザコン養成AI

## 概要

ハッカソン向けに優先度の高い機能から段階的に実装する。まずプロジェクト基盤とコアロジック（スコア算出・メッセージ生成）を構築し、次にWeb UI・LINE連携・スケジュール管理・朝のルーティン・学習計画・日記生成の順で機能を追加する。各ステップでプロパティベーステスト（fast-check）とユニットテスト（Vitest）を組み合わせて品質を確保する。

## タスク

- [ ] 1. プロジェクト基盤セットアップ
  - [ ] 1.1 Next.js (App Router) + TypeScript プロジェクト初期化
    - `npx create-next-app@latest` でApp Router + TypeScript + Tailwind CSSのプロジェクトを作成
    - Framer Motion、fast-check、Vitest、@line/bot-sdk、@aws-sdk/client-dynamodb、@aws-sdk/client-bedrock-runtime、@aws-sdk/lib-dynamodb をインストール
    - `vitest.config.ts` を作成し、テスト環境を設定
    - _Requirements: 全体_

  - [ ] 1.2 DynamoDB シングルテーブル設計のデータアクセス層を実装
    - `src/lib/dynamodb.ts` にDynamoDBクライアントとヘルパー関数を作成
    - `src/types/database.ts` に全エンティティの型定義（UserProfile, UserSettings, LineLinkage, LinkCode, FazaconScore, DiaryRecord, ScheduleRecord, StudyPlanRecord, RewardPoints）を作成
    - PK/SKパターンとGSI（GSI1, GSI2）の定義を含める
    - _Requirements: 全体（データモデル設計に基づく）_

  - [ ] 1.3 Amazon Bedrock クライアントを実装
    - `src/lib/bedrock.ts` にBedrock Converse API呼び出しのラッパーを作成
    - タイムアウト時のテンプレートメッセージフォールバック、スロットリング時の指数バックオフリトライ（最大3回）を実装
    - _Requirements: 全体（AI基盤）_

  - [ ] 1.4 LINE Messaging API クライアントを実装
    - `src/lib/line-client.ts` にLineClientを作成（署名検証、pushMessage、replyMessage、getProfile）
    - Webhook署名検証ロジック（HMAC-SHA256）を実装
    - Push API送信失敗時の指数バックオフリトライ（最大3回）を実装
    - _Requirements: 2.7, 9.6, 9.7_

- [ ] 2. ファザコン度診断（コアロジック + API + UI）
  - [ ] 2.1 ScoreCalculator を実装
    - `src/services/score-calculator.ts` に `calculateFazaconScore`、`getTitleByScore`、`generateShareText` を実装
    - 10問の診断質問データを `src/data/diagnosis-questions.ts` に定義
    - スコアは0〜100の範囲にクランプ、称号はスコア範囲ごとに定義
    - _Requirements: 1.1, 1.2, 1.3, 1.4_

  - [ ]* 2.2 ScoreCalculator のプロパティベーステストを作成
    - **Property 1: ファザコン度スコアの範囲不変条件** — 任意の回答配列に対してスコアが0〜100の整数であることを検証
    - **Validates: Requirements 1.2, 1.5**

  - [ ]* 2.3 ScoreCalculator のプロパティベーステストを作成（称号・シェアテキスト）
    - **Property 2: スコアから称号・シェアテキスト生成の完全性** — 任意の有効スコアに対して空でない称号とシェアテキストが返ることを検証
    - **Validates: Requirements 1.3, 1.4**

  - [ ]* 2.4 ScoreCalculator のユニットテストを作成
    - 具体的なスコア算出シナリオ、境界値（全問最低・最高回答）、空配列のエッジケースをテスト
    - _Requirements: 1.2, 1.3, 1.4_

  - [ ] 2.5 診断 API Routes を実装
    - `src/app/api/diagnosis/questions/route.ts` — GET: 診断質問一覧取得
    - `src/app/api/diagnosis/result/route.ts` — POST: 回答を受け取りスコア算出、Bedrockで親コメント生成、DynamoDBに保存
    - _Requirements: 1.1, 1.2, 1.3, 1.4_

  - [ ] 2.6 診断ページ UI を実装
    - `src/app/diagnosis/page.tsx` — 10問の質問を順番に表示するステッパーUI
    - 回答完了後にスコア・称号・親コメント・SNSシェアボタンを表示
    - Framer Motionでページ遷移アニメーションを追加
    - _Requirements: 1.1, 1.2, 1.3, 1.4_

- [ ] 3. 親キャラクターUI・演出
  - [ ] 3.1 親キャラクターアバターコンポーネントを実装
    - `src/components/ParentAvatar.tsx` — 父・母のアバター表示、感情に応じた表情変化（worry, happy, angry, sad, proud）
    - Framer Motionで拗ねる・喜ぶアニメーションを実装
    - 画面上部に常時表示されるレイアウトを作成
    - _Requirements: 2.1, 2.2, 2.3, 2.4_

  - [ ]* 3.2 感情→表情マッピングのプロパティベーステストを作成
    - **Property 3: 感情から表情へのマッピング完全性** — 任意の有効な感情タイプに対して有効な表情タイプが返ることを検証
    - **Validates: Requirements 2.2**

  - [ ] 3.3 MessageGenerator を実装
    - `src/services/message-generator.ts` — Bedrockを使用してキャラクター口調のおせっかいメッセージを生成
    - 父（「〜だぞ」「〜しなさい」）・母（「〜でしょ？」「〜なさいね」）の語尾パターンをプロンプトに含める
    - カテゴリ別（morning_alarm, weather_advice, umbrella_reminder, praise, worry, scold等）のプロンプトテンプレートを定義
    - _Requirements: 2.5_

  - [ ] 3.4 深夜警告・長時間未使用判定ロジックを実装
    - `src/services/time-judgment.ts` に `isNightTime`（23:00〜5:59判定）、`isInactive`（閾値ベースの未使用判定）を実装
    - _Requirements: 2.6, 2.7_

  - [ ]* 3.5 時刻判定のプロパティベーステストを作成
    - **Property 4: 深夜時間帯判定** — 任意の時刻に対して23:00〜5:59の場合にのみtrueを返すことを検証
    - **Validates: Requirements 2.6, 7.3**

  - [ ]* 3.6 長時間未使用判定のプロパティベーステストを作成
    - **Property 5: 長時間未使用判定** — 任意の最終利用時刻と現在時刻のペアに対して閾値超過時のみtrueを返すことを検証
    - **Validates: Requirements 2.7**

- [ ] 4. チェックポイント - 基盤とコアロジックの確認
  - すべてのテストが通ることを確認し、ユーザーに質問があれば確認する。

- [ ] 5. ユーザー設定・LINE連携
  - [ ] 5.1 ユーザー設定 API を実装
    - `src/app/api/settings/route.ts` — GET/PUT: ユーザー設定の取得・更新
    - `src/app/api/settings/parent-type/route.ts` — PUT: 親キャラクター変更
    - `src/app/api/line/link/route.ts` — POST: 6桁連携コード生成（TTL 5分）、DynamoDBに保存
    - _Requirements: 2.1, 全体（設定管理）_

  - [ ] 5.2 LINE Webhook エンドポイントを実装
    - `src/app/api/line/webhook/route.ts` — POST: Webhook受信、署名検証、イベント分岐処理
    - follow イベント: ウェルカムメッセージ送信
    - message イベント: 連携コード処理 → スケジュール登録 → 親AIとの会話の優先順位で分岐
    - _Requirements: 9.6, 9.7, 9.8_

  - [ ] 5.3 LINEスケジュールパーサーを実装
    - `src/services/schedule-parser.ts` に `parseScheduleFromText` を実装
    - Bedrockを使用して自然言語テキスト（例：「明日14時に歯医者」）から予定名・日時を抽出
    - パース成功時はスケジュール登録＋確認メッセージ返信、失敗時は聞き返しメッセージ返信
    - _Requirements: 9.6, 9.7, 9.8_

  - [ ]* 5.4 LINEスケジュールパース結果のプロパティベーステストを作成
    - **Property 20: LINEスケジュールパース結果の構造的正しさ** — 任意のパース成功結果に対してtitle・date・startTime・confidenceが有効な形式であることを検証
    - **Validates: Requirements 9.6, 9.7**

  - [ ] 5.5 設定ページ UI を実装
    - `src/app/settings/page.tsx` — 親キャラクター選択（父/母）、起床時刻設定、自宅位置設定、機能ON/OFF、LINE連携コード発行ボタン
    - _Requirements: 2.1, 全体（設定管理）_

  - [ ]* 5.6 LINE Webhook署名検証のユニットテストを作成
    - 正しい署名・不正な署名・空ボディのケースをテスト
    - _Requirements: セキュリティ_

- [ ] 6. おせっかいスケジュール管理
  - [ ] 6.1 スケジュール CRUD API を実装
    - `src/app/api/schedule/route.ts` — GET/POST: スケジュール一覧取得・登録
    - `src/app/api/schedule/[id]/route.ts` — PUT/DELETE: スケジュール更新・削除
    - GSI2を使用した日付ベースのスケジュール検索を実装
    - _Requirements: 9.1, 9.3, 9.5_

  - [ ] 6.2 スケジュールリマインドロジックを実装
    - `src/services/schedule-reminder.ts` にリマインド時刻算出（2時間前・1時間前・30分前）、緊急リマインド判定（15分前で未応答）を実装
    - 予定間の移動時間考慮ロジックを実装
    - _Requirements: 9.2, 9.3, 9.4_

  - [ ]* 6.3 スケジュールリマインド時刻算出のプロパティベーステストを作成
    - **Property 18: スケジュールリマインド時刻算出** — 任意の予定開始時刻に対して3つのリマインド時刻が時系列昇順であることを検証
    - **Validates: Requirements 9.2**

  - [ ]* 6.4 緊急リマインド判定のプロパティベーステストを作成
    - **Property 19: 緊急リマインド判定** — 任意の予定開始時刻・現在時刻・応答状態に対して15分前未応答時のみtrueを返すことを検証
    - **Validates: Requirements 9.4**

  - [ ] 6.5 スケジュールページ UI を実装
    - `src/app/schedule/page.tsx` — カレンダー表示、予定登録フォーム、リマインド状態表示
    - _Requirements: 9.1, 9.5_

- [ ] 7. チェックポイント - LINE連携とスケジュール管理の確認
  - すべてのテストが通ることを確認し、ユーザーに質問があれば確認する。

- [ ] 8. 朝のおせっかいルーティン
  - [ ] 8.1 天気サービスを実装
    - `src/services/weather-service.ts` に OpenWeatherMap API呼び出し（`lang=ja`）を実装
    - 天気情報キャッシュ（API失敗時のフォールバック用）を実装
    - _Requirements: 4.3, 4.4, 7.1_

  - [ ] 8.2 服装アドバイス・傘リマインドロジックを実装
    - `src/services/morning-routine.ts` に `getClothingAdvice`（気温→服装アドバイス）、`shouldRemindUmbrella`（降水確率30%以上判定）を実装
    - _Requirements: 4.3, 4.4_

  - [ ]* 8.3 気温→服装アドバイスのプロパティベーステストを作成
    - **Property 9: 気温から服装アドバイスの一貫性** — 任意の気温値に対して空でないアドバイスが返り、低温ほど暖かい服装を推奨することを検証
    - **Validates: Requirements 4.3**

  - [ ]* 8.4 降水確率→傘リマインドのプロパティベーステストを作成
    - **Property 10: 降水確率による傘リマインド判定** — 任意の降水確率に対して30以上の場合にのみtrueを返すことを検証
    - **Validates: Requirements 4.4**

  - [ ] 8.5 アラームエスカレーション・出発時刻算出ロジックを実装
    - `src/services/morning-routine.ts` に `getAlarmLevel`（未応答回数→メッセージ強度）、`calculateDepartureTime`（予定開始時刻-移動時間-バッファ）、`isUrgentDeparture`（出発10分前判定）を実装
    - 乗換案内はモック実装（`src/services/transit-mock.ts`）
    - _Requirements: 4.1, 4.2, 4.5, 4.6, 4.7_

  - [ ]* 8.6 アラームエスカレーションのプロパティベーステストを作成
    - **Property 8: アラームエスカレーション単調増加** — 任意の未応答回数nに対してメッセージ強度が単調非減少であることを検証
    - **Validates: Requirements 4.2**

  - [ ]* 8.7 出発時刻算出・緊急判定のプロパティベーステストを作成
    - **Property 11: 出発時刻算出と緊急判定** — 任意の予定開始時刻と移動時間に対して出発時刻が正しく算出され、10分前超過時のみ緊急フラグがtrueになることを検証
    - **Validates: Requirements 4.5, 4.6**

  - [ ] 8.8 朝のルーティン用 Lambda 関数を実装
    - `lambda/morning-routine/index.ts` — EventBridge Schedulerからトリガーされ、対象ユーザーの起床時刻に合わせてアラーム・天気・服装・傘・出発時刻のおせっかいメッセージをLINE Push APIで送信
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7_

- [ ] 9. 過保護心配エンジン・褒めて伸ばす依存強化
  - [ ] 9.1 心配エンジンロジックを実装
    - `src/services/worry-engine.ts` に食事リマインド判定（`shouldRemindMeal`）、気象警報処理、体調不良時の病院検索ロジックを実装
    - _Requirements: 7.1, 7.2, 7.3, 7.4_

  - [ ]* 9.2 食事リマインド判定のプロパティベーステストを作成
    - **Property 16: 食事リマインド判定** — 任意の現在時刻と食事記録状態に対して食事時間超過かつ未記録時のみtrueを返すことを検証
    - **Validates: Requirements 7.2**

  - [ ] 9.3 ご褒美ポイントシステムを実装
    - `src/services/reward-points.ts` にポイント付与・累計計算・閾値判定ロジックを実装
    - DynamoDBのRewardPointsエンティティへの読み書きを実装
    - _Requirements: 8.1, 8.2, 8.3, 8.4_

  - [ ]* 9.4 ご褒美ポイントのプロパティベーステストを作成
    - **Property 17: ご褒美ポイント累計の正確性と閾値判定** — 任意のポイント付与イベント列に対して累計が合計と一致し、閾値以上時のみご褒美フラグがtrueになることを検証
    - **Validates: Requirements 8.3, 8.4**

  - [ ] 9.5 心配・褒め用 Lambda 関数を実装
    - `lambda/worry-and-praise/index.ts` — EventBridge Schedulerからトリガーされ、食事リマインド・深夜警告・スケジュールリマインド・褒めメッセージをLINE Push APIで送信
    - _Requirements: 7.1, 7.2, 7.3, 8.1, 8.2_

- [ ] 10. チェックポイント - 朝のルーティンと心配エンジンの確認
  - すべてのテストが通ることを確認し、ユーザーに質問があれば確認する。

- [ ] 11. 試験・学習計画
  - [ ] 11.1 StudyPlanner を実装
    - `src/services/study-planner.ts` に `generateStudyPlan`（試験情報→日別学習計画生成）、`adjustPlan`（サボり日の再調整）、`getConsecutiveSkips`（連続サボり日数算出）を実装
    - Bedrockを使用して学習内容の分割・配分を生成
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6_

  - [ ]* 11.2 学習計画生成の構造的完全性のプロパティベーステストを作成
    - **Property 12: 学習計画生成の構造的完全性** — 任意の有効な試験情報に対して全日に計画が存在し、各日にcontent・targetMinutes・breakIntervalが含まれることを検証
    - **Validates: Requirements 5.1, 5.2**

  - [ ]* 11.3 学習計画再調整のプロパティベーステストを作成
    - **Property 13: 学習計画再調整の完全性** — 任意の学習計画とサボり日に対して再調整後も全学習内容がカバーされることを検証
    - **Validates: Requirements 5.5**

  - [ ]* 11.4 連続サボり日数判定のプロパティベーステストを作成
    - **Property 14: 連続サボり日数判定** — 任意のサボり履歴に対して連続サボり日数が正しく算出され、3日以上で感情的メッセージフラグがtrueになることを検証
    - **Validates: Requirements 5.6**

  - [ ] 11.5 学習計画 API Routes を実装
    - `src/app/api/study-plan/route.ts` — GET/POST: 学習計画取得・作成
    - `src/app/api/study-plan/[id]/complete/route.ts` — POST: 学習完了報告（ポイント付与含む）
    - _Requirements: 5.1, 5.4_

  - [ ] 11.6 学習計画ページ UI を実装
    - `src/app/study-plan/page.tsx` — 試験情報登録フォーム、日別学習計画表示、完了/サボり報告ボタン、進捗バー
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6_

- [ ] 12. 行動日記の自動生成
  - [ ] 12.1 DiaryGenerator を実装
    - `src/services/diary-generator.ts` に `generateDiary`（行動ログ→親目線日記生成）、`generateWeeklySummary`（週間サマリー生成）を実装
    - Bedrockを使用して「今日のうちの子」形式の日記と親の感想を生成
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.6_

  - [ ] 12.2 日記一覧ソートロジックを実装
    - `src/services/diary-service.ts` に日記一覧取得（日付降順ソート）、行動ログ記録ロジックを実装
    - _Requirements: 6.5_

  - [ ]* 12.3 日記一覧ソートのプロパティベーステストを作成
    - **Property 15: 日記一覧のソート順** — 任意の日記レコード群に対して一覧が日付降順でソートされていることを検証
    - **Validates: Requirements 6.5**

  - [ ] 12.4 日記 API Routes を実装
    - `src/app/api/diary/route.ts` — GET: 日記一覧取得
    - `src/app/api/diary/[date]/route.ts` — GET: 特定日の日記取得
    - `src/app/api/diary/weekly/route.ts` — GET: 週間サマリー取得
    - _Requirements: 6.2, 6.3, 6.5, 6.6_

  - [ ] 12.5 日記ページ UI を実装
    - `src/app/diary/page.tsx` — 日記一覧（時系列表示）、個別日記表示、週間サマリー表示
    - 親キャラクターのコメント付きカード形式で表示
    - _Requirements: 6.3, 6.4, 6.5, 6.6_

  - [ ] 12.6 日記自動生成用 Lambda 関数を実装
    - `lambda/diary-generator/index.ts` — EventBridge Schedulerから毎日22:00にトリガーされ、行動ログから日記を自動生成してDynamoDBに保存
    - _Requirements: 6.2_

- [ ] 13. 迷子防止・ナビゲーション
  - [ ] 13.1 RouteService を実装
    - `src/services/route-service.ts` に Amazon Location Service を使用したルート検索・ルート逸脱判定・迷子状態判定を実装
    - `isDeviatedFromRoute`（最近接点からの距離が閾値超過）、`isLost`（目的地から500m以上かつ10分以上滞在）を実装
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6_

  - [ ]* 13.2 ルート逸脱判定のプロパティベーステストを作成
    - **Property 6: ルート逸脱判定** — 任意の現在位置とルート座標列に対して最近接点からの距離が閾値超過時のみ逸脱と判定されることを検証
    - **Validates: Requirements 3.2**

  - [ ]* 13.3 迷子状態判定のプロパティベーステストを作成
    - **Property 7: 迷子状態判定** — 任意の現在位置・目的地・滞在時間に対して500m以上かつ10分以上の場合にのみ迷子と判定されることを検証
    - **Validates: Requirements 3.3**

- [ ] 14. ホームページ・レイアウト統合
  - [ ] 14.1 共通レイアウトとホームページを実装
    - `src/app/layout.tsx` — 親キャラクターアバター常時表示、ナビゲーションメニュー
    - `src/app/page.tsx` — ダッシュボード（今日の予定、ファザコン度、最新の親コメント、各機能へのリンク）
    - _Requirements: 2.1, 全体_

  - [ ] 14.2 全ページの統合とルーティング確認
    - 各ページ間の遷移を確認
    - 親キャラクターの表情がメッセージの感情に連動することを確認
    - _Requirements: 2.1, 2.2, 2.3, 2.4_

- [ ] 15. 最終チェックポイント - 全機能の統合確認
  - すべてのテストが通ることを確認し、ユーザーに質問があれば確認する。

## 備考

- `*` マーク付きのタスクはオプションであり、MVPを素早く完成させるためにスキップ可能
- 各タスクは特定の要件を参照しており、トレーサビリティを確保
- チェックポイントで段階的に品質を検証
- プロパティベーステストは設計ドキュメントの正当性プロパティに基づく
- ユニットテストは具体的なシナリオとエッジケースを検証
- ハッカソン向けに、タスク1〜7（診断・UI・LINE連携・スケジュール管理）を最優先で実装することを推奨
