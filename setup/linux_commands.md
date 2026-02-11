# Linuxコマンド集

Linuxコマンドをまとめたチートシート。<br>

<div style="height:30px;"></div>

---

## ファイル・ディレクトリ操作

### 基本操作
```bash
# カレントディレクトリ表示
pwd

# ディレクトリ移動
cd /path/to/directory    # 絶対パス指定
cd ~/Documents           # ホームディレクトリ配下
cd ..                    # 一つ上のディレクトリ
cd -                     # 直前のディレクトリに戻る

# ファイル・ディレクトリ一覧
ls                       # 基本
ls -l                    # 詳細表示（権限、サイズ、日時）
ls -a                    # 隠しファイル含む
ls -lh                   # サイズを人間に読みやすく（K, M, G単位）
ls -lah                  # 全部盛り（よく使う）
ls -lt                   # 更新日時順でソート（新しい順）
ls -ltr                  # 更新日時順でソート（古い順）

# ディレクトリ作成
mkdir mydir              # 単一作成
mkdir -p path/to/deep/dir  # 親ディレクトリも含めて作成（超重要！）

# ファイル作成
touch newfile.txt        # 空ファイル作成、または更新日時だけ変更

# コピー
cp file.txt backup.txt   # ファイルコピー
cp -r dir1 dir2          # ディレクトリごとコピー（-r必須）
cp -p file.txt backup.txt  # タイムスタンプや権限を保持してコピー

# 移動・リネーム
mv old.txt new.txt       # リネーム
mv file.txt /path/to/    # 別ディレクトリへ移動
mv *.txt backup/         # ワイルドカードで複数ファイル移動

# 削除
rm file.txt              # ファイル削除
rm -r dir                # ディレクトリごと削除（-r必須）
rm -rf dir               # 強制削除（確認なし）← 危険！慎重に！
rm -i file.txt           # 削除前に確認（安全）
```

**よくあるミス:**<br>
- `rm -rf /` は絶対にやらない（システム全削除）<br>
- `rm -rf *` も注意（カレントディレクトリ全削除）<br>
- 削除前に `ls` で確認する癖をつける<br>

<div style="height:30px;"></div>

---

## ファイル内容確認・検索

### ファイル内容表示
```bash
# 全体表示
cat file.txt             # 全体を表示
cat -n file.txt          # 行番号付きで表示

# ページング（大きいファイル向け）
less file.txt            # スクロール可能（q で終了）
more file.txt            # 古い方式（less推奨）

# 先頭・末尾表示
head file.txt            # 先頭10行表示
head -n 20 file.txt      # 先頭20行表示
tail file.txt            # 末尾10行表示
tail -n 50 file.txt      # 末尾50行表示
tail -f /var/log/app.log # リアルタイム監視（ログファイル監視に超便利！）
```

### ファイル検索
```bash
# ファイル名で検索
find . -name "*.txt"           # カレントディレクトリ以下の.txtファイル
find /path -name "config.json" # 特定ファイル名検索
find . -name "*.log" -mtime -7 # 7日以内に更新されたログファイル
find . -type f -name "*.txt"   # ファイルのみ（-type d ならディレクトリのみ）
find . -name "*.tmp" -delete   # 検索して削除（慎重に！）

# サイズで検索
find . -size +100M         # 100MB以上のファイル
find . -size -1M           # 1MB以下のファイル

# 実行権限があるファイル
find . -type f -executable
```

### テキスト検索（grep）
```bash
# 基本検索
grep "keyword" file.txt            # ファイル内検索
grep "error" *.log                 # 複数ファイル検索
grep -i "error" file.txt           # 大文字小文字区別しない
grep -n "error" file.txt           # 行番号付きで表示
grep -v "debug" file.txt           # マッチしない行を表示（except）

# 再帰検索（ディレクトリ配下全て）
grep -r "TODO" .                   # カレント以下全検索
grep -r "function" src/            # src配下全検索
grep -rn "import" .                # 行番号付き再帰検索

# 前後の行も表示
grep -A 3 "error" file.txt         # マッチ行+後3行
grep -B 2 "error" file.txt         # マッチ行+前2行
grep -C 2 "error" file.txt         # マッチ行+前後2行

# パイプと組み合わせ
ps aux | grep python               # プロセス一覧からpython検索
cat access.log | grep "404"        # ログから404エラー抽出
history | grep git                 # コマンド履歴から検索
```

**実務tip:**<br>
- `grep -rn "keyword" .` が最もよく使う<br>
- ログ調査では `grep -C 5` で前後の文脈も見る<br>

<div style="height:30px;"></div>

---

## 権限管理

### 権限の見方
```bash
ls -l
# -rw-r--r--  1 user group  1234 Feb 11 10:00 file.txt
# ↑
# - rw- r-- r--
# │ │   │   └─ その他ユーザーの権限
# │ │   └───── グループの権限
# │ └─────── 所有者の権限
# └────────── ファイルタイプ（- はファイル、d はディレクトリ）
#
# r = read（読み込み）= 4
# w = write（書き込み）= 2
# x = execute（実行）= 1
```

### chmod（権限変更）
```bash
# 数値指定（よく使う）
chmod 644 file.txt       # -rw-r--r-- （ファイルの基本）
chmod 755 script.sh      # -rwxr-xr-x （実行ファイルの基本）
chmod 600 secret.key     # -rw------- （秘密鍵など、自分だけ読み書き）
chmod 700 ~/.ssh         # drwx------ （.sshディレクトリの推奨）

# よく使う組み合わせ
# 644 = 自分：読み書き、他人：読み込みのみ（設定ファイルなど）
# 755 = 自分：全権限、他人：読み込み+実行（スクリプト、ディレクトリ）
# 600 = 自分だけ読み書き（秘密鍵、パスワードファイル）
# 700 = 自分だけ全権限（プライベートなディレクトリ）

# 記号指定
chmod +x script.sh       # 実行権限を追加（全員に）
chmod -x script.sh       # 実行権限を削除
chmod u+x script.sh      # 所有者(user)に実行権限追加
chmod g+w file.txt       # グループ(group)に書き込み権限追加
chmod o-r file.txt       # その他(others)から読み込み権限削除

# 再帰的に変更
chmod -R 755 directory/  # ディレクトリ配下全て変更
```

### chown（所有者変更）
```bash
# 所有者変更
chown user file.txt           # 所有者のみ変更
chown user:group file.txt     # 所有者とグループ両方変更
chown -R user:group dir/      # ディレクトリ配下全て変更

# 実務例
sudo chown www-data:www-data /var/www/html  # Webサーバー用に変更
```

**実務でよくある:**<br>
- Dockerでマウントしたファイルの権限問題<br>
- Gitでcloneしたスクリプトに実行権限がない → `chmod +x script.sh`<br>
- SSH秘密鍵の権限が緩いと怒られる → `chmod 600 ~/.ssh/id_rsa`<br>

<div style="height:30px;"></div>

---

## プロセス管理

### プロセス確認
```bash
# プロセス一覧
ps aux                   # 全プロセス表示（よく使う）
ps aux | grep python     # python関連プロセスのみ
ps -ef                   # BSD形式（auxと内容は同じ）

# リアルタイム監視
top                      # CPU・メモリ使用率リアルタイム表示（q で終了）
htop                     # topの見やすい版（要インストール）

# プロセスツリー表示
pstree                   # 親子関係が見える
pstree -p                # PID付きで表示
```

### プロセス終了
```bash
# 安全に終了
kill <PID>               # プロセスに終了シグナル送信（SIGTERM）
kill 1234                # PID 1234のプロセスを終了

# 強制終了
kill -9 <PID>            # 強制終了（SIGKILL）
kill -9 1234             # プロセスが応答しない時の最終手段

# 名前で終了
pkill python             # 名前でプロセス終了
pkill -f "celery worker" # コマンドライン全体でマッチ
killall python           # 同名プロセス全て終了（注意！）

# 実務例
ps aux | grep "node server.js" # まずPID確認
kill 5678                      # 該当PIDを終了
```

### バックグラウンド実行
```bash
# バックグラウンドで実行
python script.py &       # 末尾に & でバックグラウンド実行

# フォアグラウンド→バックグラウンド
# Ctrl+Z でサスペンド → bg でバックグラウンド化

# バックグラウンドジョブ確認
jobs                     # ジョブ一覧
fg %1                    # ジョブ1をフォアグラウンドに戻す

# ログアウト後も実行継続
nohup python script.py &         # nohup使用
nohup python script.py > output.log 2>&1 &  # ログファイルに出力
```

<div style="height:30px;"></div>

---

## ネットワーク

### 接続確認
```bash
# ping（疎通確認）
ping google.com          # 応答確認（Ctrl+C で停止）
ping -c 4 google.com     # 4回だけ送信

# ポート確認
netstat -tuln            # リスニングポート一覧
netstat -tuln | grep 8080  # 8080番ポートが開いてるか確認
ss -tuln                 # netstatの新しい版（推奨）
lsof -i :8080            # 8080番ポートを使用中のプロセス確認
```

### HTTPリクエスト
```bash
# curl（APIテスト、ファイルダウンロード）
curl https://example.com                    # GET リクエスト
curl -X POST https://api.example.com        # POST リクエスト
curl -H "Content-Type: application/json" \
     -d '{"key":"value"}' \
     https://api.example.com                # JSON送信

curl -o output.txt https://example.com      # ファイルに保存
curl -O https://example.com/file.zip        # 元のファイル名で保存
curl -L https://example.com                 # リダイレクト追従

# ヘッダー表示
curl -I https://example.com                 # ヘッダーのみ表示
curl -v https://example.com                 # 詳細表示（デバッグ用）

# wget（ファイルダウンロード特化）
wget https://example.com/file.zip           # ダウンロード
wget -c https://example.com/large.zip       # 中断したダウンロード再開
wget -r https://example.com                 # 再帰的にダウンロード
```

### DNS・IP確認
```bash
# IPアドレス確認
ifconfig                 # ネットワーク設定（古い）
ip addr                  # ネットワーク設定（新しい、推奨）
ip a                     # 省略形

# DNS引き
nslookup google.com      # IPアドレス確認
dig google.com           # 詳細なDNS情報
host google.com          # シンプルなDNS確認
```

<div style="height:30px;"></div>

---

## 圧縮・解凍

### tar（最頻出）
```bash
# 圧縮
tar -czvf archive.tar.gz directory/   # ディレクトリを圧縮
tar -czvf backup.tar.gz file1 file2   # 複数ファイルを圧縮

# 解凍
tar -xzvf archive.tar.gz              # 解凍
tar -xzvf archive.tar.gz -C /path/to/ # 指定ディレクトリに解凍

# 中身確認（解凍せずに）
tar -tzvf archive.tar.gz              # アーカイブ内容一覧

# オプションの意味
# c = create（作成）
# x = extract（展開）
# z = gzip（gzip圧縮）
# v = verbose（詳細表示）
# f = file（ファイル名指定）
```

**覚え方:**<br>
- 圧縮: `tar -czvf 作る名前.tar.gz 元ディレクトリ`<br>
- 解凍: `tar -xzvf 解凍する.tar.gz`<br>

### zip
```bash
# 圧縮
zip archive.zip file.txt             # ファイル圧縮
zip -r archive.zip directory/        # ディレクトリ圧縮（-r必須）

# 解凍
unzip archive.zip                    # 解凍
unzip archive.zip -d /path/to/       # 指定ディレクトリに解凍
unzip -l archive.zip                 # 中身確認（解凍せずに）
```

### gzip（単一ファイル圧縮）
```bash
# 圧縮
gzip file.txt            # file.txt.gz が作られ、元ファイルは削除される
gzip -k file.txt         # 元ファイルを保持（keep）

# 解凍
gunzip file.txt.gz       # 解凍
gzip -d file.txt.gz      # 解凍（-d = decompress）
```

<div style="height:30px;"></div>

---

## テキスト処理

### 基本操作
```bash
# 行数・文字数カウント
wc file.txt              # 行数、単語数、バイト数
wc -l file.txt           # 行数のみ
wc -w file.txt           # 単語数のみ

# ソート
sort file.txt            # アルファベット順ソート
sort -r file.txt         # 逆順
sort -n file.txt         # 数値としてソート
sort -u file.txt         # 重複削除してソート

# 重複削除
uniq file.txt            # 連続する重複行削除（sortと併用）
sort file.txt | uniq     # ソート後に重複削除（よく使う）
sort file.txt | uniq -c  # 重複回数も表示

# 列抽出
cut -d',' -f1 file.csv   # CSVの1列目抽出（カンマ区切り）
cut -d' ' -f2,3 file.txt # スペース区切りで2,3列目抽出
```

### sed（置換）
```bash
# 基本置換
sed 's/old/new/' file.txt           # 各行の最初のoldをnewに置換（表示のみ）
sed 's/old/new/g' file.txt          # 全てのoldをnewに（g = global）
sed -i 's/old/new/g' file.txt       # ファイルを直接書き換え（-i）

# 実務例
sed 's/localhost/127.0.0.1/g' config.txt  # 設定ファイル書き換え
sed -i 's/DEBUG/INFO/g' app.py      # コード内の文字列一括置換

# 行削除
sed '3d' file.txt                   # 3行目削除
sed '/pattern/d' file.txt           # パターンにマッチする行削除
```

### awk（列処理）
```bash
# 列抽出
awk '{print $1}' file.txt           # 1列目表示
awk '{print $1, $3}' file.txt       # 1列目と3列目表示

# 区切り文字指定
awk -F',' '{print $1}' file.csv     # CSV（カンマ区切り）の1列目
awk -F':' '{print $1}' /etc/passwd  # コロン区切りの1列目

# 条件付き
awk '$3 > 100' file.txt             # 3列目が100より大きい行
awk '/error/ {print $0}' log.txt    # errorを含む行全体表示

# 実務例
ps aux | awk '{print $2, $11}'      # PIDとコマンド名だけ表示
df -h | awk '$5 > 80 {print $0}'    # ディスク使用率80%超えのみ表示
```

### パイプ組み合わせ例
```bash
# ログから特定エラーを抽出して件数カウント
cat app.log | grep "ERROR" | wc -l

# アクセスログからIPアドレス抽出してランキング
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# プロセスをメモリ使用量順にソート
ps aux | sort -k4 -rn | head -10

# 特定拡張子のファイルサイズ合計
find . -name "*.log" -exec du -k {} \; | awk '{sum+=$1} END {print sum/1024 "MB"}'
```

<div style="height:30px;"></div>

---

## システム情報

### ディスク使用量
```bash
# ディスク全体
df -h                    # ディスク使用量（人間に読みやすい単位）
df -h /                  # ルートパーティションのみ

# ディレクトリごとのサイズ
du -sh directory/        # 指定ディレクトリの合計サイズ
du -sh *                 # カレントディレクトリ配下の各ディレクトリサイズ
du -h --max-depth=1      # 1階層のみ表示
du -sh * | sort -h       # サイズ順ソート

# 大きいファイル探し
du -ah | sort -rh | head -20  # 上位20件表示
```

### メモリ使用量
```bash
# メモリ状況
free -h                  # メモリ使用量（人間に読みやすい単位）
free -m                  # MB単位表示

# スワップ確認
swapon -s                # スワップ領域確認
```

### CPU情報
```bash
# CPU情報
lscpu                    # CPU詳細情報
cat /proc/cpuinfo        # CPU情報の生データ
nproc                    # CPUコア数
```

### OS・カーネル情報
```bash
# OS確認
cat /etc/os-release      # ディストリビューション確認
lsb_release -a           # 同上（Ubuntu/Debianなど）
uname -a                 # カーネル情報全て表示
uname -r                 # カーネルバージョンのみ

# アーキテクチャ
arch                     # x86_64 or arm64 など
uname -m                 # 同上
```

### 稼働時間
```bash
uptime                   # システム稼働時間、ロードアベレージ
who                      # ログイン中のユーザー
w                        # ユーザーと何をしてるか
last                     # ログイン履歴
```

<div style="height:30px;"></div>

---

## よく使うコマンド組み合わせ

### ログ調査
```bash
# リアルタイムでログ監視しながらエラー抽出
tail -f /var/log/app.log | grep --line-buffered "ERROR"

# 直近1時間のエラーログ件数
find /var/log -name "*.log" -mmin -60 -exec grep -c "ERROR" {} \;

# 特定時間帯のログ抽出
grep "2026-02-11 14:" app.log > error_14h.log
```

### ファイル整理
```bash
# 7日以上前のログファイル削除
find /var/log -name "*.log" -mtime +7 -delete

# 空のディレクトリ削除
find . -type d -empty -delete

# 特定拡張子のファイル一括リネーム
for file in *.txt; do mv "$file" "${file%.txt}.md"; done
```

### ディスク容量調査
```bash
# ディスク使用率トップ10
du -ah / 2>/dev/null | sort -rh | head -10

# 1GB以上のファイルを探す
find / -size +1G -type f -exec ls -lh {} \; 2>/dev/null

# /var/log の容量確認とクリーンアップ候補表示
du -sh /var/log/* | sort -rh | head -10
```

### プロセス調査
```bash
# メモリ使用量トップ10
ps aux --sort=-%mem | head -10

# CPU使用量トップ10
ps aux --sort=-%cpu | head -10

# 特定ユーザーのプロセス一覧
ps -u username

# ゾンビプロセス検索
ps aux | grep 'Z'
```

### ネットワーク調査
```bash
# 外部IPアドレス確認
curl ifconfig.me
curl ipinfo.io/ip

# ポート開放確認
nc -zv localhost 8080    # 8080番ポートが開いてるか

# 特定ポートで通信してるプロセス
lsof -i :3000
netstat -tulpn | grep :3000
```

### 一括処理
```bash
# 複数ファイルの文字コード変換（UTF-8へ）
for file in *.txt; do
  iconv -f SHIFT-JIS -t UTF-8 "$file" > "utf8_$file"
done

# ディレクトリ配下の全.pyファイルで文字列置換
find . -name "*.py" -exec sed -i 's/old_func/new_func/g' {} \;

# 複数サーバーに同じコマンド実行（ssh使用）
for host in server1 server2 server3; do
  ssh user@$host "uptime"
done
```

<div style="height:30px;"></div>

---

## 実務Tips

### コマンド履歴活用
```bash
history                  # コマンド履歴表示
history | grep docker    # 過去のdockerコマンド検索
!123                     # 履歴の123番目のコマンド実行
!!                       # 直前のコマンド再実行
sudo !!                  # 直前のコマンドをsudo付きで再実行（超便利！）
```

### エイリアス設定（~/.bashrc or ~/.zshrc）
```bash
alias ll='ls -lah'
alias ..='cd ..'
alias ...='cd ../..'
alias gs='git status'
alias gp='git pull'
alias dc='docker compose'
```

### 便利なショートカットキー
- `Ctrl + C`: 実行中のコマンド中断
- `Ctrl + D`: ログアウト（exitの代わり）
- `Ctrl + L`: 画面クリア（clearの代わり）
- `Ctrl + R`: コマンド履歴検索（超便利！）
- `Ctrl + A`: 行頭に移動
- `Ctrl + E`: 行末に移動
- `Ctrl + U`: カーソル位置から行頭まで削除
- `Ctrl + K`: カーソル位置から行末まで削除

### 標準出力・エラー出力のリダイレクト
```bash
command > output.txt              # 標準出力をファイルへ（上書き）
command >> output.txt             # 標準出力をファイルへ（追記）
command 2> error.txt              # 標準エラー出力をファイルへ
command > output.txt 2>&1         # 標準出力とエラー出力を同じファイルへ
command &> output.txt             # 同上（省略記法）
command > /dev/null 2>&1          # 出力を全て捨てる
```

### よくあるトラブル対処
```bash
# 「Permission denied」→ sudo付けるか権限確認
ls -l file.txt           # 権限確認
sudo command             # 管理者権限で実行

# 「command not found」→ パスが通ってるか確認
which command            # コマンドの場所確認
echo $PATH               # PATH環境変数確認

# 「No space left on device」→ ディスク容量確認
df -h                    # ディスク使用量確認
du -sh /*                # どこが容量食ってるか調査

# プロセスが終了しない
ps aux | grep process_name  # PID確認
kill -9 PID                 # 強制終了
```

<div style="height:50px;"></div>

---
