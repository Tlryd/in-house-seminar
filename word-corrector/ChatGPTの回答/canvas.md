# 要件定義書: Google Docs議事メモ誤字修正アプリ

## 1. 概要

### 1.1 目的

オンライン会議の自動文字起こし結果としてGoogle Docsに保存された議事メモを対象に、音声認識由来の誤字・誤変換・表記ゆれを検出し、修正候補を提示・反映できるアプリを開発する。

本アプリは、単なる文章校正ではなく、業務用語・IT用語・固有名詞・社内用語の誤変換修正を主目的とする。

### 1.2 基本方針

* 辞書による確実な修正を優先する
* AIは辞書で拾えない誤変換の補助に使う
* 初期版では元のGoogle Docs文書を直接破壊的に変更しない
* 修正結果は、元文書のコピーとして作成する
* 誤修正を避けるため、AI候補は原則としてユーザー確認後に反映する

### 1.3 想定利用者

* オンライン会議の議事メモを確認・整形する担当者
* 開発・保守・運用案件の会議記録を扱うエンジニア
* 顧客名、システム名、プロジェクト名、IT用語の誤変換を修正したい利用者

---

## 2. スコープ

### 2.1 MVPで実装する範囲

MVPでは以下を実装対象とする。

1. Googleアカウントによるログイン
2. Google Docs URL入力による対象文書指定
3. Google Docs本文の取得
4. 固定辞書による誤変換検出
5. AIによる追加の修正候補生成
6. 修正候補一覧の表示
7. ユーザーによる個別承認・却下
8. 承認済み修正を反映したGoogle Docsコピーの作成
9. 修正履歴の保存
10. 誤変換パターンの辞書登録

### 2.2 MVPで実装しない範囲

以下は初期版では対象外とする。

* Google Docs上でのリアルタイム校正
* 元文書への直接上書き修正
* Google Docsの提案モード完全再現
* 複数ユーザーによる同時編集制御
* 会議音声ファイルとの照合
* 議事録要約機能
* タスク抽出機能
* 発言者別の修正最適化
* 組織全体の高度な辞書管理

---

## 3. 用語定義

| 用語     | 定義                                |
| ------ | --------------------------------- |
| 議事メモ   | オンライン会議の自動文字起こし結果を含むGoogle Docs文書 |
| 誤変換    | 音声認識や自動文字起こしにより、本来とは異なる語句に変換されたもの |
| 固定辞書   | 人が事前に登録する正誤変換ルールや用語一覧             |
| 学習辞書   | ユーザーの修正履歴から生成される誤変換パターン           |
| 修正候補   | 誤字・誤変換に対する置換案                     |
| 確信度    | 修正候補の信頼度を示す値または分類                 |
| 修正版コピー | 元文書をコピーし、承認済み修正を反映したGoogle Docs文書 |

---

## 4. 想定システム構成

### 4.1 アプリ形態

Webアプリとして実装する。

想定構成は以下とする。

* フロントエンド

  * 文書URL入力
  * 検出結果一覧表示
  * 修正候補の承認・却下
  * 辞書登録画面
* バックエンド

  * Google OAuth連携
  * Google Docs API連携
  * Google Drive API連携
  * 辞書照合処理
  * AI補完処理
  * 修正反映処理
  * 履歴保存
* データベース

  * ユーザー情報
  * 辞書
  * 修正履歴
  * 実行履歴

### 4.2 外部サービス

* Google OAuth
* Google Docs API
* Google Drive API
* AI API

  * 実装時に利用モデルを差し替え可能な構成にする
  * Gemini、OpenAI API、Amazon Bedrock等を将来選択できるようにする

---

## 5. 機能要件

## 5.1 認証機能

### 5.1.1 Googleログイン

ユーザーはGoogleアカウントでログインできる。

#### 要件

* Google OAuthを使用する
* ユーザーがアクセス可能なGoogle Docsのみ処理対象にする
* アプリが要求するスコープは必要最小限にする
* ログイン状態を保持できる
* ログアウトできる

#### 受け入れ条件

* ユーザーがGoogleアカウントでログインできる
* ログイン後、自分が権限を持つGoogle Docs URLを指定できる
* 権限のない文書を指定した場合、エラーとして扱われる

---

## 5.2 文書指定機能

### 5.2.1 Google Docs URL入力

ユーザーはGoogle DocsのURLを入力して対象文書を指定できる。

#### 要件

* Google Docs URLを入力できる
* URLからdocumentIdを抽出できる
* 不正なURLの場合はエラーを表示する
* Google Docs以外のURLは対象外とする
* 対象文書のタイトルを取得して画面に表示する

#### 受け入れ条件

* 正しいGoogle Docs URLを入力すると文書情報を取得できる
* URL形式が不正な場合、処理を開始しない
* アクセス権限がない場合、権限エラーを表示する

---

## 5.3 文書取得機能

### 5.3.1 Google Docs本文取得

対象Google Docs文書から本文テキストを取得する。

#### 要件

* Google Docs APIを使用して本文を取得する
* 段落単位で本文を扱えるようにする
* 各段落の位置情報を保持する
* 修正反映時に対象箇所を特定できる情報を保持する
* コメント、提案、変更履歴はMVPでは処理対象外とする

#### 受け入れ条件

* 対象文書の本文を取得できる
* 段落ごとのテキストを内部データとして保持できる
* 空文書の場合はエラーまたは対象なしとして扱える

---

## 5.4 辞書管理機能

### 5.4.1 固定辞書

誤変換パターンを辞書として管理する。

#### 要件

辞書項目は以下の情報を持つ。

| 項目           | 内容     |
| ------------ | ------ |
| id           | 辞書項目ID |
| wrong_text   | 誤変換文字列 |
| correct_text | 正しい文字列 |
| category     | 分類     |
| description  | 説明     |
| enabled      | 有効・無効  |
| priority     | 優先度    |
| created_at   | 作成日時   |
| updated_at   | 更新日時   |

#### categoryの例

* IT用語
* 人名
* 会社名
* システム名
* プロジェクト名
* 製品名
* 社内用語
* 顧客用語
* 表記ゆれ

#### 辞書例

| wrong_text | correct_text | category |
| ---------- | ------------ | -------- |
| ドネ         | .NET         | IT用語     |
| じゃば        | Java         | IT用語     |
| ギットハブ      | GitHub       | IT用語     |
| コパイロット     | Copilot      | IT用語     |
| サーバ        | サーバー         | 表記ゆれ     |

#### 受け入れ条件

* 辞書項目を登録できる
* 辞書項目を編集できる
* 辞書項目を無効化できる
* 有効な辞書項目だけが検出処理に使われる

---

### 5.4.2 修正履歴からの辞書登録

ユーザーが承認した修正を辞書に登録できる。

#### 要件

* 修正候補の承認後、同じ誤変換を今後も修正候補として扱えるように辞書登録できる
* 登録時にカテゴリを選択できる
* 既存辞書と重複する場合は警告する
* 登録するかどうかはユーザーが選べる

#### 受け入れ条件

* 承認済み修正から辞書項目を作成できる
* 重複登録を避けられる
* 次回以降の検出処理で登録済み辞書が使われる

---

## 5.5 誤字・誤変換検出機能

### 5.5.1 辞書ベース検出

固定辞書および学習辞書を使い、文書内の誤変換候補を検出する。

#### 要件

* 文書本文に対して辞書のwrong_textを検索する
* 一致した箇所を修正候補として抽出する
* 同一誤変換が複数箇所にある場合、すべて検出する
* 文字位置または段落位置を保持する
* 辞書由来の候補は確信度を高く設定する

#### 受け入れ条件

* 辞書に登録された誤変換が文書内で検出される
* 検出結果に修正前・修正後・分類・理由が表示される
* 同じ誤変換が複数回登場する場合、複数件として扱える

---

### 5.5.2 表記ゆれ検出

表記ゆれ辞書を使い、統一対象の語句を検出する。

#### 要件

* 表記ゆれカテゴリの辞書を使う
* 統一後の表記を修正候補として提示する
* 文書全体で表記が混在している場合に検出する

#### 受け入れ条件

* 「サーバ」と「サーバー」などの表記ゆれを検出できる
* 統一候補を提示できる

---

### 5.5.3 AI補完検出

辞書で検出できない不自然な語句をAIで検出する。

#### 要件

* 対象文書または段落をAIに渡して修正候補を生成する
* AIには全文を無制限に渡さず、必要範囲に分割する
* AIからは構造化データで候補を受け取る
* AI候補には確信度を付ける
* AI候補は自動反映せず、ユーザー確認対象にする
* 辞書候補とAI候補が重複した場合は辞書候補を優先する

#### AI出力形式

AIからは以下のJSON形式で候補を返す想定とする。

```json
{
  "candidates": [
    {
      "paragraph_index": 0,
      "original_text": "コパイロット",
      "suggested_text": "Copilot",
      "reason": "IT用語として自然な表記に修正",
      "confidence": "medium",
      "category": "IT用語"
    }
  ]
}
```

#### 受け入れ条件

* 辞書にない誤変換についてAI候補を提示できる
* AI候補は一覧で確認できる
* AI候補は承認されるまで修正に反映されない

---

## 5.6 修正候補表示機能

### 5.6.1 候補一覧表示

検出された修正候補を一覧表示する。

#### 要件

候補一覧には以下を表示する。

| 項目   | 内容                  |
| ---- | ------------------- |
| 対象箇所 | 段落番号または文脈           |
| 修正前  | 元の文字列               |
| 修正後  | 修正候補                |
| 種別   | IT用語、人名、表記ゆれ等       |
| 検出元  | 辞書、AI、修正履歴          |
| 確信度  | high / medium / low |
| 理由   | 修正候補の理由             |
| 操作   | 承認、却下、編集            |

#### 受け入れ条件

* 検出候補が一覧で表示される
* 候補ごとに承認・却下できる
* 候補ごとに修正後文字列を編集できる

---

### 5.6.2 候補フィルタ

候補を種別・検出元・確信度で絞り込める。

#### 要件

* 種別で絞り込める
* 検出元で絞り込める
* 確信度で絞り込める
* 承認済み・却下済み・未確認で絞り込める

#### 受け入れ条件

* 候補数が多い場合でも確認しやすい
* 未確認候補だけを表示できる

---

## 5.7 修正承認機能

### 5.7.1 個別承認・却下

ユーザーは候補ごとに承認・却下できる。

#### 要件

* 候補を個別に承認できる
* 候補を個別に却下できる
* 修正後文字列を編集してから承認できる
* 承認・却下状態を保存する

#### 受け入れ条件

* 承認した候補だけが修正版コピーに反映される
* 却下した候補は反映されない
* 編集済み候補は編集後の文字列で反映される

---

### 5.7.2 同一パターンの一括承認

同じ誤変換パターンをまとめて承認できる。

#### 要件

* 同じoriginal_textとsuggested_textの候補をグルーピングできる
* グループ単位で一括承認できる
* グループ単位で一括却下できる

#### 受け入れ条件

* 文書内に同じ誤変換が複数ある場合、一括処理できる
* 一括承認後、対象候補の状態がすべて承認済みになる

---

## 5.8 修正反映機能

### 5.8.1 修正版Google Docsコピー作成

承認済み候補を反映したGoogle Docsコピーを作成する。

#### 要件

* 元文書をGoogle Drive上でコピーする
* コピー文書のタイトルは元文書名に接尾辞を付ける
* 例: `元文書名_修正版_YYYYMMDD_HHmm`
* 承認済み候補のみコピー文書に反映する
* 元文書は変更しない
* 修正完了後、修正版コピーのURLを表示する

#### 受け入れ条件

* 元文書が変更されない
* 修正版コピーがGoogle Drive上に作成される
* 承認済み修正だけが反映される
* 完了後に修正版コピーを開ける

---

## 5.9 実行履歴機能

### 5.9.1 実行履歴保存

文書チェックの実行履歴を保存する。

#### 要件

実行履歴には以下を保存する。

| 項目                    | 内容       |
| --------------------- | -------- |
| id                    | 実行ID     |
| user_id               | 実行者      |
| source_document_id    | 元文書ID    |
| source_document_title | 元文書タイトル  |
| copied_document_id    | 修正版コピーID |
| candidate_count       | 検出件数     |
| approved_count        | 承認件数     |
| rejected_count        | 却下件数     |
| status                | 実行状態     |
| created_at            | 実行日時     |
| completed_at          | 完了日時     |

#### 受け入れ条件

* 実行ごとに履歴が保存される
* 過去の実行結果を確認できる
* 修正版コピーのURLを再確認できる

---

## 5.10 修正履歴機能

### 5.10.1 修正履歴保存

候補ごとの修正結果を保存する。

#### 要件

修正履歴には以下を保存する。

| 項目              | 内容                           |
| --------------- | ---------------------------- |
| id              | 修正履歴ID                       |
| run_id          | 実行ID                         |
| paragraph_index | 段落番号                         |
| original_text   | 修正前文字列                       |
| suggested_text  | 修正候補                         |
| final_text      | 実際に反映した文字列                   |
| source          | dictionary / ai / history    |
| category        | 分類                           |
| confidence      | 確信度                          |
| status          | approved / rejected / edited |
| reason          | 理由                           |
| created_at      | 作成日時                         |

#### 受け入れ条件

* 候補ごとの承認・却下結果が保存される
* 編集後に承認した場合、final_textに編集後文字列が保存される
* 修正履歴を辞書登録に利用できる

---

## 6. 画面要件

## 6.1 ログイン画面

### 表示項目

* アプリ名
* Googleログインボタン
* 利用上の注意

### 操作

* Googleアカウントでログインする

---

## 6.2 文書指定画面

### 表示項目

* Google Docs URL入力欄
* チェック開始ボタン
* 対象文書タイトル
* エラーメッセージ

### 操作

* URLを入力する
* チェックを開始する

---

## 6.3 修正候補一覧画面

### 表示項目

* 対象文書タイトル
* 検出件数
* 承認件数
* 却下件数
* 未確認件数
* 候補一覧テーブル
* フィルタ
* 一括承認ボタン
* 一括却下ボタン
* 修正版コピー作成ボタン

### 操作

* 候補を承認する
* 候補を却下する
* 候補の修正後文字列を編集する
* 条件で絞り込む
* 承認済み候補を反映してコピーを作成する

---

## 6.4 辞書管理画面

### 表示項目

* 辞書一覧
* 誤変換文字列
* 正しい文字列
* カテゴリ
* 有効・無効
* 優先度

### 操作

* 辞書項目を追加する
* 辞書項目を編集する
* 辞書項目を無効化する
* 辞書項目を削除する

---

## 6.5 実行履歴画面

### 表示項目

* 実行日時
* 文書タイトル
* 検出件数
* 承認件数
* 却下件数
* 実行状態
* 修正版コピーURL

### 操作

* 過去の実行結果を見る
* 修正版コピーを開く

---

## 7. データ設計案

## 7.1 users

```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  google_user_id TEXT UNIQUE NOT NULL,
  email TEXT NOT NULL,
  name TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

## 7.2 dictionaries

```sql
CREATE TABLE dictionaries (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  wrong_text TEXT NOT NULL,
  correct_text TEXT NOT NULL,
  category TEXT NOT NULL,
  description TEXT,
  enabled BOOLEAN NOT NULL DEFAULT TRUE,
  priority INTEGER NOT NULL DEFAULT 100,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## 7.3 correction_runs

```sql
CREATE TABLE correction_runs (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  source_document_id TEXT NOT NULL,
  source_document_title TEXT NOT NULL,
  copied_document_id TEXT,
  candidate_count INTEGER NOT NULL DEFAULT 0,
  approved_count INTEGER NOT NULL DEFAULT 0,
  rejected_count INTEGER NOT NULL DEFAULT 0,
  status TEXT NOT NULL,
  error_message TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## 7.4 correction_candidates

```sql
CREATE TABLE correction_candidates (
  id TEXT PRIMARY KEY,
  run_id TEXT NOT NULL,
  paragraph_index INTEGER NOT NULL,
  original_text TEXT NOT NULL,
  suggested_text TEXT NOT NULL,
  final_text TEXT,
  source TEXT NOT NULL,
  category TEXT,
  confidence TEXT NOT NULL,
  reason TEXT,
  status TEXT NOT NULL DEFAULT 'pending',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (run_id) REFERENCES correction_runs(id)
);
```

---

## 8. API設計案

## 8.1 認証

### GET /api/auth/google

Google OAuthログインを開始する。

### GET /api/auth/callback

Google OAuthのコールバックを処理する。

### POST /api/auth/logout

ログアウトする。

---

## 8.2 文書処理

### POST /api/documents/inspect

Google Docs URLから文書情報を取得する。

#### Request

```json
{
  "url": "https://docs.google.com/document/d/xxxx/edit"
}
```

#### Response

```json
{
  "documentId": "xxxx",
  "title": "会議メモ",
  "paragraphCount": 120
}
```

---

### POST /api/correction-runs

誤変換検出を開始する。

#### Request

```json
{
  "documentUrl": "https://docs.google.com/document/d/xxxx/edit",
  "useAi": true
}
```

#### Response

```json
{
  "runId": "run_xxxx",
  "candidateCount": 25
}
```

---

### GET /api/correction-runs/:runId/candidates

修正候補一覧を取得する。

---

### PATCH /api/correction-candidates/:candidateId

候補の状態を更新する。

#### Request

```json
{
  "status": "approved",
  "finalText": "GitHub Copilot"
}
```

---

### POST /api/correction-runs/:runId/apply

承認済み候補を反映したGoogle Docsコピーを作成する。

#### Response

```json
{
  "copiedDocumentId": "yyyy",
  "copiedDocumentUrl": "https://docs.google.com/document/d/yyyy/edit"
}
```

---

## 8.3 辞書

### GET /api/dictionaries

辞書一覧を取得する。

### POST /api/dictionaries

辞書項目を登録する。

### PATCH /api/dictionaries/:dictionaryId

辞書項目を更新する。

### DELETE /api/dictionaries/:dictionaryId

辞書項目を削除または無効化する。

---

## 9. 処理フロー

## 9.1 文書チェックの基本フロー

1. ユーザーがGoogleログインする
2. ユーザーがGoogle Docs URLを入力する
3. アプリがURLからdocumentIdを抽出する
4. アプリがGoogle Docs APIで文書本文を取得する
5. アプリが段落単位で本文を分割する
6. アプリが辞書ベースで誤変換を検出する
7. useAiがtrueの場合、AI補完検出を実行する
8. 辞書候補とAI候補を統合する
9. 重複候補を整理する
10. 修正候補をDBに保存する
11. ユーザーに候補一覧を表示する
12. ユーザーが候補を承認・却下する
13. ユーザーが修正版コピー作成を実行する
14. アプリが元文書をコピーする
15. アプリがコピー文書に承認済み修正を反映する
16. アプリが修正版コピーURLを表示する

---

## 9.2 候補統合ルール

候補が重複した場合の優先順位は以下とする。

1. 固定辞書
2. 学習辞書
3. AI候補

同じ対象箇所に複数候補がある場合、優先順位が高い候補を主候補として表示する。

---

## 9.3 確信度ルール

| 検出元  | 条件             | 確信度    |
| ---- | -------------- | ------ |
| 固定辞書 | 完全一致           | high   |
| 学習辞書 | 過去に複数回承認済み     | high   |
| 学習辞書 | 過去に1回承認済み      | medium |
| AI   | 明確なIT用語・固有名詞候補 | medium |
| AI   | 文脈推定のみ         | low    |

---

## 10. 非機能要件

## 10.1 性能

* 数万文字程度のGoogle Docs文書を処理できること
* 通常の議事メモでは、候補生成が実用的な時間内に完了すること
* 大きな文書では段落単位またはチャンク単位で処理すること
* AI APIの呼び出し回数を抑えること

## 10.2 可用性

* Google APIやAI APIでエラーが発生した場合、ユーザーに分かる形で表示すること
* 一時的なAPIエラーには再試行できること
* 処理途中で失敗した場合、元文書を変更しないこと

## 10.3 セキュリティ

* OAuthトークンを安全に管理すること
* 文書本文を不要に永続保存しないこと
* 保存が必要な場合は最小限にすること
* AI APIに送信する内容は必要範囲に限定すること
* ログに文書本文を不用意に出力しないこと
* ユーザーがアクセス権限を持たない文書は処理しないこと

## 10.4 保守性

* AIモデルを差し替えられるように抽象化すること
* Google Docs連携処理を独立したモジュールにすること
* 辞書検出処理を独立したモジュールにすること
* 修正反映処理を独立したモジュールにすること
* テストしやすい構造にすること

---

## 11. エラーハンドリング

## 11.1 想定エラー

| エラー                | 対応                 |
| ------------------ | ------------------ |
| Google未ログイン        | ログイン画面に誘導する        |
| 不正なGoogle Docs URL | URL形式エラーを表示する      |
| 文書アクセス権限なし         | 権限エラーを表示する         |
| 文書本文なし             | 対象テキストなしとして表示する    |
| Google API制限       | 時間を置いて再試行するよう表示する  |
| AI APIエラー          | 辞書候補のみで続行可能にする     |
| コピー作成失敗            | 元文書は変更せずエラー表示する    |
| 修正反映失敗             | コピー文書URLと失敗内容を表示する |

---

## 12. テスト観点

## 12.1 単体テスト

* Google Docs URLからdocumentIdを抽出できる
* 不正URLを検出できる
* 辞書項目と本文を照合できる
* 同一誤変換を複数検出できる
* 候補の重複を統合できる
* 承認済み候補だけを抽出できる
* 修正履歴から辞書登録できる

## 12.2 結合テスト

* Googleログイン後に文書情報を取得できる
* 文書本文を取得して候補を生成できる
* 候補を承認・却下できる
* 修正版コピーを作成できる
* 修正版コピーに承認済み修正だけが反映される

## 12.3 異常系テスト

* 権限のない文書URLを指定する
* 存在しない文書URLを指定する
* 空の文書を指定する
* AI APIが失敗する
* Google Docs APIが失敗する
* コピー作成中に失敗する

---

## 13. 初期データ

MVPでは以下のサンプル辞書を初期登録する。

| wrong_text | correct_text | category |
| ---------- | ------------ | -------- |
| ドネ         | .NET         | IT用語     |
| ドットネット     | .NET         | IT用語     |
| じゃば        | Java         | IT用語     |
| ジャバスクリプト   | JavaScript   | IT用語     |
| ギットハブ      | GitHub       | IT用語     |
| コパイロット     | Copilot      | IT用語     |
| サーバ        | サーバー         | 表記ゆれ     |
| ユーザ        | ユーザー         | 表記ゆれ     |
| デービー       | DB           | IT用語     |
| エーピーアイ     | API          | IT用語     |

---

## 14. 実装時の注意点

### 14.1 Google Docsの修正反映

Google Docs APIでは、本文の位置指定や置換処理に注意が必要である。

実装では以下を守ること。

* 元文書には直接変更を加えない
* まずDrive APIでコピーを作成する
* コピー文書に対してのみ修正を反映する
* 複数箇所を置換する場合、位置ずれを避けるため後方から処理する
* 単純な全文置換で誤爆する可能性があるため、段落位置や文脈を使って対象箇所を特定する

### 14.2 AI利用

AI補完は補助機能であり、辞書処理より優先しない。

実装では以下を守ること。

* AI候補は必ず構造化データとして扱う
* AI出力をそのまま信用しない
* JSONパース失敗時の処理を用意する
* AI候補は原則として未承認状態で保存する
* 確信度highであっても、MVPでは自動反映しない

### 14.3 ログ出力

* 本文全体をログに出さない
* OAuthトークンをログに出さない
* AI APIキーをログに出さない
* エラー時も機密情報を含めない

---

## 15. 完了条件

MVPの完了条件は以下とする。

1. Googleログインできる
2. Google Docs URLを指定できる
3. 対象文書の本文を取得できる
4. 辞書に基づく誤変換候補を表示できる
5. AIによる追加候補を表示できる
6. 候補を承認・却下できる
7. 承認済み候補だけを反映したGoogle Docsコピーを作成できる
8. 元文書が変更されない
9. 修正履歴が保存される
10. 承認済み修正から辞書登録できる

---

## 16. Copilot CLI向け実装指示

この要件定義書をもとに、MVPを実装する。

### 実装方針

* まず最小構成で動くWebアプリを作る
* 認証、Google Docs取得、辞書検出、候補表示、コピー作成、修正反映の順に実装する
* AI補完は辞書検出が動作した後に追加する
* 不明点がある場合は、破壊的変更を避ける安全側の仕様を採用する
* 元文書を直接変更する実装は禁止する

### 優先順位

1. Google OAuthログイン
2. Google Docs URL入力
3. 文書本文取得
4. 辞書ベース検出
5. 候補一覧表示
6. 承認・却下
7. Google Docsコピー作成
8. コピー文書への修正反映
9. 修正履歴保存
10. AI補完
11. 辞書管理画面

### 実装後に確認すること

* 元文書が変更されていないこと
* 修正版コピーが作成されること
* 承認済み候補だけが反映されること
* AI候補が勝手に反映されないこと
* エラー時に処理が安全に停止すること
