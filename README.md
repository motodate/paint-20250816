# ペイントアプリ（HTML版）

> **📚 学習用プロジェクト**  
> このリポジトリは Claude Code を使用したライブコーディング学習用のテストアプリケーションです。  
> GitHub Pages での公開を前提として、Svelte 5 + Vite で構築されたシンプルなペイントツールを実装しています。

ブラウザ上で動作するシンプルなペイントツール。Svelte 5 + Vite で構築されており、**ビルド後は静的ファイルとして GitHub Pages などで公開可能**です。

## 特徴

- 🎨 ペンツール（10色プリセット、太さ調整可能）
- 🧹 消しゴムツール（透明消去）
- 💾 PNG形式での画像ダウンロード
- 🗑️ キャンバス全消去
- ⌨️ キーボードショートカット対応
- 📱 タッチデバイス対応
- 🖥️ 高DPI ディスプレイ対応

## 開発環境のセットアップ

### 必要な環境
- Node.js 18以上（開発時のみ必要）
- npm または pnpm

### インストール

```bash
# 依存関係のインストール
npm install

# 開発サーバーの起動
npm run dev
```

開発サーバーが起動したら、ブラウザで `http://localhost:5173` を開いてください。

## ビルド

```bash
# プロダクション用ビルド
npm run build
```

`dist/` フォルダに静的ファイルが生成されます。これらのファイルは **Node.js 不要で、HTTPサーバー上でブラウザだけで動作**します。

### ⚠️ ローカルファイルでは動作しません
本アプリは ES Modules を使用しているため、**CORS制限により `file://` プロトコルでは動作しません**。

- ❌ **動作しない**: `file:///path/to/index.html` を直接ブラウザで開く
- ✅ **動作する**: HTTPサーバー経由でアクセス

ローカルで動作確認する場合は、簡易HTTPサーバーを使用してください：
```bash
# 方法1: Python
python -m http.server 8000

# 方法2: Node.js
npx serve .

# 方法3: PHP  
php -S localhost:8000
```

## GitHub Pages へのデプロイ

### ⚠️ 重要な注意事項
GitHub Pages は以下の制約があります：
- 公開できるのは「リポジトリのルート（/）」または「ルート直下の /docs フォルダ」のみ
- サブフォルダ（例: /paint-app/dist）からの公開は**不可能**

そのため、ビルド後のファイルをリポジトリのルートに配置する必要があります。

### 方法1: 手動デプロイ（リポジトリルートに配置）

1. paint-app ディレクトリでビルドを実行
```bash
cd paint-app
npm run build
```

2. ビルドファイルをリポジトリのルートにコピー
```bash
# リポジトリのルートに戻る
cd ..

# dist の内容をルートにコピー
cp -r paint-app/dist/* .

# コミット & プッシュ
git add .
git commit -m "Deploy paint app to GitHub Pages"
git push origin main
```

3. GitHub リポジトリの設定
   - Settings → Pages を開く
   - Source: Deploy from a branch
   - Branch: `main` / `/ (root)`
   - Save をクリック

### 方法2: GitHub Actions を使った自動デプロイ

`.github/workflows/deploy.yml` を作成：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: write
  pages: write
  id-token: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
          cache-dependency-path: ./paint-app/package-lock.json
      
      - name: Install and Build
        run: |
          cd paint-app
          npm ci
          npm run build
      
      - name: Copy to root
        run: |
          cp -r paint-app/dist/* .
          
      - name: Commit and push
        run: |
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'
          git add .
          git commit -m "Deploy paint app to GitHub Pages" || exit 0
          git push
```

GitHub リポジトリの設定：
- Settings → Pages
- Source: Deploy from a branch
- Branch: `main` / `/ (root)`

## 公開 URL

GitHub Pages にデプロイ後、以下の URL でアクセス可能：
- `https://[ユーザー名].github.io/[リポジトリ名]/`

### ベースパスの設定（リポジトリ名がある場合）

`vite.config.js` でベースパスを設定：

```js
export default defineConfig({
  base: '/[リポジトリ名]/',
  plugins: [svelte()],
})
```

## ライセンス

MIT