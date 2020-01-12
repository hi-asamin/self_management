# CloudWatchでシステム監視
## 監視の目的
* すぐに障害発生を確認できるようにする
* 復旧にすぐに取りかかれるようにする

## 具体的な監視内容
1. 「正常な状態」を監視項目＋正常な結果の形で定義する
1. 「正常な状態」でなくなった際の対応方法を監視項目ごとに定義する
1. 「正常な状態」であることを継続的に確認する
1. 「正常な状態」でなくなった場合には通知が来るようにし、すぐ「正常な状態」に復旧させる

## 死活監視
* 正常にシステムが動作しているかを確認する

## メトリクス監視
* パフォーマンスを定量的に確認する
* 指標を決め、指標が閾値以上・以下となっているかを把握

## 監視項目を決める際のポイント
### システムや利用状況はかわるので、足りない監視を都度足していく
* 項目が多過ぎると、監視疲れする
* システムも利用状況も変わるので、都度監視項目を調整すればOK

### 最初は基本的な要素でOK
* CPU、メモリ、ディスク、ネットワークの使用率・枯渇
→これらの情報を確認できれば、障害発生時に何時が起点なのかを把握できる