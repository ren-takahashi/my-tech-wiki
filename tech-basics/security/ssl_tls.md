# SSL/TLS証明書の詳細解説

## SSL/TLSとは

SSL/TLS（Secure Sockets Layer/Transport Layer Security）は、インターネット上で安全に通信するための暗号化技術です。<br>
現在は「SSL」という名前で呼ばれることが多いですが、実際に使われているのは後継技術の「TLS」です。<br>

### 何ができるのか
- **通信内容の暗号化**: データを盗み見られても解読できない形にする<br>
- **サーバーの身元証明**: 接続先が本物のサーバーかどうかを確認<br>
- **データの改ざん検知**: 通信途中でデータが書き換えられていないかチェック<br>

<div style="height:50px;"></div>

# ★ パブリック証明書の取得手順（Let's Encrypt編）

### 全体の流れ：「お店の営業許可証をもらう」ようなもの
外部からアクセスされるWebサイトには、公的に認められた証明書が必要です。<br>
Let's Encryptは無料でブラウザに信頼される証明書を発行してくれる認証局です。<br>

### ステップ1：前準備（土台作り）

**1-1. ドメイン名の取得**<br>
- お名前.comやムームードメインなどでドメインを購入<br>
- 例：`mywebsite.com` のような、あなた専用の住所を確保<br>
- 年間1,000円程度の費用がかかる<br>

**1-2. サーバーの準備**<br>
- VPSやクラウドサーバー（AWS、さくらVPS等）を契約<br>
- LinuxのWebサーバー（Apache or Nginx）をインストール<br>
- グローバルIPアドレスが割り当てられている状態にする<br>

**1-3. DNS（ドメインネームシステム）設定**<br>
- ドメイン管理画面で「Aレコード」を設定<br>
- `mywebsite.com` → `あなたのサーバーのIPアドレス` の紐付け<br>
- 設定後、世界中からその名前でアクセスできるまで数時間〜1日待つ<br>

### ステップ2：証明書発行ツールの準備

**2-1. Certbotのインストール**<br>
```bash
# Ubuntu/Debianの場合
sudo apt update
sudo apt install certbot python3-certbot-nginx

# CentOS/RHELの場合  
sudo yum install certbot python3-certbot-nginx
```

**2-2. Webサーバーの基本設定**<br>
- Nginxの場合：`/etc/nginx/sites-available/default` を編集<br>
- ドメイン名（server_name）を正しく設定<br>
- HTTP（ポート80）でアクセスできる状態にしておく<br>

### ステップ3：証明書の発行（本人確認）

**3-1. Certbotで証明書申請**<br>
```bash
sudo certbot --nginx -d mywebsite.com
```

**3-2. Let's Encryptによる本人確認**<br>
- Let's Encryptがあなたのサーバーにアクセス<br>
- 「本当にそのドメインの持ち主？」を確認するため特別なファイルを要求<br>
- Certbotが自動でそのファイルを置いて証明完了<br>

**3-3. 証明書の自動設定**<br>
- 成功すると証明書ファイルが `/etc/letsencrypt/live/` に保存<br>
- NginxのHTTPS設定が自動で追加される<br>
- ブラウザで `https://mywebsite.com` にアクセスして鍵マークを確認<br>

### ステップ4：自動更新の設定

**4-1. 更新テスト**<br>
```bash
sudo certbot renew --dry-run
```

**4-2. 自動更新スケジュール**<br>
```bash
sudo crontab -e
# 以下を追加（毎日正午に更新チェック）
0 12 * * * certbot renew --quiet
```

**理由**: Let's Encryptの証明書は90日で期限切れになるため、自動更新は必須<br>

### SSL証明書期限切れ障害の背景知識

**なぜ期限切れ障害が起こるのか？**<br>
- **従来の証明書**: 昔は1年〜3年の長期間有効な証明書が主流だった<br>
- **手動更新の習慣**: 長い期限のため「覚えていれば大丈夫」という運用が一般的<br>
- **Let's Encryptの登場**: 2016年から90日間の短期証明書が無料で提供開始<br>
- **移行期の混乱**: 長期証明書から短期証明書への移行で運用方法が変化<br>

**期限切れが起こる典型パターン**<br>
1. **自動更新設定の失敗**: cron設定ミスや権限不足で更新が動かない<br>
2. **有料証明書の更新忘れ**: 1年証明書で「来年更新すればいい」と忘れる<br>
3. **開発環境の放置**: テスト用証明書を長期間メンテナンスせず放置<br>
4. **監視不足**: 期限切れ前のアラート設定がない<br>
5. **責任者不在**: 証明書管理の担当者が不明確<br>

**Let's Encryptが90日にした理由**<br>
- **セキュリティ向上**: 万が一秘密鍵が漏洩しても被害期間を短縮<br>
- **自動化の促進**: 短期間なので手動更新は現実的でなく、自動化が必須に<br>
- **証明書の透明性**: 短期間で新しい証明書に切り替わるため追跡しやすい<br>

**現在のデファクトスタンダード**<br>
- Let's Encrypt利用時は自動更新が必須（ほぼ100%自動化）<br>
- 有料証明書でも1年更新 + アラート設定が一般的<br>
- 大企業では証明書管理システム（Certificate Manager等）で一元管理<br>


## 有償証明書 vs 無償証明書の違い

### 無償証明書（Let's Encrypt等）

**特徴**<br>
- **完全無料**: 費用は一切かからない<br>
- **短期間有効**: 90日間（自動更新前提）<br>
- **ドメイン認証のみ**: ドメインの所有者確認のみ<br>
- **同等の暗号化**: 技術的なセキュリティレベルは有償と同じ<br>

**メリット**<br>
- 導入コストゼロで気軽に始められる<br>
- 個人サイトや小規模サービスに最適<br>
- 自動化により運用負荷が低い<br>

**デメリット**<br>
- 会社情報等の追加認証がない<br>
- サポートが限定的（コミュニティベース）<br>
- 一部の古いブラウザ・システムで信頼されない場合がある<br>

### 有償証明書（Symantec、DigiCert、GlobalSign等）

**種類と特徴**<br>

**1. ドメイン認証（DV: Domain Validation）**<br>
- 費用：年間5,000円〜20,000円程度<br>
- 認証内容：ドメイン所有者確認のみ<br>
- 発行期間：数分〜数時間<br>
- 無償証明書とほぼ同等の機能<br>

**2. 組織認証（OV: Organization Validation）**<br>
- 費用：年間30,000円〜100,000円程度<br>
- 認証内容：ドメイン所有 + 会社の実在確認<br>
- 発行期間：1〜3営業日<br>
- 証明書に会社名が表示される<br>

**3. 拡張認証（EV: Extended Validation）**<br>
- 費用：年間100,000円〜300,000円程度<br>
- 認証内容：最も厳格な会社実在確認<br>
- 発行期間：1〜2週間<br>
- ブラウザのアドレスバーに会社名が表示（緑色表示）<br>

### 使い分けの指針

**無償証明書を選ぶべき場面**<br>
- 個人ブログやポートフォリオサイト<br>
- 開発・テスト環境<br>
- 小規模なWebアプリケーション<br>
- 技術的な暗号化のみが目的<br>

**有償証明書を選ぶべき場面**<br>
- **ECサイト**: 顧客の信頼性向上のため<br>
- **企業の公式サイト**: ブランドイメージとして<br>
- **金融系サービス**: 規制要件やコンプライアンス<br>
- **大規模サービス**: 24時間サポートが必要<br>
- **古いシステム対応**: レガシーブラウザ対応が必要<br>

### よくある誤解

**「無償証明書はセキュリティが低い」は間違い**<br>
- 暗号化の強度は有償・無償で同じ<br>
- 違いは「認証レベル」と「サポート」のみ<br>
- GoogleやFacebook等の大手サービスも無償証明書を使用している場合がある<br>

**「緑色の鍵マークが必要」も今は古い考え**<br>
- 現在のブラウザではEV証明書でも特別な緑色表示はしない<br>
- 一般ユーザーは鍵マークの色の違いをほとんど認識していない<br>
- 重要なのは「HTTPSになっていること」自体<br>

---

<div style="height:50px;"></div>

# ★ プライベート認証局の構築手順

### 全体の流れ：「社内専用の身分証明書発行所を作る」
外部に公開しない社内システムや開発環境では、自分で認証局を作って証明書を発行できます。<br>
ブラウザ警告は出ますが、社内端末に設定すれば警告なしでアクセス可能になります。<br>

**プライベート認証局の特徴**<br>
- **完全無料**: 自分で構築するので費用は一切かからない<br>
- **有償・無償の概念なし**: 外部の認証局を使わないため、料金体系は存在しない<br>
- **社内専用**: 外部ユーザーのブラウザでは警告が表示される<br>
- **自由な期間設定**: 証明書の有効期間を自由に決められる（1年、10年など）<br>
- **完全なコントロール**: 発行、更新、削除を全て自分で管理<br>

**証明書の発行について**<br>
- **基本は1サイト1証明書**: `internal.company.com` なら1つの証明書<br>
- **複数サイトがある場合**: サイトごとに個別の証明書を発行<br>
  - `hr.company.com` → HR用証明書<br>
  - `sales.company.com` → 営業用証明書<br>
  - `dev.company.com` → 開発用証明書<br>
- **ワイルドカード証明書**: `*.company.com` で複数サブドメインをカバー可能<br>
- **何枚でも無料**: 認証局を一度作れば、必要なだけ証明書を発行できる<br>

### ステップ1：認証局（CA）の作成

**この作業を行う場所**: 認証局を構築するサーバー（LinuxサーバーやPC）<br>
**重要**: これは企業のホームページなどを公開するWebサーバーとは**別のマシン**<br>
**役割**: 社内で使う証明書を発行する「印鑑屋さん」を作る専用の作業<br>

**具体例**<br>
- 管理者のPC（WindowsでもLinuxでもOK）<br>
- 社内の管理用サーバー<br>
- クラウドの小さなインスタンス（AWS EC2のt2.micro、さくらVPS 1GB等の最低スペックなど）<br>
- 仮想マシン（VMware、VirtualBox等）<br>

**Webサーバーとの関係**<br>
1. **CA構築マシン**（ここで作業）: 証明書を作る「工場」<br>
2. **Webサーバー**: 実際にホームページを公開するサーバー（別マシン）<br>
3. **クライアントPC**: 社員がアクセスする端末<br>

**1-1. 作業ディレクトリの準備**<br>
```bash
mkdir -p ~/private-ca/{ca,server,client}
cd ~/private-ca/ca
```
認証局関連のファイルを整理するためのフォルダを作成<br>

**1-2. CA用の秘密鍵作成**<br>
```bash
openssl genrsa -aes256 -out ca-private-key.pem 4096
```
- パスワードを求められるので、安全な文字列を設定<br>
- この鍵は「印鑑」のようなもので、厳重に管理する<br>
- この鍵で他の証明書に「お墨付き」を与える<br>

**1-3. CA証明書の作成**<br>
```bash
openssl req -new -x509 -days 3650 -key ca-private-key.pem -out ca-certificate.pem
```
入力項目の例：<br>
- Country Name: JP<br>
- State: Tokyo<br>
- City: Tokyo<br>
- Organization: My Company Ltd<br>
- Organizational Unit: IT Department<br>
- Common Name: **My Company Private CA**（重要：CA名）<br>
- Email: admin@mycompany.com<br>

**完了すると**: この段階で「社内専用の認証局」が完成<br>

### ステップ2：サーバー証明書の作成

**2-1. サーバー用秘密鍵の作成**<br>
```bash
cd ~/private-ca/server
openssl genrsa -out server-private-key.pem 2048
```

**2-2. 証明書署名要求（CSR）の作成**<br>
```bash
openssl req -new -key server-private-key.pem -out server.csr
```
入力項目の例：<br>
- Country Name: JP<br>
- State: Tokyo<br>
- City: Tokyo<br>
- Organization: My Company Ltd<br>
- Organizational Unit: Web Server<br>
- Common Name: **internal.mycompany.com**（重要：実際のサーバー名）<br>
- Email: webmaster@mycompany.com<br>

**2-3. CAでサーバー証明書に署名**<br>
```bash
openssl x509 -req -days 365 -in server.csr \
  -CA ~/private-ca/ca/ca-certificate.pem \
  -CAkey ~/private-ca/ca/ca-private-key.pem \
  -CAcreateserial -out server-certificate.pem
```

### ステップ3：Webサーバーへの設定

**3-1. 証明書ファイルの配置**<br>
```bash
sudo cp server-certificate.pem /etc/ssl/certs/
sudo cp server-private-key.pem /etc/ssl/private/
sudo chmod 600 /etc/ssl/private/server-private-key.pem
```

**3-2. Nginx設定例**<br>
```nginx
server {
    listen 443 ssl;
    server_name internal.mycompany.com;
    
    ssl_certificate /etc/ssl/certs/server-certificate.pem;
    ssl_certificate_key /etc/ssl/private/server-private-key.pem;
    
    # その他の設定...
}
```

### ステップ4：クライアント端末の設定

**重要**: 以下は社内システムにアクセスする**各ユーザーの端末**での作業<br>
**どちらか一つ**: 使用しているOSに応じて4-1または4-2のいずれかを実行<br>

**4-1. Windows端末への設定**（Windowsユーザーのみ）<br>
1. CA証明書（ca-certificate.pem）をクライアントPCにコピー<br>
2. ファイルをダブルクリック → 「証明書のインストール」<br>
3. 「ローカルマシン」→「信頼されたルート証明機関」に保存<br>
4. ブラウザを再起動<br>

**4-2. Linux端末への設定**（Linuxユーザーのみ）<br>
```bash
sudo cp ca-certificate.pem /usr/local/share/ca-certificates/mycompany-ca.crt
sudo update-ca-certificates
```

**設定が必要な人**<br>
- 社内システムにアクセスする全ての社員<br>
- 各自が使っているPC/端末で1回だけ設定すればOK<br>
- 新しい社員が入った時も同様の設定が必要<br>

**4-3. 確認方法**<br>
- ブラウザで `https://internal.mycompany.com` にアクセス<br>
- 警告が出ずに鍵マークが表示されれば成功<br>

---

# ★ パブリック と プライベート証明書の使い分け

### パブリック証明書が適している場面
- 外部に公開するWebサイト<br>
- ECサイトなど一般ユーザーがアクセスするサービス<br>
- API等で外部サービスと連携する場合<br>

### プライベート証明書が適している場面
- 社内システムやイントラネット<br>
- 開発・テスト環境<br>
- 限られたメンバーのみがアクセスするサービス<br>

### セキュリティ上の注意点
- プライベートCA の秘密鍵は厳重に管理（漏洩すると偽の証明書を作られる）<br>
- 定期的な証明書の更新と管理<br>
- 不要になった証明書の確実な削除<br>

---
