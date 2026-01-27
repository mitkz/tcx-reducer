# TCX ファイル間引きツール

GPSトラックポイントを間引いてTCXファイルのサイズを削減するWebアプリケーションです。

## 概要

このツールは、TCX（Training Center XML）ファイル内のトラックポイント（位置情報記録）を指定した数だけスキップして間引き、ファイルサイズを削減します。GPS機器から出力される詳細なトラックデータのファイルサイズを小さくしたい場合に便利です。

## 使い方

### Vercelでの使用（推奨）

デプロイ済みのアプリケーションをブラウザで開いて使用できます。

### ローカルでの使用

1. `index.html` をブラウザで開く
2. TCXファイルをアップロード（クリックまたはドラッグ&ドロップ）
3. 間引き数を指定（例: 2 = 2レコードごとに1つ残す）
4. 「間引いて保存」ボタンをクリック
5. 間引かれたファイルがダウンロードされます

## 機能

- ✅ ドラッグ&ドロップ対応
- ✅ ファイルサイズとレコード数の表示
- ✅ 間引き処理の即時実行
- ✅ レスポンシブデザイン（スマートフォン対応）
- ✅ クライアントサイドで完結（サーバー不要）
- ✅ セキュリティ対策実装済み

## 間引きの仕組み

間引き数を `n` とすると、`n+1` レコードごとに1つのレコードを残します。

例:
- 間引き数 = 1: 元のレコード数の50%を保持
- 間引き数 = 2: 元のレコード数の33%を保持
- 間引き数 = 3: 元のレコード数の25%を保持

## 制限事項

- ファイルサイズ: 最大10MB
- 間引き数: 1〜100の範囲
- 対応形式: TCX (Training Center XML)

## TCXファイルとは

TCX（Training Center XML）は、Garminなどのフィットネス機器で使用されるXML形式のファイルフォーマットです。GPS位置情報、心拍数、ケイデンスなどのトレーニングデータを含みます。

## セキュリティ対策

このアプリケーションには以下のセキュリティ対策が実装されています：

- ✅ ファイルサイズ制限（10MB）
- ✅ ファイル名の検証（パストラバーサル対策）
- ✅ ファイルタイプの検証
- ✅ XMLパースエラーハンドリング
- ✅ 入力値の範囲チェック
- ✅ HTMLエスケープ処理
- ✅ クライアントサイド処理（サーバー送信なし）

## ブラウザ要件

- モダンブラウザ（Chrome, Firefox, Safari, Edge の最新版）
- JavaScript 有効

## ライセンス

このプロジェクトは自由に使用できます。

## サンプルTCXファイルの作成

テスト用のサンプルTCXファイルを作成する場合は、以下のような内容で `.tcx` 拡張子で保存してください：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TrainingCenterDatabase xmlns="http://www.garmin.com/xmlschemas/TrainingCenterDatabase/v2">
  <Activities>
    <Activity Sport="Running">
      <Lap StartTime="2024-01-01T10:00:00Z">
        <Track>
          <Trackpoint>
            <Time>2024-01-01T10:00:00Z</Time>
            <Position>
              <LatitudeDegrees>35.6812</LatitudeDegrees>
              <LongitudeDegrees>139.7671</LongitudeDegrees>
            </Position>
            <AltitudeMeters>10.0</AltitudeMeters>
          </Trackpoint>
          <Trackpoint>
            <Time>2024-01-01T10:00:05Z</Time>
            <Position>
              <LatitudeDegrees>35.6813</LatitudeDegrees>
              <LongitudeDegrees>139.7672</LongitudeDegrees>
            </Position>
            <AltitudeMeters>10.5</AltitudeMeters>
          </Trackpoint>
          <!-- 以下、Trackpointを繰り返し -->
        </Track>
      </Lap>
    </Activity>
  </Activities>
</TrainingCenterDatabase>
```

## トラブルシューティング

### ファイルが読み込めない
- ファイルが有効なTCX形式か確認してください
- ファイルサイズが10MB以下か確認してください

### 間引きがうまくいかない
- 間引き数が1〜100の範囲内か確認してください
- 元のファイルにTrackpointが含まれているか確認してください

### ダウンロードできない
- ブラウザのポップアップブロックを無効にしてください
- ブラウザのダウンロード設定を確認してください
