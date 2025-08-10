# GitHub で SSH を設定する

## 概要
GitHub はクラウドサーバのホスティングサービスなので、ネットワークを介してデータを何度もやり取りする作業が中心になる。  
そのため、重要なファイルコードが経由したサーバ上で漏洩する危険を回避するため暗号状態の「**SSH通信**」を行うことが必須になっている。  
そのため、GitHubにSSHキーを登録して、SSH通信を行う設定を行う必要がある。

## 1. SSHキーを生成する
まずは、ホームディレクトリに移動し「**.ssh**」の不可視ディレクトリを作成する。  
その中でキーを管理するのが慣習。作成したら中に入り「**ssh-keygen**(キージェネレート:鍵の生成)」コマンドで秘密鍵と公開鍵を生成する。

```bash
# .ssh作成してそのディレクトリへ移動する
mkdir ~/.ssh
cd ~/.ssh
```

```bash
# SSHキーを生成する
ssh-keygen -t ed25519 -C "githubに登録したメールアドレス"
```
オプションの`-t`で暗号化方式を指定。
`-C`はコメントを付けるオプションで、GitHubに登録したメールアドレスを入力しておくと後で識別しやすい。
キーを生成時に暗号キーの名前を確認されるが、省略すると`id_ed25519`と付与される。
> 💡 `ed25519` は、新しい暗号方式で、より安全性が高いとされている。

```bash
# キーを生成をするため、以下実行
Generating public/private ed25519 key pair. 
```

以下の表示が出てくる。

```bash
Enter file in which to save the key　 (/home/username/.ssh/id_ed25519):
# SSHキーのファイル名を指定。
# Enterキーを押して省略した場合はed25519がファイル名になるが、後でキーの用途が理解できるよう明示的に「github」と入力しておく。
```

```bash
Enter passphrase (empty for no passphrase):　入力しないでEnter
Enter same passphrase again:　入力しないでEnter
# 上記は基本的にどちらもEnterを押して省略する（登録すると毎回pushのたびにログインパスワード入力が必要になる）
# パスフレーズを入力しても表示されないが、入力はされている。
```
指定したファイル名の秘密鍵と公開鍵が生成される。

## 2. SSHキーの確認
ファイル名を「github」と指定したので、「github」と「github.pub」というファイルが生成される。
> 💡 `github`が秘密鍵で、`github.pub`が公開鍵。

github.pub（公開鍵）をエディタなどで中身を確認する。  
空白はそのままの状態で中身の「**ssh-ed25519 …**」で始まるコードを全てコピーしたら、GitHubにアクセスし、以下の手順通りに公開鍵を登録していく。

1. GitHubの右上のアイコンメニューから「**Settings**」 を選択する。
2. 左サイドメニューの「**SSH and GPG keys**」を選択
3. 右上の「**New SSH Key**」ボタンを押す。
4. 「**Title**」はPCのOS名と登録日時を記述することで、数年後に識別しやすい管理名などを入力し、「**Key**」には先ほどコピーした公開鍵の内容を貼り付けて「**Add SSH key**」を押す。
> 💡 Titleは「github-<ユーザー名>-<OS名>-<日付>」のように、分かりやすいものにする。

## 3. SSH設定ファイルの作成
.sshディレクトリ内にconfig (拡張子無し)ファイルを自身で用意する。
VSCodeで開き設定コードをそのまま貼り付ける。
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

## 4. SSH接続の確認
SSHキーの登録が完了したら、SSH接続が正しく行えるか確認する。
以下のコマンドを実行して、GitHubにSSH接続できるか確認する。
```bash
ssh -T git@github.com
```

以下の文が出ればOK
```bash
Hi <GitHubのアカウント名>! You've successfully authenticated, but GitHub does not provide shell access.
```

その後、`clone`など問題なく行えれば、無事設定が完了したことになる。


## メモ
Ubuntuでだと上手くいかなかったが、powerShellだと上手くできた。