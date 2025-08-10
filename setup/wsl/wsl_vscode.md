# WSL で VS Code と連携するまでの手順まとめ

## 1. VS Code をインストール（Windows 側で実行）
https://code.visualstudio.com/ からインストール。  
※すでにインストール済みならスキップ。

## 2. VS Code に「WSL」拡張機能をインストール（旧：Remote - WSL）

### 方法:
VS Code 左サイドバーの拡張機能（四角いアイコン）から「WSL」と検索し、
**Microsoft 製の「WSL」拡張機能**をインストール。

## 3. WSL 上で `code .` を実行する準備

Windows の PowerShell で以下を実行して WSL に入る：

```powershell
wsl ~ # ~ はログイン後すぐにホームディレクトリへ移動するオプション
```

WSL 上で以下のコマンドを実行：
```bash
code .
```
> 💡 `code .` は現在のディレクトリを VS Code で開くコマンド。

VS Code が起動したら以下の点を確認する。
- 左下のステータスバーに「WSL: Ubuntu」などと表示されていること
- ターミナルが WSL のものになっていること（`bash`や`zsh`など）


初回起動時には VS Code 側で WSL 用のサーバが自動で構成される。  
少し時間がかかる場合がありますが、成功すれば VS Code が起動する。

初回起動時に以下のようなポップアップが表示される：

```
Do you trust the authors of the files in this folder?
```
特定のフォルダや全体を「信頼」することで、機能制限なく使えるようになる。

## 4. 拡張機能の注意点
WSL 上で VS Code を使用する場合、拡張機能は、WSL 環境にインストールされる。
そのため、Windows 側でインストールした拡張機能は WSL 環境では利用できない。
WSL 環境で必要な拡張機能は、WSL 上で再度インストールする必要がある。