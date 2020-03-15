# Kubernetes
## コンテナオーケストレーション

* 複数コンテナの管理
* 自動水平スケール
* 監視・自動デプロイ
* ローリングアップデート

## Master
* Nodeを制御するマシン。kubectlから命令を与えるとNodeに指示を出す
* クラスタ全体に関わる決定を行う（クラスタ内のどのNodeも実行できる）
* 以下のコンポーネントを保有する

### kube-apiserver
kubernetes APIを外部に提供するコンポーネント
→kubernetesの制御を行う

### etcd
一貫性、高可用性を持ったKVSで、全kubernetesクラスター情報の永続化に使われる。
→kubernetesのデータストアとして利用する場合は、必ずデータのバックアッププランを作成する必要がある

### kube-scheduler
Podを監視し、Nodeへの割当を行う。

### kube-controller-manager
Replication Controllerなどの各種コントローラーを起動し管理するマネージャー

### cloud-controller-manager
基盤となるクラウドプロバイダーと対話するコントローラーの実行。リリース1.6で追加された。
→ノード、経路、サービス、ボリュームなどの作成、更新、削除を実施する

## Node
稼働中のPod管理やkubernetesの実行環境を提供する

### kubelet
Podを起動し管理するエージェント(Nodeのメイン処理)
→podSpecのセットを取得し、その数が正常起動されている状態を保つ。

### kube-proxy
ネーミングルールに基づき内部や外部のネットワークとPodへのネットワーク通信を可能にする

## Pod
* Kubenetesで扱う最小単位
* 同一環境で動作するDockerコンテナの集合（1つでもいい）

## レプリカセット
* Podの集合でPodをスケールできる

## デプロイメント
* レプリカセットの集合（レプリカセットの世代管理）

## サービス
* 外部公開、名前解決、L4ロードバランサ

### サービスの種類
* ClasterIP: クラスタネットワーク内にIPアドレスを公開し、名前指定でPodへ到達できるようにする
* NodePort: ClasterIPに加えて、Nodeのポートにポートマッピングして受付できるようにする
* LoadBalancer: NodePortに加えて、クラウドプロバイダーのLBを利用してサービスを公開する。
* ExternalName: 外部サービスに接続

## ConfigMap
* kubernetes上で設定情報を集約するリソース

## Secret
* 機微情報を扱う（Base64でエンコードされた文字列）

## Ingress
* L7ロードバランサ（URLでサービスを切り替える）
* マニフェスト作成時はデフォルトだとSSLのリダイレクトが走る
→nginx.ingress.kubernetes.io/ssl-redirect: "false" で無効化

## kubectl
kubernetesのCLIクライアント（kubernetesのコマンド）

## kube-controller-managerで管理されるコントローラー
* ノードコントローラー：ノードがダウンした場合の通知と対応を担当する
* レプリケーションコントローラー：システム内の全レプリケーションコントローラーオブジェクトについて、Podの数を正しく保つ役割
* エンドポイントコントローラー：エンドポイントオブジェクトの紐付け（ServiceとPodの紐付け）
* サービスアカウントとトークンコントローラー：新規の名前空間に対して、デフォルトアカウントとAPIアクセストークンを作成する

## マニフェスト
* YAML形式で記述する
* 種別、メタデータ、コンテナ定義の3構成
* podはnameとnamespaceの組み合わせで一意になるようにする（namespaceは指定しない場合はdefaultになる）
* labelsは任意でkey, value形式で設定する
* specはpodに含むコンテナを指定する
