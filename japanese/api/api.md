# API

このセクションではGUIとフロントエンドのWebサーバー間で使用されるAPIについて記載します。

APIの大部分はRESTfulです。例外は、検索からのデータの起動と監視に関連するデータ交換と転送の性質上、websocket を使用する検索 API です。

## 基本APIs

* [ログイン](login.md)
* [ユーザー情報](userprefs.md)
* [ユーザーアカウント操作](account.md)
* [ユーザーグループ操作](groups.md)
* [通知](notifications.md)
* [検索（クエリ）操作](searchctrl.md)
* [検索（クエリ）結果のダウンロード](download.md)
* [検索（クエリ）履歴](searchhistory.md)
* [ログ](loglevel.md)
* [データインジェスト](ingest.md)
* [その他のAPI](misc.md)
* [システム管理](management.md)

## DatalaiQ内のオブジェクト

DatalaiQにはユーザーが自由に作成可能なオブジェクトがあります。それらのAPIについて以下に記載します。

* [自動抽出](extractors.md)
* [ダッシュボード](dashboards.md)
* [キット](kits.md)
* [マクロ](macros.md)
* [プレイブック](playbooks.md)
* [リソース](resources.md)
* [スケジュール検索](scheduledsearches.md)
* [検索（クエリ）ライブラリ](searchlibrary.md)
* [テンプレート](templates.md)
* [アクショナブル](actionables.md)
* [ユーザーファイル](userfiles.md)

## 検索（クエリ）と統計

[Webソケット検索（クエリ）](websocket-search.md)

[検索（クエリ）への再接続](websocket-search-attach.md)

[Rendererとの接続](websocket-render.md)

## System統計情報

システム統計情報も接続にWebソケットを使用します。システム統計情報には一般的なクラスターの状態を監視するのに必要な情報が全て含まれています。

[システム統計情報Websocket](websocket-stats.md)

その他の統計情報はRESTAPIにより取得できます。

[システム統計情報（REST API）](stats-json.md)

## テストAPI

システムにはWebサーバーが正常に起動し、動作しているかを確認するためのテストAPIを `/api/test` に備えています。テストAPIに認証は必要なく、ステータス200と空のbodyを返します。
