# 要員管理ミニアプリ ― 設計ドキュメント（初版）

**目的**  
5パラメータ＋スキル表を用いて、開発メンバーの**スキル可視化**だけを行うシンプルな Web アプリを GitHub Pages 上にホスティングする。
**スキル可視化に特化した軽量な要員管理アプリ**が手にはいります。  

---

## 1. スコープと前提

| 項目 | 内容 |
|------|------|
| 対象機能 | ①メンバー情報の閲覧<br>②タグ／キーワードによる簡易フィルタ |
| 除外機能 | 稼働率、アサイン状況、ガント管理、勤怠など |
| 運用形態 | - **静的ファイルのみ**（HTML＋JS＋JSON）<br>- GitHub Pages で公開 |
| 想定ユーザー | 社内メンバー（閲覧のみ） |
| データ更新 | JSON を GitHub で直接編集 or PR で更新（バージョン管理） |

---

## 2. 機能要件

### 2.1 一覧表示
| No | 機能 | 詳細 |
|----|------|------|
| F‑01 | メンバー一覧 | `members.json` を読み込み、カード/リストで表示 |
| F‑02 | 簡易フィルタ | 名前・タグ・スキル名をテキスト検索（JS クライアントサイド） |

### 2.2 詳細表示
| No | 機能 | 詳細 |
|----|------|------|
| F‑03 | プロフィール詳細 | 5 パラメータをレーダーチャート表示（Chart.js）<br>戦法／技能／タグ／プロジェクト履歴／名言をセクション表示 |

---

## 3. 非機能要件

| 項目 | 目標値・方針 |
|------|-------------|
| パフォーマンス | 100 人程度まで 1 秒以内に初期描画 |
| ブラウザ互換 | 現行版 Chrome / Edge / Firefox / Safari |
| セキュリティ | すべて公開情報前提・認証なし |
| 可搬性 | 静的ファイルのみで動作（依存ライブラリは CDN 利用） |

---

## 4. データモデル（`members.json`）

```jsonc
{
  "members": [
    {
      "id": "yamada_taro",          // 一意 ID
      "name": "山田 太郎",
      "title": "GitHubを統べし者",
      "parameters": {               // 0–100
        "統率": 67,
        "武力": 91,
        "知力": 82,
        "政治": 59,
        "魅力": 76
      },
      "skills": {
        "active": [
          "爆速ホットフィックス",
          "コードベース瞬間把握",
          "カオス仕様整理爆撃"
        ],
        "passive": [
          "闇コード通訳者",
          "トレース耐性",
          "デグレ敏感体質"
        ]
      },
      "tags": [
        "#Reactマスター",
        "#ホットフィクサー",
        "#メンター属性あり"
      ],
      "projects": [
        {
          "name": "某金融系管理システム",
          "year": "2022",
          "note": "レガシーな仕様を読み解きSPA化を推進"
        }
      ],
      "quote": "設計がない？ なら俺が書く。"
    }
    // ...他メンバー
  ]
}
```

> **更新フロー**  
> 1. PR で JSON 追記 or 変更  
> 2. GitHub Actions なしで自動デプロイ（Pages のブランチ公開）  

---

## 5. ファイル構成

```
yoin-kanri/             （GitHub リポジトリ）
├── index.html          … 一覧＋詳細 SPA
├── css/                … 必要なら分離
│   └── style.css
├── js/
│   └── app.js          … fetch & 描画ロジック
└── data/
    └── members.json
```

- **index.html**  
  - `<script src="./js/app.js" type="module">`
  - `<link rel="stylesheet" href="./css/style.css">`
- **CDN**  
  - Chart.js, (必要なら) Fuse.js で全文検索

---

## 6. UI/UX ラフ

| 画面 | 主要要素 |
|------|---------|
| 一覧 | 検索ボックス＋タグフィルタ<br>メンバーカード（名前・5角形ミニチャート・タグ） |
| 詳細 | ①名前・肩書き<br>②レーダーチャート（5 パラメータ）<br>③戦法／技能テーブル<br>④タグ群<br>⑤プロジェクト履歴タイムライン<br>⑥名言 |

---

## 7. 技術スタック & 実装ポイント

| 項目 | 採用技術 / 手法 |
|------|----------------|
| ビルド | なし（素の ES Modules） |
| ルーティング | ハッシュルート `index.html#yamada_taro` |
| データ取得 | `fetch('./data/members.json')` |
| 描画 | Vanilla JS or 小規模 FW（Lit / Alpine.js でも可） |
| グラフ | Chart.js レーダーチャート |
| デプロイ | GitHub Pages（`main` ブランチ root） |

---

## 8. 将来拡張アイデア（参考）

| カテゴリ | 追加機能 |
|----------|----------|
| データ管理 | GitHub Actions + PR マージで JSON 検証 (CI) |
| UI | タグクラウド／ヒートマップ式スキルマップ |
| 編集 | GitHub 連携フォーム（Issues → PR 生成） |
| 認証 | 社内 SSO 連携（必要になったら） |

---
