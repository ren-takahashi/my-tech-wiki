# ✅ WSLでVS Codeと連携するまでの手順まとめ

## 1.VS Codeをインストール（Windows 側で実行）
https://code.visualstudio.com/ からインストール。  
※すでにインストール済みならスキップ。

## 2.VS Codeに「WSL」拡張機能をインストール（旧：Remote - WSL）

### 方法:
VS Code左サイドバーの拡張機能（四角いアイコン）から「WSL」と検索し、
**Microsoft 製の「WSL」拡張機能**をインストール。

## 3. WSL上で`code .`を実行する準備

WindowsのPowerShellで以下を実行して WSLに入る：

```powershell
wsl ~ # ~ はログイン後すぐにホームディレクトリへ移動するオプション
```

WSL上で以下のコマンドを実行：
```bash
code .
```
> 💡 `code .` は現在のディレクトリをVS Codeで開くコマンド。

VS Codeが起動したら以下の点を確認する。
- 左下のステータスバーに「WSL: Ubuntu」などと表示されていること
- ターミナルがWSLのものになっていること（`bash`や`zsh`など）


初回起動時にはVS Code側で WSL用のサーバが自動で構成される。  
少し時間がかかる場合がありますが、成功すればVS Codeが起動する。

初回起動時に以下のようなポップアップが表示される：

```
Do you trust the authors of the files in this folder?
```
特定のフォルダや全体を「信頼」することで、機能制限なく使えるようになる。

## 4. 拡張機能の注意点
WSL上でVS Codeを使用する場合、拡張機能は、WSL環境にインストールされる。
そのため、Windows側でインストールした拡張機能はWSL環境では利用できない。
WSL環境で必要な拡張機能は、WSL上で再度インストールする必要がある。