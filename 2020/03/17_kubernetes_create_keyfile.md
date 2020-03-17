# keyfileの作成

```bash
# 1024文字のランダム文字列を生成しbase64で暗号化
# 改行コードを削除
# 必要サイズにカット（1024）
# その結果をkeyfileに出力
openssl rand -base64 1024 | tr -d '¥r¥n' | cut -c 1-1024 > keyfile
```

### rootユーザーのユーザー名、パスワード、keyfileの3つsecretfileを作成する

```bash
kubectl create secret generic mongo-secret \
--from-literal=root_username=admin \
--from-literal=root_password=Passw0rd \
--from-file=./keyfile
``

### 作成したsecretをyaml形式で取得する
```bash
kubectl get secret/mongo-secret -o yaml
````
