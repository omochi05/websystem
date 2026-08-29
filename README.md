# Web掲示板サービス　構築手順書
## 1.概要
構築するweb掲示板では、テキストの投稿、投稿日時・IDの表示、
5MB以下の画像のアップロードと表示を行えるようにする
- PHP / nginx / MySQL / Dockerを使用

## 2.今回のファイル構成
```text
Dockertest/
├── Dockerfile
├── compose.yml
├── nginx/
│   └── conf.d/
│       └── default.conf
└── public/
    └── zenkitest.php
```

## 3.Dockerの準備

### Dockerのインストール

```bash
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
```

ec2-userがsudoなしでDockerを操作できるようにする
```bash 
sudo usermod -a -G docker ec2-user 
```

一度SSHを切断して再ログインする。

再ログイン後、
```bash
docker ps
```
を実行する
エラーが出なければDockerの準備完了

## 4.Docker Composeをインストールする

### Docker Composeのインストール
```bash
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v5.1.2/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```
インストール確認
```bash
docker compose version
```

### buildxのインストール
```bash
mkdir -p ~/.docker/cli-plugins
ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')
BUILDX_URL=$(curl -s https://api.github.com/repos/docker/buildx/releases/latest | grep "browser_download_url.*linux-$ARCH" | cut -d '"' -f 4)
curl -L $BUILDX_URL -o ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
```
インストール確認
```bash
docker buildx version
```

## 5.作業ディレクトリ作成
```bash
mkdir Dockertest
cd Dockertest
```

## 6.compose.ymlの作成
ここには主に
web   → nginx
php   → PHP-FPM
mysql → MySQL
の3サービスを定義する

ファイルの作成
```bash
vim compose.yml
```
中身はこちら→https://github.com/omochi05/websystem/blob/main/compose.yml

## 7.nginxの設定を作る
```bash
mkdir nginx
mkdir nginx/conf.d
```

80番ポートでWebアクセスを受ける
↓
/var/www/public を公開する
↓
.phpならPHPコンテナへ処理を渡す

ファイル作成
```bash
vim nginx/conf.d/default.conf
```
中身はこちら→https://github.com/omochi05/websystem/blob/main/nginx/conf.d/default.conf

## 8.PHP用のDockerfileを作る
PHPからMySQLへ接続するために必要な設定などをここで行う。
```bash
vim Dockerfile
```
中身はこちら→https://raw.githubusercontent.com/omochi05/websystem/refs/heads/main/Dockerfile

## 9.PHPファイルの作成
ここに自分が使う掲示板PHPを配置する
```bash
mkdir public
vim public/zenkitest.php
```
中身はこちら:https://github.com/omochi05/websystem/blob/main/public/zenkitest.php

## 10.コンテナを起動する
```bash
docker compose up -d
```
起動確認
```bash
docker compose ps
```
EC2のセキュリティグループでHTTP(TCP/80)のインバウンド
通信が許可されていることを確認する。
その後サービスの起動とWebのPORTSが
```text
0.0.0.0:80->80/tcp
```
になっているか確認

## 11.MySQLの掲示板テーブルを作成
MySQLに入る
```bash
docker compose exec mysql mysql example_db
```
テーブルを作成する
```sql
CREATE TABLE `bbs_entries` (
    `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `body` TEXT NOT NULL,
    `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);
```
画像ファイル名を保存するカラムを追加
```sql
ALTER TABLE `bbs_entries`
ADD COLUMN image_filename TEXT DEFAULT NULL;
```
終わったら、
```sql
exit
```
これでDB側の準備終了

## 12.動作確認
ブラウザにアクセス
```text
http://EC2のパブリックIP/zenkitest.php
```
実際に
テキストだけ投稿する
IDが表示されるか見る
投稿日時が表示されるか見る
画像付きで投稿する
画像が表示されるか見る
ページを更新して投稿が残っているか見る
か確認する

## 再構築確認
新しいAmazon Linuxインスタンスを作成し、本手順書の手順に沿って再構築する。

再構築後、「12.動作確認」と同じ項目を確認する。
