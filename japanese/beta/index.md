# DatalaiQベータプログラム

あなたがこのメッセージを読んでいるのは、DatalaiQ Betaの早期アクセスグループに参加しているからです!  皆様のご参加に心から感謝いたします。また、フィードバックやバグレポートをお待ちしております。

バグやフィードバックがあれば [support@ppln.co](mailto:support@ppln.co) に報告してください。

## 現在の "ベータの状態"

DatalaiQ 4.2.0のリリースを準備しています。このリリースの主な新機能は、データエクスプローラーで、ポイント＆クリックで簡単にデータを遊べるようになります。

### 望まれる試験

このスプリントのテスト希望事項（優先順位順）

* データエクスプローラー - できるだけ多くの異なるデータソースで試してみてください。フィルタの追加、フィルタの削除、他のクエリモードへのピボット。
* クエリスタジオ - クエリ編集ボックスの使い勝手や堅牢性を確認し、多くのクエリやタグの間を移動し、他のレンダラーで新しいフォーマットを確認します。
* タグアクセラレーター - [特定のタグアクセラレーション定義](#!configuration/accelerators.md#Accelerating_Specific_Tags) を追加

## インストールとアップデート

私たちは、このビルドが皆様の使用とテストのために利用可能になったことを大変うれしく思っています。私たちは新しいUbuntuリポジトリとDockerイメージを作成しました。安定版からベータ版への切り替えは、aptのソースリポジトリを変更することで行えます（ゼロからインストールする場合は、私たちのクイックスタート手順書を参照してください）。

### アップグレードする:
`etc/apt/sources.list.d/gravwell.list` ファイルを編集して、`https://update.gravwell.io/debian/` を `https://update.gravwell.io/debianbeta/` に置き換えてください。その後、`apt update` と `apt upgrade` をすると、新しいリリースになるはずです。

### ゼロからのインストール:

```
# 署名鍵を取得する
apt install apt-transport-https gnupg wget
wget -O /usr/share/keyrings/gravwell.asc https://update.gravwell.io/debian/update.gravwell.io.gpg.key

# ベータレポジトリを追加
echo 'deb [ arch=amd64 ] https://update.gravwell.io/debianbeta/ community main' | sudo tee /etc/apt/sources.list.d/gravwell.list

# パッケージインストール
apt update
apt install gravwell
```

### Docker

Dockerイメージは [datalaiq/beta](https://hub.docker.com/r/gravwell/beta) で公開されています。ドキュメントにあるどのDockerコマンドでも、`datalaiq/datalaiq`を`datalaiq/beta`に変更すれば、"そのまま動く "はずです。


## 謝辞

DatalaiQ がもたらす新しい機能に、私たちはとても期待しています。ベータ・プログラムにご興味をお持ちいただき、ご参加ありがとうございました。皆様のご協力なくしては実現できませんでした。

フィードバック、バグレポート、そして特に新しいツールで作ったクールなものを見せてください。
