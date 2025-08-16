# ペイントアプリ（HTML版）

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

`dist/` フォルダに静的ファイルが生成されます。これらのファイルは **Node.js 不要で、ブラウザだけで動作**します。

## GitHub Pages へのデプロイ

### 方法1: 手動デプロイ

1. ビルドを実行
```bash
npm run build
```

2. `dist` フォルダの内容を `gh-pages` ブランチにプッシュ
```bash
# gh-pages ブランチがない場合は作成
git checkout -b gh-pages

# dist フォルダの内容をコピー
cp -r dist/* .

# コミット & プッシュ
git add .
git commit -m "Deploy to GitHub Pages"
git push origin gh-pages
```

3. GitHub リポジトリの設定
   - Settings → Pages を開く
   - Source: Deploy from a branch
   - Branch: `gh-pages` / `/ (root)`
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
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

GitHub リポジトリの設定：
- Settings → Pages
- Source: GitHub Actions を選択

### 方法3: Vite の gh-pages プラグインを使用

1. プラグインをインストール
```bash
npm install --save-dev gh-pages
```

2. `package.json` に deploy スクリプトを追加
```json
{
  "scripts": {
    "deploy": "npm run build && gh-pages -d dist"
  }
}
```

3. デプロイ実行
```bash
npm run deploy
```

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