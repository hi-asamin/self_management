# RDS構築
## RDSの作成準備
### セキュリティグループの作成（EC2）
* 必要なところからしかアクセスできないように限定する
* ルールの追加のソースは個別のIPを指定するとサーバを複数台運用すると増えるためセキュリティグループを設定する
* セキュリティグループの文字列を入力して候補が出力されない場合は、セキュリティグループのグループIDをコピーして貼り付ける

### DBサブネットグループの作成
* マルチAZを構築するためのもの

### DBパラメータグループの作成
* 適応タイプはdynamicだと動的に変更されるが、staticだとインスタンス起動後に設定が反映される

### DBオプショングループの作成


## RDSの作成
### DBエンジン
* バックアップは30日がおすすめ
* バックアップ設定時刻はUTC（世界標準時刻：日本-9時間）で設定する
* 手動で取得したスナップショットは自動削除されず放置しておくと多少料金も発生するので不要になったら削除する
* 費用は浮かせられるが、停止に時間がかかる可能性と、起動時にAWS常にRDSインスタンスのリソースがなければ起動できないので、本番では停止しない方がいい

### 本番環境
### DB詳細の指定
### [詳細設定]の設定

## DB作成
* create database aws_study_infra default character set utf8 collate utf8_general_ci;
* create user 'aws_study_infra'@'%' identified by 'password';
* grant all on aws_study_infra.*TO 'aws_study_infra'@'%';
* flush privileges;

## wordpress
* yumに5.4以降のphpがないためamazon-linuxのパッケージマネージャを利用してphpをインストール
amazon-linux-extras install -y php7.2

* phpのライブラリをインストール
yum install -y php php-mbstring

* wordpressをインストール
wget https://ja.wordpress.org/latest-ja.tar.gz

* wordpressを解凍
tar xzvf latest-ja.tar.gz

* apacheが読み込めるディレクトリに再配置する
cp -r * /var/www/html/

* 所有者を変更する
chown apache:apache /var/www/html/ -R

* apacheを再起動する
systemctl restart httpd.service
