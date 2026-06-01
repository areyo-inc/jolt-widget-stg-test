# jolt-widget-stg-test

Jolt リファラー機能の stg 環境動作確認用テストページ。

公開URL: https://areyo-inc.github.io/jolt-widget-stg-test/

---

## できること

| 機能 | 内容 |
| --- | --- |
| プログラム切替 | 4プログラム（BtoB Pのみ / P+R / Rのみ / BtoC Rのみ）のプリセットを保存しワンクリックで切替 |
| ウィジェット | フローティング・ポップアップ・インラインの3形式を同一ページに展開。フル表示形式は専用ページ |
| リードフォーム | 紹介リンク経由のリード作成テスト（`lead-form/`）。URL fragment の cookie_value 経由で帰属し、`/api/tracking/submit` → `/api/tracking/cv` を発火 |
| タグ: updateLeadStatus | BtoBリード自動作成タグ（既存）の送信フォーム。qualified / won |
| タグ: track（カスタムイベント） | BtoC カスタムイベントタグ。**任意の `event_name`(slug) + amount** を指定して `Jolt('track', '<event_name>', { amount })` を送信 |
| サーバーAPI: `/api/v1/events` | サーバー間呼び出し用カスタムイベントAPI。`X-Jolt-API-Key` 必須・ブラウザから簡易送信できるフォーム |
| 状態観察 | localStorage / sessionStorage / document.cookie / タグ版本 の確認 |
| テストツール | sessionStorage プローブのリセット / localStorage 全クリア |

---

## ファイル構成

```
.
├── index.html         メインページ（プログラム選択・ウィジェット3形式・タグ送信・状態観察）
├── full.html          フル表示形式（Phase 2）専用ページ
├── lead-form/
│   ├── index.html     紹介リンク経由のリード作成テスト用フォーム
│   └── thanks.html    フォーム送信後のCV発火ページ
└── README.md
```

外部依存:
- ウィジェット: `https://tag.stg.jolt.me/widget/v1/widget.js`
- タグ: `https://tag.stg.jolt.me/v1/tag.js`
- API: `https://external-api.stg.jolt.me`

---

## 使い方

### 初回セットアップ（プリセット登録）

1. https://areyo-inc.github.io/jolt-widget-stg-test/ を開く
2. 「0. プログラム設定」セクションの **プリセットを編集（初回のみ）** を展開
3. 4プログラム分の `vendor_program_id` と `public_token` を入力
   - `vendor_program_id` はベンダーダッシュボードURLの `/programs/XXX` の末尾
   - `public_token` はベンダーダッシュボード → プログラム設定 → 連携設定 から取得
4. 「プリセットを保存」をクリック
   - localStorage に保存される（このブラウザのみ）

### 日常の切替

ヘッダー下の **プリセット切替** ボタンをクリックするだけ。

### URL での直接指定

```
https://areyo-inc.github.io/jolt-widget-stg-test/?program=01XXX...&pubToken=pub_xxx
https://areyo-inc.github.io/jolt-widget-stg-test/?preset=0   # 0..3 でプリセットを切替
```

`preset` 指定が最優先、次に `program` + `pubToken`。

---

## 動作確認の流れ（一例）

```
1. プリセットを切替（Section 0）
2. ウィジェットでメアド入力 → 紹介リンク取得（Section 1）
3. 受信したマジックリンクメールでメール認証 → このページに戻る
4. テストツールの「sessionStorage プローブをリセットしてリロード」を押す（Section 6）
5. ウィジェットが認証済み状態で表示されることを確認（実績タブが詳細表示に）
6. 別ブラウザ（シークレット）で紹介リンクをクリック → 帰属Cookieセット
7. 別ブラウザ側でリードを作成（以下のいずれか）:
   - Section 2 の「リードフォームを開く」→ 問い合わせフォームを送信（既存BtoBパイプライン）
   - Section 3 の updateLeadStatus タグを送信（BtoB既存タグ）
   - Section 4 の track（任意 event_name のカスタムイベント / BtoC）を送信
   - Section 4b の POST /api/v1/events（サーバーAPI・API キー必須 / BtoC）を送信
8. ベンダーダッシュボードの「紹介」一覧で referrer_id がセットされたリードが出ていることを確認
9. BtoC: コンバージョンイベント実績タブで event_name ごとの実績が記録されていることを確認
10. リード → 適格 → 取引成立 を進めて報酬発火を確認
10. リファラーをブロックして、紹介リンクから新規リードが作れなくなることを確認
```

### リードフォーム経由の動作（lead-form/）

紹介リンクをクリック → リダイレクトで lead-form/index.html に遷移 → URL fragment の `cookie_value` を localStorage に保存 → フォーム送信 (`POST /api/tracking/submit`) → thanks.html へ → `POST /api/tracking/cv` でリード作成。

紹介リンクの設定でリダイレクト先を `https://areyo-inc.github.io/jolt-widget-stg-test/lead-form/` に向けると、このページが「ベンダーサイトの問い合わせフォーム」を模した遷移先として動作します。

---

## タグの送信仕様（参考）

### updateLeadStatus（BtoB既存）

```js
Jolt('init', 'pub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx');
Jolt('updateLeadStatus', {
  status: 'qualified' | 'won',        // 必須
  email: 'lead@example.com',          // organization_id 未指定時必須
  organization_id: 'org_xxx',         // email 未指定時必須・推奨
  company_name: '株式会社サンプル',
  last_name: '山田',
  first_name: '太郎'
});
```

エンドポイント: `POST https://external-api.stg.jolt.me/api/tracking/lead-status`

### track（カスタムイベント / BtoC）

BtoC モード改訂で、固定3種（signup/purchase/renewal）は廃止され、**ベンダーがダッシュボードで定義したコンバージョンイベントの `event_name`(slug) を任意に指定**して発火する方式になりました。

```js
// 第2引数がベンダー定義の event_name (slug)。第3引数の payload に amount 等を載せる。
Jolt('track', '<event_name>', {
  vendor_program_id: '01XXX...',
  customer_contact_email: 'customer@example.com',
  amount: 5000,                       // 任意（円。%報酬時必須）
  customer_name: '山田 太郎',          // 任意
  customer_contact_last_name: '山田',  // 任意
  customer_contact_first_name: '太郎', // 任意
  external_source: 'stripe',          // 任意（冪等性に利用）
  external_source_id: 'pi_xxxxxxx'    // 任意（冪等性キー）
});
```

エンドポイント: `POST https://external-api.stg.jolt.me/v1/tag/events`（body に `event_name` を含む）

- `event_name` はベンダーダッシュボードの「コンバージョンイベント定義」で作成した slug と一致させる（未定義の slug はエラー）。
- 帰属は `jolt_referral` Cookie 経由。Cookie がない場合は帰属不能として処理終了（タグはサイレント）。

### サーバーAPI: `POST /api/v1/events`（BtoC / サーバー間呼び出し）

Stripe Webhook 等のサーバーサイドから呼び出すカスタムイベントAPI。Cookie を使わず `customer_contact_email` で既存紹介を解決する。**`X-Jolt-API-Key` ヘッダーが必須**で、`external_source` / `external_source_id` も必須（冪等性キー）。

```bash
curl -X POST https://external-api.stg.jolt.me/api/v1/events \
  -H "Content-Type: application/json" \
  -H "X-Jolt-API-Key: <stg-api-key>" \
  -d '{
    "vendor_program_id": "01XXX...",
    "event_name": "renewal",
    "customer_contact_email": "customer@example.com",
    "amount": 500,
    "external_source": "stripe",
    "external_source_id": "inv_xxxxxxx"
  }'
```

ブラウザからの簡易確認は Section 4b のフォームでも可能（CORS で弾かれる場合は curl で検証）。

### BtoC カスタムイベント実績の確認

送信後、ベンダーダッシュボードの**コンバージョンイベント実績タブ**で、`event_name` ごとのイベント実績が記録されていることを確認できます。

---

## トラブルシュート

### ウィジェットが表示されない

- クリエイティブの **公開ステータス**を「公開中」に変更（デフォルトは「停止」）
- ブラウザの DevTools コンソール / Network タブを確認
- CSP で `tag.stg.jolt.me` がブロックされていないか確認（このページでは緩いので発生しないはず）

### マジックリンク認証後にウィジェットが未認証のまま

ウィジェットの認証チェックは 1セッションにつき 1 回しか走らない仕様。
**「sessionStorage プローブをリセットしてリロード」**（Section 5）を実行してください。

### タグの送信結果が見えない

タグはレスポンスを返さない fire-and-forget 設計。
DevTools → Network タブで `lead-status` / `tag/events` のリクエスト/レスポンスを確認してください。
（サーバーAPI Section 4b は fetch でレスポンスを画面表示します）

### カスタムイベントが記録されない

- `event_name`(slug) がベンダーダッシュボードの「コンバージョンイベント定義」と一致しているか確認（未定義 slug はエラー）。
- ブラウザタグ（Section 4）は帰属 Cookie `jolt_referral` が無いと帰属不能で処理終了します。
- サーバーAPI（Section 4b）は `X-Jolt-API-Key` が正しいか、`external_source` / `external_source_id` が入っているか確認してください。

---

## 注意

- このリポジトリは stg 動作確認のためだけのツールです
- 本番環境の vendor_program_id / public_token を投入しないでください（プリセットは localStorage に平文保存されます）
- 動作確認終了後は archive 推奨
