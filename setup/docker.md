# Docker セットアップ

目的: 一貫した開発環境をコンテナで再現。

## 前提
- Docker Desktop または Docker Engine が利用可能

## 手順
1. Docker のインストール
2. docker-compose の準備
3. (Linux) 権限設定: ユーザーを docker グループへ追加

## 動作確認
```bash
docker run --rm hello-world
```

## ベストプラクティス
- .dockerignore を用意
- イメージを小さく保つ (multi-stage build)
- ボリューム/ネットワークの適切な分離

## 参考リンク
- https://docs.docker.com/
