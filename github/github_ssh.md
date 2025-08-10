# GitHub で SSH を設定する

## 概要
GitHub はクラウドサーバのホスティングサービスなので、ネットワークを介してデータを何度もやり取りする作業が中心になる。  
そのため、重要なファイルコードが経由したサーバ上で漏洩する危険を回避するため暗号状態の「**SSH通信**」を行うことが必須になっている。  
そのため、GitHub に SSH キーを登録して、SSH 通信を行う設定を行う必要がある。

## 1. SSH キーを生成する
まずは、ホームディレクトリに移動し「**.ssh**」の不可視ディレクトリを作成する。  
その中でキーを管理するのが慣習。作成したら中に入り「**ssh-keygen**(キージェネレート:鍵の生成)」コマンドで秘密鍵と公開鍵を生成する。

```bash
# .ssh作成してそのディレクトリへ移動する
mkdir ~/.ssh
cd ~/.ssh
```

```bash
# SSH キーを生成する
ssh-keygen -t ed25519 -C "github に登録したメールアドレス"
```
オプションの`-t`で暗号化方式を指定。
`-C`はコメントを付けるオプションで、GitHub に登録したメールアドレスを入力しておくと後で識別しやすい。
キーを生成時に暗号キーの名前を確認されるが、省略すると`id_ed25519`と付与される。
> 💡 `ed25519` は、新しい暗号方式で、より安全性が高いとされている。

```bash
# キーを生成をするため、以下実行
Generating public/private ed25519 key pair. 
```

以下の表示が出てくる。

```bash
Enter file in which to save the key　 (/home/username/.ssh/id_ed25519):
# SSH キーのファイル名を指定。
# Enter キーを押して省略した場合は ed25519 がファイル名になるが、後でキーの用途が理解できるよう明示的に「github」と入力しておく。
```

```bash
Enter passphrase (empty for no passphrase):　入力しないで Enter
Enter same passphrase again:　入力しないで Enter
# 上記は基本的にどちらも Enter を押して省略する（登録すると毎回 push のたびにログインパスワード入力が必要になる）
# パスフレーズを入力しても表示されないが、入力はされている。
```
指定したファイル名の秘密鍵と公開鍵が生成される。

## 2. SSH キーの確認
ファイル名を「github」と指定したので、「github」と「github.pub」というファイルが生成される。
> 💡 `github`が秘密鍵で、`github.pub`が公開鍵。

github.pub（公開鍵）をエディタなどで中身を確認する。  
空白はそのままの状態で中身の「**ssh-ed25519 …**」で始まるコードを全てコピーしたら、GitHub にアクセスし、以下の手順通りに公開鍵を登録していく。

1. GitHub の右上のアイコンメニューから「**Settings**」 を選択する。
2. 左サイドメニューの「**SSH and GPG keys**」を選択
3. 右上の「**New SSH Key**」ボタンを押す。
4. 「**Title**」は PC の OS 名と登録日時を記述することで、数年後に識別しやすい管理名などを入力し、「**Key**」には先ほどコピーした公開鍵の内容を貼り付けて「**Add SSH key**」を押す。
> 💡 Title は「github-<ユーザー名>-<OS名>-<日付>」のように、分かりやすいものにする。

## 3. SSH 設定ファイルの作成
.ssh ディレクトリ内に config (拡張子無し)ファイルを自身で用意する。
VS Code で開き設定コードをそのまま貼り付ける。
```
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/github
    User git
```

| **Host** | リモートリポジトリのURL「**git@…**」とマッチさせる。複数のGitHub IDを使い分ける際は`github-accountName`にする。 |
| --- | --- |
| **HostName** | アクセス先の**サービスドメイン**(ホストアドレス) |
| **IdentityFile** | **秘密鍵ファイル**のパス |
| **User** | クライアントユーザが操作する管理**システム** |

## 4. SSH 接続の確認
SSH キーの登録が完了したら、SSH 接続が正しく行えるか確認する。
以下のコマンドを実行して、GitHub に SSH 接続できるか確認する。
```bash
ssh -T git@github.com
```

以下の文が出れば OK
```bash
Hi <GitHub のアカウント名>! You've successfully authenticated, but GitHub does not provide shell access.
```

その後、`clone`など問題なく行えれば、無事設定が完了したことになる。


## メモ
Ubuntu でだと上手くいかなかったが、PowerShell だと上手くできた。