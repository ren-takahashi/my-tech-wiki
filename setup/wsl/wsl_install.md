# 🐧 WSL2 + Ubuntu 環境の構築手順（Windows）

## 前提
- Windows 10 (21H2以降) または Windows 11 を使用
- 管理者権限の PowerShell を使用すること

---

## 1. WSL をインストール
まず、PowerShell を「管理者権限」で開き、以下のコマンドを実行：

```powershell
wsl --install
```

> 💡 これで WSL2、仮想マシンプラットフォーム、Linuxカーネルなど必要なものが自動でインストールされる。  
> Linuxディストリビューションは、Ubuntuがデフォルトでインストールされる。  
> Ubuntu以外のディストリビューションを選ぶ場合は、`wsl --install -d <ディストリビューション名>` を使用する。
> 基本的に Ubuntu が推奨される。  
> - Ubuntu はコミュニティが活発でドキュメントも豊富。
> - WSL2 との互換性も高い。
> - Ubuntu はデフォルトで WSL2 をサポートしている。
 
インストールがうまくいかない場合:
BIOS/UEFIで仮想化機能が有効になっているか確認する。
ここには詳しく書かないので、必要に応じて調べる。

---

## 2. インストール状態を確認（任意）
以下のコマンドで状態を確認できます：

```powershell
wsl --status
```

以下のような表示がされる：

```
既定のバージョン: 2
WSL1 は、現在のマシン構成ではサポートされていません。
```

---

## 3. 初回起動とユーザー作成

初回起動時、自動で Ubuntu が起動し、以下のようにユーザー登録を求められる：

```powershell
Create a default Unix user account: accountName #アカウント名を入力する
New password: #passwordを入力する
Retype new password: #passwordを再入力する
passwd: password updated successfully
```

Linux 側のユーザーが作成される（Windowsとは別）  
accountNameとpasswordは忘れないようにどこかにメモしておく。
> 💡 WSL2のLinuxは「別のコンピューター」のようなもので、そこにログインするためのpasswordが必要

---

## 4. Windowsの再起動
初回インストール後は Windowsの再起動が必要。

## 5. 動作確認

再起動後、Linuxコマンドが使える状態になるので、
PowerShell を開き、以下のコマンドで Ubuntu を起動：

```powershell
wsl
```

```bash
#pwdコマンドを実行して、現在のディレクトリを確認してみる。
takahashiren@rt-laptop:/mnt/c/Users/username$ pwd
/mnt/c/Users/username

# exitでWSLを終了できる。
takahashiren@rt-laptop:/mnt/c/Users/username$ exit
```

> 💡 `/mnt/c/` 以下は Windows の `C:\` ドライブを指します。  
mntディレクトリは、Windowsのドライブをマウントするためのディレクトリ。  
基本的にはマウントのディレクトリは使用しないようにする。  
理由は、WSLのファイルシステムと Windowsのファイルシステムは異なるため、パフォーマンスや互換性の問題が発生する可能性があるため。

## 6. WSLをホームディレクトリで起動
`wsl`で起動するとマウントディレクトリに移動してしまうので、ホームディレクトリで起動する。
以下のコマンドでホームディレクトリで WSL を起動できる：

```powershell
#homeディレクトリで起動＆ログイン
wsl ~

# ログイン後、以下のように表示される
# takahashirenはwslのアカウント名で、rt-laptopはWindowsのホスト名（PCの名前）
takahashiren@rt-laptop:~$
```
> 💡 ホームディレクトリ配下で作業する（Linuxファイルシステムを使う）

---
これで WSL2 + Ubuntu のベース環境が構築完了。

---

## 補足
### パフォーマンス最適化
WSL2のメモリ使用量を設定ファイル（.wslconfig）で制限できる。

#### 設定手順
1. Windowsの「ユーザーフォルダ」（C:\Users\[ユーザー名]）に移動
2. .wslconfig という名前のファイルを作成
3. 以下の内容を記載：
```
[wsl2]
memory=4GB          # メモリを4GBに制限
processors=2        # CPUコア数を2つに制限
swap=2GB           # スワップファイルを2GBに設定
```
