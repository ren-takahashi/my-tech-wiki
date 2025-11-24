# GitHub 新規リポジトリ作成と初回コミットの手順

GitHubで新しいリポジトリを作成し、ローカルで作業を開始するまでの手順をまとめました。

## 1. GitHubでリポジトリを作成

### 1-1. GitHubにアクセス
- [GitHub](https://github.com) にログイン
- 右上の「+」アイコンをクリック → 「New repository」を選択

### 1-2. リポジトリ設定
以下の項目を入力・選択：

- **Repository name**: プロジェクト名（例: `my-new-project`）
- **Description**: プロジェクトの説明（任意）
- **Public / Private**: 公開設定を選択
  - Public: 誰でも見られる
  - Private: 自分と指定した人だけ見られる
- **Initialize this repository with**:
  - ✅ **Add a README file**: チェック推奨（リポジトリの説明ファイル）
  - ✅ **Add .gitignore**: 必要に応じて選択（言語/フレームワークのテンプレート）
  - ✅ **Choose a license**: 必要に応じて選択（MIT、Apache等）

### 1-3. リポジトリ作成
- 「Create repository」ボタンをクリック

## 2. ローカルにクローン

### 2-1. リポジトリURLをコピー
- 作成したリポジトリのページで「Code」ボタンをクリック
- **SSH** タブを選択（推奨）
- URLをコピー（例: `git@github.com:username/my-new-project.git`）

### 2-2. ローカルにクローン
```bash
# 作業したいディレクトリに移動
cd ~/projects

# リポジトリをクローン
git clone git@github.com:username/my-new-project.git

# クローンしたディレクトリに移動
cd my-new-project
```

## 3. 初回コミット

### 3-1. ファイルを作成・編集
```bash
# 例: 新しいファイルを作成
echo "# My New Project" > README.md
echo "console.log('Hello, World!');" > index.js
```

### 3-2. Git状態を確認
```bash
# 変更されたファイルを確認
git status
```

### 3-3. ファイルをステージング
```bash
# 全てのファイルを追加
git add .

# または特定のファイルだけ追加
git add README.md index.js
```

### 3-4. コミット
```bash
# コミットメッセージを付けてコミット
git commit -m "Initial commit"
```

### 3-5. GitHubにプッシュ
```bash
# mainブランチにプッシュ
git push origin main

# または（デフォルトブランチがmainの場合）
git push origin main
```

## 4. 確認

GitHubのリポジトリページをブラウザで更新して、ファイルがアップロードされていることを確認。

---

## トラブルシューティング

### SSH接続エラーが出る場合
SSH鍵の設定が必要 → [github_ssh.md](./github_ssh.md)

### ブランチ名が違う場合
```bash
# 現在のブランチ名を確認
git branch

# ブランチ名を変更したい場合
git branch -M main
```

### リモートURLを確認したい場合
```bash
git remote -v
```

### 既存のローカルプロジェクトをGitHubに上げたい場合
```bash
# 1. GitHubで空のリポジトリを作成（READMEなどは追加しない）

# 2. ローカルプロジェクトのディレクトリで
git init
git add .
git commit -m "Initial commit"

# 3. リモートリポジトリを追加
git remote add origin git@github.com:username/my-new-project.git

# 4. プッシュ
git branch -M main
git push -u origin main
```


# よく使うGitコマンド
## ブランチ作成、移動
```bash
# 新しいブランチを作成して切り替え
git checkout -b {ブランチ名}

# ブランチ切り替え
# オプションとしてswitchの後にハイフン（-）をつけると直前のブランチに戻れる
git switch {ブランチ名}

# ブランチ一覧表示
# -a オプションでリモートブランチも表示
git branch

# 今いるブランチに、{取り込みたいブランチ名}の変更を取り込む
git merge {取り込みたいブランチ名}
# 補足:
# git mergeは、現在のブランチに指定したブランチの変更を統合する操作。
# mainから切っている作業ブランチで自分が作業している際に、誰かがmainに変更を加えた場合、
# 作業ブランチにmainの最新変更を取り込みたいときに使う。
# （mainに移動してgit pullで最新のmainを取得してから作業ブランチに戻りgit merge mainを実行する）
```
### ブランチ削除
```bash
# ブランチの削除（ローカル）
# -d オプションで安全に削除（リモートにマージ済みのブランチのみ削除可能）
# -D オプションで強制削除（マージ未済みでも削除可能）
git branch -d {ブランチ名}

# ブランチの削除（リモート）
git push origin --delete {ブランチ名}
```

## 差分管理
```bash
# 状態確認
git status

# 変更を追加（コミット対象に指定する（ステージングに上げる））
git add .

# コミット
# -m "{コミットメッセージ}" オプションでメッセージを付けることもできる（基本コメントは付けたい）
# コミット実行時点ではリモートには反映されない。
git commit

# プッシュ（リモートリポジトリに変更を反映）
git push

# プル（リモートの変更を取得して反映）
# -p オプションで削除されたリモートブランチをローカルからも削除できる（ブランチの同期）
git pull

# 変更を一時退避
# -u オプションで、ディレクトリごと退避可能
git stash

# 直前の退避分を戻す
git stash pop

```

## たまに使うやつ
```bash
# フェッチ（リモートの変更を取得のみで、作業コピーには反映されない）
git fetch

# ログ確認
git log --oneline
```

## 慎重に使うやつ
```bash
# リセット（直前のコミットを取り消し、変更内容はステージングに残す）
git reset --soft HEAD^

# 【注意】リセット（直前のコミットを取り消し、変更内容も破棄）
git reset --hard HEAD^
```

## 【注意】プッシュの取り消し手順

### 履歴を残したくない場合
```bash
# 1. 直前のコミットを取り消しをする（一旦、変更内容はステージングに残す）
git reset --soft HEAD^
# 2. 変更差分を退避
git stash -u
# 3. 強制プッシュの-fオプションをつけてプッシュすることでリモートを上書き
git push -f origin HEAD
```
### 履歴を残したい場合
```bash
# 1. resetではなくrevertで直前のコミットを打ち消す新しいコミットを作成
git revert HEAD^
# 2. 通常通りプッシュ
git push origin HEAD
```