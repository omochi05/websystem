# Web掲示板サービス　構築手順書
## 1.概要
　 構築するweb掲示板では、テキストの投稿、投稿日時・IDの表示、
　 5MB以下の画像のアップロードと表示を行えるようにする
   - PHP / nginx / MySQL / Dockerを使用

## 2.今回のファイル構成
```
Dockertest/
|
|---Dockerfile
|
|---compose.yml
|
|---nginx/
|    |
|    |___conf.d
|       |
|       |___default.conf
|
|___public/
     |
     |___zenkitest.php

```

## 3.Dockerの準備

### Dockerのインストール

```
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
```

### Dockerサービスの起動
Docker compose up -d

docker compose upを実行すると、compoe.ymlに定義したnginx、PHP、MySQLの各コンテナが作成され、Web掲示板を動作させるための環境が構築される。
