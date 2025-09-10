# TASKS.md

_Notion記事自動生成システム 実装チェックリスト_

> このファイルはリポジトリ内でシンプルに進捗管理するためのチェックリストです。
> タスクが完了したら `- [ ]` を `- [x]` に更新してください。

---

## Milestones

- **Week 1**: P0 基盤整備 / P1 抽出・前処理
- **Week 2**: P2 Claude連携 / P3 SSE
- **Week 3**: P4 フロントUI
- **Week 4**: P5 Notion連携 / P6 テスト
- **Week 5**: P7 Docker運用 / P8 Docs

---

## Week 1

### [W1-D1] リポジトリ初期化と共通設定

- [ ] ルート構成作成（`frontend/`, `backend/`, `proxy/`, `.claude/commands/`）
- [ ] `.env.example` 作成（`NOTION_API_KEY`, `NOTION_DATABASE_ID`, `CLAUDE_BIN`, `REQUEST_TIMEOUT_SEC`）
- [ ] エラーコード定義（E100〜E900）追加

### [W1-D2] FastAPI スケルトン & /extract 雛形

- [ ] `app/main.py` 作成（CORS: `*`、SSE準備）
- [ ] `/extract` エンドポイント雛形
- [ ] Trafilatura 導入・疎通

### [W1-D3] 抽出フォールバック & ファイル入力

- [ ] BeautifulSoup フォールバック実装
- [ ] `.md/.txt` 複数ファイルアップロード処理

### [W1-D4] 正規化 & 入力サイズ制限

- [ ] 軽量Markdown正規化（H2/H3、フェンス保持、ノイズ最小化）
- [ ] `MAX_FILE_SIZE_MB=10`, `MAX_TOTAL_UPLOAD_MB=100` バリデーション

### [W1-D5] /extract テスト

- [ ] pytest セットアップ
- [ ] 正常/異常（無効URL）テスト
- [ ] マイルストーン: 正規化Markdownを返却

---

## Week 2

### [W2-D6] Claude ラッパ & /outline コマンド雛形

- [ ] `.claude/commands/outline.md` 作成（成果物のみ出力）
- [ ] `claude_runner.py`（subprocess・timeout・stderr収集）

### [W2-D7] /outline 実装 & JSON 検証/修復

- [ ] `/outline` SSE 実装（`progress/partial/complete/error`）
- [ ] JSONスキーマ検証 → NG時 `/repair` 呼び出し

### [W2-D8] /generate 実装 & Markdown 検証

- [ ] `.claude/commands/generate.md`
- [ ] `/generate` SSE 実装
- [ ] Markdown検証（H1禁止、ID必須、フェンス閉包）

### [W2-D9] /regenerate 実装 & 差し替え

- [ ] `.claude/commands/regenerate.md`
- [ ] 見出し〜次の同階層直前まで置換ロジック

### [W2-D10] /repair 実装 & 単体テスト

- [ ] `.claude/commands/repair.md`
- [ ] outline/generate/regenerate/repair のpytest
- [ ] マイルストーン: Claude連携通貫

---

## Week 3

### [W3-D11] フロント骨組み

- [ ] Next.js scaffolding
- [ ] ステップウィザード土台（Step1/3/4/5）

### [W3-D12] アウトラインエディタ

- [ ] H2/H3表示・インライン編集
- [ ] ノードID・目安文字数表示

### [W3-D13] アウトラインD&D

- [ ] 並べ替え（階層整合も検証）
- [ ] 編集反映の差分送出

### [W3-D14] 本文エディタ

- [ ] Markdownプレビュー
- [ ] 編集エリアとID維持

### [W3-D15] 再生成モーダル & SSE クライアント

- [ ] scope/target/extra 指示UI
- [ ] SSE (`progress/partial/complete/error`), keep-alive=20s
- [ ] マイルストーン: outline→generate→本文表示が通る

---

## Week 4

### [W4-D16] Notion クライアント基盤

- [ ] `.env` 読込・権限チェック
- [ ] DB接続の疎通

### [W4-D17] /to-notion 実装

- [ ] Name/Category/Keywords 書込
- [ ] 未存在値の自動作成

### [W4-D18] プレビュー & 再送

- [ ] Notion形式プレビュー
- [ ] 失敗詳細表示＋「再送」ボタン

### [W4-D19] テスト

- [ ] pytest: to-notion 正常/異常
- [ ] Playwright: 入力→送信 E2E

### [W4-D20] CI

- [ ] GitHub Actions: pytest + Playwright
- [ ] PR 時自動実行

---

## Week 5

### [W5-D21] Dockerfile & Compose

- [ ] frontend/backend/proxy の Dockerfile
- [ ] `docker-compose.yml`（dev/prod）

### [W5-D22] Nginx 設定

- [ ] SSE: `proxy_buffering off;` / `proxy_read_timeout`
- [ ] CORS: `*`
- [ ] SSL（任意）

### [W5-D23] ドキュメント

- [ ] README / 運用ガイド / コマンド一覧 / 仕様書同梱
- [ ] Mermaid 図の表示確認

### [W5-D24] 最終E2E & 修正

- [ ] compose 環境で通貫テスト
- [ ] 既知バグ修正

### [W5-D25] リリース

- [ ] タグ付与・CHANGELOG
- [ ] 完了レビュー

---

## ラベル提案

- `area:backend` `area:frontend` `area:proxy` `area:infra`
- `type:feature` `type:bug` `type:docs` `type:test`
- `prio:P0` `prio:P1` `prio:P2`

---

## 完了条件（各タスク共通）

- 受け入れ基準を満たす
- 該当する単体/E2Eテストが通過
- 必要に応じてドキュメント更新済み
