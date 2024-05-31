# 処理の概要

1. EventBridge => Producer Lambda で ControlTowerの更新イベントを受け取って Consumer Lambda を起動（厳密にはSQS をキックする）
2. Consumer Lambda で各アカウント （xリージョン）毎にAPI リクエストを実行してconfig の設定を変更

# デプロイ方法

1. ローカルでZipファイル(2個)を作る
   - 各フォルダ内で以下のコマンドを実行して Zip ファイルをつくる
   - UIでのフォルダ圧縮だとZipがバグるのでコマンドじゃないとダメ
   - 実行するコマンド
     - `zip -rq ct_configrecorder_override_consumer_v2.zip .`
     - `zip -rq ct_configrecorder_override_producer.zip .`
2. 作成したZipを root アカウント の s3 にアップロード
   - バケット名は何でもOK
   - 作成したバケット配下に /config のディレクトリを作成
   - `/config` ディレクトリ配下にZipを2つ格納
3. root アカウントで CloudFormation の Stack を新規作成
   - ステップ1
      - 既存のテンプレートを選択 => テンプレートファイルのアップロード
      - template.yaml を 指定
   - ステップ2
     - スタック名： ControlTowerConfigCustomization
     - パラメータ:
       - ConfigRecorderExcludedResourceTypes: カンマ区切りで除外したいソースタイプを記載
       - LambdaSourceCodeS3Bucket: Zip ファイルをアップロードしたS3バケット名を記載
       - TargetAccounts: 対象アカウントを配列形式で記載
   - ステップ3
     - 全て空欄でOK（Roleなど）
   - ステップ4
     - 「送信」を押すと各種リソースが展開される
     - 続いて ProducerLambda、ConsumerLambda が 実行される

# 参考情報

- 記事: <https://aws.amazon.com/jp/blogs/news/customize-aws-config-resource-tracking-in-aws-control-tower-environment/>