# 要件ドキュメント: ファザコン・マザコン養成AI

## はじめに

「ファザコン・マザコン養成AI」は、AWS Summitハッカソン「人をダメにするサービス」テーマに基づくサービスである。ユーザー（子供役）の日常生活に対して、AIが過保護な親のように徹底的におせっかいを焼くことで、ユーザーを「自分では何もできない状態」＝ファザコン・マザコンに養成する。天気・乗換案内・スケジュール管理・学習計画・日記作成など、本来自分でやるべきことをすべて親AIが代行し、ユーザーの自立心を奪い取る。

## 用語集

- **Parent_AI**: ユーザーに対して過保護な親として振る舞うAIシステム全体
- **Child_User**: サービスを利用するユーザー（子供役）
- **Fazacon_Score**: ユーザーのファザコン・マザコン度を数値化したスコア（0〜100）
- **Morning_Routine_Engine**: 朝の起床から外出までの一連のおせっかいを管理するエンジン
- **Schedule_Manager**: Child_Userの予定を管理し、それに基づいたおせっかいを生成するモジュール
- **Navigation_Guardian**: 迷子防止・タクシー手配・最適ルート案内を行うモジュール
- **Study_Planner**: 試験に向けた学習計画を立案するモジュール
- **Diary_Generator**: Child_Userの行動ログから日記を自動生成するモジュール
- **Worry_Engine**: 親の心配性を再現し、過剰な心配メッセージを生成するエンジン
- **Osekkai_Message**: Parent_AIがChild_Userに送る、おせっかいなメッセージ
- **Dependency_Level**: Child_UserがParent_AIにどれだけ依存しているかの度合い

## 要件

### 要件1: ファザコン度診断

**ユーザーストーリー:** Child_Userとして、自分のファザコン・マザコン度を知りたい。それにより、自分がどれだけ「ダメになっているか」を楽しく確認できる。

#### 受け入れ基準

1. WHEN Child_Userが診断を開始した時、THE Parent_AI SHALL 10問の質問を順番に表示する
2. WHEN Child_Userが全ての質問に回答した時、THE Parent_AI SHALL Fazacon_Scoreを0〜100の範囲で算出する
3. WHEN Fazacon_Scoreが算出された時、THE Parent_AI SHALL スコアに応じた称号（「まだまだ自立してるわね」「いい子ね、もっとママに頼りなさい」「完璧なファザコン！パパ嬉しい！」等）を表示する
4. WHEN 診断結果が表示された時、THE Parent_AI SHALL 結果をSNSでシェアするためのテキストとリンクを生成する
5. WHEN Child_Userがサービスを継続利用した時、THE Parent_AI SHALL 利用頻度・依存度に基づいてFazacon_Scoreを自動更新する

### 要件2: 面白いUI・演出

**ユーザーストーリー:** Child_Userとして、親AIとのやり取りを楽しみたい。それにより、サービスを使い続けるモチベーションが生まれる。

#### 受け入れ基準

1. THE Parent_AI SHALL 画面上部に常時表示される親キャラクター（父・母を選択可能）のアバターを表示する
2. WHEN Osekkai_Messageが送信された時、THE Parent_AI SHALL 親キャラクターの表情をメッセージの感情（心配・喜び・怒り・泣き）に合わせて変化させる
3. WHEN Child_Userがおせっかいを無視した時、THE Parent_AI SHALL 親キャラクターが拗ねるアニメーションを再生する
4. WHEN Child_Userがおせっかいに従った時、THE Parent_AI SHALL 親キャラクターが喜ぶアニメーションと「いい子ね！」等の褒め言葉を表示する
5. THE Parent_AI SHALL メッセージの語尾を親キャラクターの性格に合わせる（父：「〜だぞ」「〜しなさい」、母：「〜でしょ？」「〜なさいね」）
6. WHEN Child_Userが深夜にアプリを開いた時、THE Parent_AI SHALL 「こんな時間まで起きてるの！？早く寝なさい！」と叱るメッセージを表示する
7. WHEN Child_Userが長時間アプリを使用していない時、THE Parent_AI SHALL 「最近連絡ないけど元気にしてるの？ちゃんとご飯食べてる？」と心配するプッシュ通知を送信する

### 要件3: 迷子防止・ナビゲーション

**ユーザーストーリー:** Child_Userとして、外出時に迷子にならないようにしたい。それにより、自分で道を覚える必要がなくなり、さらにダメになれる。

#### 受け入れ基準

1. WHEN Child_Userが目的地を設定した時、THE Navigation_Guardian SHALL 最も安全で分かりやすいルートを提案する
2. WHEN Child_Userがルートから外れた時、THE Navigation_Guardian SHALL 「あらあら、道間違えちゃったの？こっちよ！」とOsekkai_Messageで正しい方向を案内する
3. WHEN Child_Userが目的地から500m以上離れた場所で10分以上移動していない時、THE Navigation_Guardian SHALL 迷子状態と判定する
4. WHEN 迷子状態と判定された時、THE Navigation_Guardian SHALL 「もう！迷子になっちゃったの？近くの交番を探してあげるからね！」とメッセージを表示し、現在地周辺の交番・警察署の位置情報を検索して表示する
5. WHILE Child_Userがナビゲーション中である時、THE Navigation_Guardian SHALL 「次の角を右よ」「もうすぐ着くわよ、頑張って！」等の励ましメッセージを定期的に表示する
6. WHEN Child_Userが目的地に到着した時、THE Navigation_Guardian SHALL 「よく頑張ったわね！えらい！」と褒めるメッセージを表示する

### 要件4: 朝のおせっかいルーティン

**ユーザーストーリー:** Child_Userとして、朝の準備を全て親AIに管理してもらいたい。それにより、自分で考えて行動する力を完全に失える。

#### 受け入れ基準

1. WHEN 設定された起床時刻になった時、THE Morning_Routine_Engine SHALL 「朝よ！起きなさい！」とアラームとOsekkai_Messageを送信する
2. WHEN Child_Userがアラームを止めなかった時、THE Morning_Routine_Engine SHALL 30秒ごとに「まだ寝てるの！？遅刻するわよ！」と段階的に強い口調のメッセージを送信する
3. WHEN Child_Userが起床を確認した時、THE Morning_Routine_Engine SHALL 当日の天気予報を取得し、気温に応じて「今日は寒いから上着持っていきなさいね」等の服装アドバイスを表示する
4. WHEN 当日の降水確率が30%以上の時、THE Morning_Routine_Engine SHALL 「傘持った？絶対忘れないでね！」とOsekkai_Messageを送信する
5. WHEN Child_Userの当日の予定がSchedule_Managerに登録されている時、THE Morning_Routine_Engine SHALL 乗換案内APIを使用して最適な出発時刻を算出し、「余裕を見て{出発時刻}にはお家出なさいね」とメッセージを表示する
6. WHEN Child_Userが出発予定時刻の10分前になっても外出していない時、THE Morning_Routine_Engine SHALL 「もう！まだ家にいるの！？急ぎなさい！」と緊急メッセージを送信する
7. WHEN Child_Userが外出した時、THE Morning_Routine_Engine SHALL 「忘れ物ない？ハンカチ持った？ティッシュは？」と最終確認メッセージを送信する

### 要件5: 試験・学習計画

**ユーザーストーリー:** Child_Userとして、試験に向けた学習計画を親AIに立ててもらいたい。それにより、自分で計画を立てる能力を失い、さらに親に依存できる。

#### 受け入れ基準

1. WHEN Child_Userが試験情報（科目・日程・範囲）を登録した時、THE Study_Planner SHALL 試験日までの日数を逆算し、日別の学習計画を自動生成する
2. WHEN 学習計画が生成された時、THE Study_Planner SHALL 各日の学習内容・目標時間・休憩タイミングを含む詳細スケジュールを表示する
3. WHEN 学習予定の時間になった時、THE Study_Planner SHALL 「そろそろ勉強の時間よ！{科目名}のテキスト開きなさい！」とOsekkai_Messageを送信する
4. WHEN Child_Userが学習完了を報告した時、THE Study_Planner SHALL 「よく頑張ったわね！ご褒美にアイス買ってあげる！」と褒めるメッセージを表示する
5. WHEN Child_Userが学習予定をサボった時、THE Study_Planner SHALL 「あなたの将来が心配なの！ちゃんとやりなさい！」と叱るメッセージを送信し、計画を再調整する
6. IF Child_Userが3日連続で学習予定をサボった場合、THEN THE Study_Planner SHALL 「もう知らない！…嘘よ、やっぱり心配だから一緒にやりましょう」と感情的なメッセージを表示する

### 要件6: 行動日記の自動生成

**ユーザーストーリー:** Child_Userとして、自分の行動記録を親AIに日記として書いてもらいたい。それにより、自分で振り返る力を失い、記録すら親に依存できる。

#### 受け入れ基準

1. WHILE Child_Userがサービスを利用中である時、THE Diary_Generator SHALL 位置情報・アプリ操作・予定消化状況を行動ログとして記録する
2. WHEN 1日の終わりになった時（22:00）、THE Diary_Generator SHALL 行動ログを基に親目線の日記を自動生成する
3. WHEN 日記が生成された時、THE Diary_Generator SHALL 「今日のうちの子」というタイトルで、親が子供の行動を語る形式で日記を表示する
4. THE Diary_Generator SHALL 日記に親の感想（「今日はちゃんと傘持って行けてえらかったわ」「帰りが遅くて心配したのよ」等）を含める
5. WHEN Child_Userが日記を閲覧した時、THE Diary_Generator SHALL 過去の日記を時系列で一覧表示する
6. WHEN 1週間分の日記が蓄積された時、THE Diary_Generator SHALL 「今週のうちの子まとめ」として週間サマリーを生成する

### 要件7: 過保護心配エンジン

**ユーザーストーリー:** Child_Userとして、常に親に心配されている安心感を得たい。それにより、自分で危機管理する能力を完全に手放せる。

#### 受け入れ基準

1. WHEN 気象警報が発令された時、THE Worry_Engine SHALL 「大変！{警報内容}が出てるわよ！外出しないで！お願いだから！」と緊急Osekkai_Messageを送信する
2. WHEN Child_Userが食事の時間（12:00, 19:00）を過ぎても食事記録がない時、THE Worry_Engine SHALL 「ちゃんとご飯食べた？コンビニ弁当ばっかりじゃダメよ！」とメッセージを送信する
3. WHEN Child_Userが23:00以降もアプリを操作している時、THE Worry_Engine SHALL 「まだ起きてるの！？明日に響くわよ！早く寝なさい！」と就寝を促すメッセージを送信する
4. WHEN Child_Userが体調不良を報告した時、THE Worry_Engine SHALL 「大丈夫！？熱は？お水飲んだ？すぐ病院行きなさい！近くの病院はここよ！」と近隣の病院情報を含むメッセージを送信する

### 要件8: 褒めて伸ばす依存強化

**ユーザーストーリー:** Child_Userとして、些細なことでも褒められたい。それにより、親の承認なしでは行動できない人間になれる。

#### 受け入れ基準

1. WHEN Child_Userが予定通りに行動した時、THE Parent_AI SHALL 「さすがうちの子！天才！」等の過剰な褒め言葉を表示する
2. WHEN Child_Userが朝時間通りに起床した時、THE Parent_AI SHALL 「えらい！起きれたのね！パパ感動！」と褒めるメッセージとアニメーションを表示する
3. THE Parent_AI SHALL Child_Userの行動に対して「ご褒美ポイント」を付与する
4. WHEN ご褒美ポイントが一定数に達した時、THE Parent_AI SHALL 「頑張ったご褒美に好きなもの買ってあげる！」と仮想ご褒美を表示する

### 要件9: おせっかいスケジュール管理

**ユーザーストーリー:** Child_Userとして、予定の管理を全て親AIに任せたい。それにより、自分でスケジュールを把握する必要がなくなる。

#### 受け入れ基準

1. WHEN Child_Userが予定を登録した時、THE Schedule_Manager SHALL 予定の前日に「明日は{予定名}よ！準備はできてる？」とリマインドメッセージを送信する
2. WHEN 予定の当日になった時、THE Schedule_Manager SHALL 予定の2時間前・1時間前・30分前にそれぞれ段階的なリマインドを送信する
3. WHEN 複数の予定が同日にある時、THE Schedule_Manager SHALL 予定間の移動時間を考慮し、「次の予定まで{移動時間}分かかるから、{出発時刻}には出なさいね」と案内する
4. WHEN Child_Userが予定を忘れそうな時（予定開始15分前に反応がない場合）、THE Schedule_Manager SHALL 「もしかして忘れてない！？{予定名}は{時刻}からよ！」と緊急リマインドを送信する
5. WHEN 予定が完了した時、THE Schedule_Manager SHALL 「お疲れ様！次の予定は{次の予定名}よ。それまでゆっくり休みなさいね」と労いのメッセージを表示する
6. WHEN Child_UserがLINEで自然言語のテキスト（例：「明日14時に歯医者」「来週月曜10時から会議」）を送信した時、THE Schedule_Manager SHALL Bedrockを使用してテキストから予定名・日時を抽出し、スケジュールを自動登録する
7. WHEN LINEからスケジュールが登録された時、THE Schedule_Manager SHALL 「{予定名}を{日時}に登録したわよ！ちゃんと覚えておきなさいね！」と確認メッセージをLINEで返信する
8. WHEN Bedrockがテキストから日時を正しく抽出できなかった時、THE Schedule_Manager SHALL 「ごめんなさい、いつの予定かわからなかったわ。もう一度教えてくれる？」と聞き返すメッセージをLINEで返信する

