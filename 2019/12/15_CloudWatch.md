# cloudwatch
AWSサービスの監視やモニタリングができる監視サービス(運用監視のマネージドサービス)

## 概要
* AWSサービスのメトリクス（リソースの状況）を監視する
* メトリクスに対して閾値を登録し、その条件を満たしたら通知する（アラーム発生）
EC2 -> CloudWatch -> SNS -> メール

## SNS
* publisher: メッセージ送信
* Amazon SNS: トピックを作成・管理
* subscriber: 通知を受信
* publisherとsubscriberを疎結合にすることで、publisherの宛先に変更があってもSNSのみ対応すれば良くなる

## CPU負荷を確認する方法
* yes > dev/null &
→「yes」はターミナル上に永遠にyを標準出力するコマンド
