# 🐳 WSL (Ubuntu) に Docker を CLI でインストールする手順
この手順は、**Docker Desktop を使わず、CLI だけで WSL 環境に Docker を導入する**ためのもの。  
必要最低限のツールを入れ、`docker` コマンドでコンテナを操作できるようにします。

## このセットアップの目的
- WSL2 環境で Docker を CLI から操作したい
- GUI不要でCLI中心の開発スタイルが好み
- Docker Desktop を使いたくない or 環境にインストールできない
- 軽量なコンテナ開発・検証をしたい
- DevContainer や docker-compose を使って複数プロジェクトを管理したい


## 1. WSL 環境を用意
WSL2＋Ubuntu がインストールされている状態を前提とする。  
Ubuntu にログインした状態で作業します。

## 2. パッケージリスト更新 & アップグレード
Docker のインストールに必要な依存関係を安定させ、最新のシステム状態に整える為に以下のコマンドを実行。
```bash
sudo apt update
sudo apt upgrade -y
```

- `apt update`:  
  パッケージリストを最新に更新。これにより、APT が新しいバージョンを認識できるようになる。

- `apt upgrade -y`:  
  既存のパッケージをすべて最新にアップグレード。  
  `-y` は「すべて Yes で進める」という意味で、対話的な確認をスキップする。

## 3. 依存パッケージのインストール
Docker リポジトリ追加や鍵の取得に必要な最低限のパッケージを揃える為に以下のコマンドを実行。
```bash
sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```
- `ca-certificates`: HTTPS 通信に必要な認証局証明書を提供  
- `curl`: 外部からファイル（ Docker 公式 GPG 鍵など）を取得するために使用  
- `gnupg`: GPG 鍵の処理に必要な暗号化ツール  
- `lsb-release`: Ubuntu のバージョン情報を取得できるようにする

## 4. Docker 公式 GPG 鍵を追加
APT が Docker リポジトリから安全にパッケージを取得できるように、Docker の公開鍵を追加する。
以下のコマンドを実行。
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
- `mkdir -p /etc/apt/keyrings`: 鍵を保存するディレクトリを作成（既にある場合は無視）  
- `curl ... | sudo gpg --dearmor -o`: Docker の公開鍵をダウンロードし、APT で使える形式に変換して保存  

正常に処理されているかを一応確認したい場合、以下のコマンドを実行。
```bash
ls -l /etc/apt/keyrings/docker.gpg
```
以下のような出力があれば成功している。
```bash
-rw-r--r-- 1 root root 1234 Aug  2 12:34 /etc/apt/keyrings/docker.gpg
```

> 💡 GPG（GNU Privacy Guard） は、データの暗号化とデジタル署名を行うためのツールです。GPG 鍵は、この暗号化システムで使用される「デジタルな鍵」のこと。

---

## 5. Docker リポジトリを APT に追加
Docker を公式リポジトリからインストールできるようにするため、以下のコマンドを実行。
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

- `dpkg --print-architecture`: アーキテクチャ（例: amd64）を取得  
- `lsb_release -cs`: Ubuntu のコードネーム（例: jammy、focal）を取得  
- `/etc/apt/sources.list.d/docker.list`: Docker 公式のパッケージリストを APT に登録

## 6. Docker エンジン本体のインストール
Docker を CLI ベースで使うために必要なすべての主要コンポーネントをインストール。
以下のコマンドを実行。
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- `docker-ce`: Docker Community Edition（Docker 本体）  
- `docker-ce-cli`: `docker` コマンドを使うための CLI ツール  
- `containerd.io`: コンテナの実行エンジン（低レベルランタイム）  
- `docker-buildx-plugin`: Docker BuildKit の拡張  
- `docker-compose-plugin`: `docker compose` サブコマンドを使うためのプラグイン  

## 7. Docker の動作確認
Docker が正常に動いていることを確認するために、以下のコマンドを実行。
```bash
sudo docker version
```
- クライアントとサーバー（デーモン）のバージョン情報が表示されれば、インストール成功。


## 8. 一般ユーザーとして Docker を使う（sudo なしで実行）

```bash
sudo usermod -aG docker $USER
```

- `$USER` を `docker` グループに追加することで、`sudo` なしで `docker` コマンドを実行できるようにします。

- `$USER` は 自動的に WSL にログインしているユーザー名に展開されるシェル変数。なので WSL にログインしているユーザー名に置き換えたりせず`$USER`のまま実行して問題ない。

🚨 **重要**：この変更は即時反映されない。  
下記を行ってください：

```bash
exit       # WSLセッションを終了し、再ログインする
```
または、ターミナルを閉じて WSL を再起動する。


## 9. 最後の確認
```bash
docker info
```
Docker デーモンの情報が表示されれば、すべてのセットアップが完了。

## おまけ：軽量なコンテナで動作確認
試しに軽量の公式コンテナで動作確認したい場合は、以下コマンド実行してみる。
```bash
docker run hello-world
```
「Hello from Docker!」が表示されれば成功。
即終了するコンテナなので実行後何もしなくてもいいが、削除しておいてもよい。  

### 削除する場合の手順
1. 履歴を確認(`docker ps`では稼働中のみ表示される為、表示されない)  
さっきのコンテナが表示される。
```bash
docker ps -a # 履歴を確認
```

2. コンテナを削除
```bash
docker rm $(docker ps -a -q) # すべてのコンテナを削除
```
3. イメージを削除
```bash
docker rmi hello-world # hello-world イメージを削除
```
※ 先にコンテナ削除 → その後イメージ削除 が基本順序。
イメージは、そのイメージ由来のコンテナが存在していると削除できません（依存関係があるため）。
