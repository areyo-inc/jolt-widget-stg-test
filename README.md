# jolt-widget-stg-test

Jolt リファラー機能の stg 環境動作確認用テストページ。

## 使い方

公開URL末尾に `?program={vendor_program_id}` を付けてアクセスする：

```
https://areyo-inc.github.io/jolt-widget-stg-test/?program=01XXXXX...
```

3つの設置方式（インライン / 要素クリックポップアップ / フローティング）が同一ページに展開されるので、それぞれの挙動を確認できる。

## 注意

- このリポジトリは stg 動作確認のためだけの一時的なものです。動作確認終了後は archive 推奨。
- 中身は静的 HTML 1 ファイルのみ。
- ウィジェットスクリプトは `https://tag.stg.jolt.me/widget/v1/widget.js` を参照。
