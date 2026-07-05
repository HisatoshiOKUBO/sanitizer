# 踏み込み消毒槽　交換記録アプリ（Supabase REST API 版）

PHPサーバーは不要です。`index.html` 単体を GitHub Pages / Cloudflare Pages 等の
静的ホスティングに置き、データの読み書きはブラウザから直接 Supabase の REST API
（PostgREST）を呼び出す構成です。

## ファイル構成
```
sanitizer_app/
├── index.html   このファイルだけで完結するフロントエンド
└── schema.sql    Supabase の SQL Editor で1回だけ実行するテーブル定義
```

## セットアップ手順

### 1. Supabaseでテーブルを作成
1. Supabaseダッシュボード → 対象プロジェクト → **「SQL Editor」**
2. `schema.sql` の中身を貼り付けて実行
   - `locations`（設置場所）・`records`（交換記録）の2テーブルが作成されます
   - 事務所・薬品庫がデフォルトで登録されます
   - RLS（Row Level Security）を有効にし、anon keyでの読み書きを許可するポリシーも同時に設定されます

### 2. Project URL と anon key を確認
Supabaseダッシュボード → **「Project Settings」→「API」** を開き、以下をコピーします。
- **Project URL**（例：`https://xxxxxxxxxxxxxxxxxxxx.supabase.co`）
- **anon public** キー（長い文字列。`service_role` キーは絶対に使わないでください）

### 3. index.html を書き換え
ファイル冒頭付近の以下の2行を、確認した値に書き換えます。

```js
const SUPABASE_URL = 'https://xxxxxxxxxxxxxxxxxxxx.supabase.co';
const SUPABASE_ANON_KEY = 'ここにanon public keyを貼り付け';
```

### 4. GitHub / Cloudflare Pages にアップロード
`index.html` をそのままリポジトリに置いてデプロイすれば完了です。サーバー側の設定は一切不要です。

## 動作の仕様メモ

- **場所の追加・削除**：設定タブから行えます。削除しても過去の記録は場所名がそのまま残るため履歴は消えません。
- **実施日**：カレンダーから選択できます。未選択の場合は本日の日付が自動的に入ります。
- **重複記録**：同じ日・同じ場所の記録が既にある場合、上書き確認のダイアログが表示されます。
- **訂正**：一覧タブで記録をタップすると、日付・場所を後から修正、または削除できます。
- **出力**：一覧タブの「出力」ボタンから、確認日・確認者を入力したうえで PDF / CSV / Excel を選択して出力できます。すべてブラウザ内で生成されます（サーバー処理なし）。
  - PDF は印刷用ページが新しいタブで開くので、ブラウザの印刷機能から「PDFに保存」してください。
  - CSV / Excel はそのままファイルとしてダウンロードされます（Excel生成には SheetJS を利用しています）。

## セキュリティに関する注意

anon key はブラウザのソースコードに直接書き込まれるため、**このURLとキーを知っている人は誰でもデータの読み書きができます**。社内・関係者のみが使う想定であれば通常問題ありませんが、次の点にご留意ください。

- index.html を公開リポジトリに置く場合、anon key も公開されることになります。非公開リポジトリを推奨します。
- より厳密にアクセス制限したい場合は、Supabase Authenticationを導入してログイン制にする、またはRLSポリシーを絞り込む対応が可能です。必要であればお申し付けください。
