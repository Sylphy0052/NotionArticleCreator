# /outline — アウトライン生成コマンド
>
> 目的：正規化済みの素材Markdownとルールから、H2/H3のみの**アウトラインJSON**を生成する。
> 出力は **JSONのみ**（説明・コードフェンス・余計な文字禁止）。

## 使用サブエージェント

- **OutlineAgent**（`.claude/agents/outline.md`）
  - このコマンドの主担当。入力ルールとSTYLEに従い、H2/H3の階層を設計し、UUID/parentId/orderを整合させたJSONを出力する。

## 入力（引数仕様）

- `--var STYLE`: `"A" | "B" | "C"`
  - A: 技術ブログ風（簡潔・実践重視）
  - B: 解説記事風（序論→背景→手順→考察→まとめ）
  - C: ノート風（短セクション多数・要点中心）
- `--var STRUCTURE_RULES_JSON`: **必須**。必須セクション・順序・禁止項目などのルールJSON（ファイルパス）
- `--var LENGTH_HINTS_JSON`: 任意。各見出しの目安文字数JSON（ファイルパス）
- `--input-file`: **必須**。抽出・正規化済みの素材Markdown

## バリデーション（実行前）

- `STRUCTURE_RULES_JSON` の存在とJSON整合を確認。整合NGなら実行を中断してエラー終了。
- `STYLE` が A/B/C のいずれかであること。
- 入力Markdownが非空であること。

## 実行手順（フロー）

1. **OutlineAgent** を呼び出し、以下の制約で生成を指示する：
   - `outlineVersion="1.0"` 固定
   - `nodes[].level ∈ {2,3}`（H1は作らない）
   - `nodes[].id` は **UUID v4**
   - `nodes[].order` は **親単位で 0..N の連番**
   - `nodes[].parentId` は **H2→null / H3→親H2のUUID**
   - `nodes[].title` は日本語で重複禁止・具体的
   - `targetLength` は `LENGTH_HINTS_JSON` を優先、無い場合は相対配分で推定（任意）
   - ルールJSON（必須/禁止/順序）に厳密準拠
2. 生成結果は **JSONのみ** を標準出力へ返す（末尾カンマ・コメント・前置き/後書き禁止）。

## 出力（契約）

```json
{
  "outlineVersion": "1.0",
  "nodes": [
    {
      "id": "UUIDv4",
      "level": 2,
      "title": "日本語の見出し（H2）",
      "order": 0,
      "parentId": null,
      "targetLength": 600
    },
    {
      "id": "UUIDv4",
      "level": 3,
      "title": "日本語の小見出し（H3）",
      "order": 0,
      "parentId": "親H2のUUIDv4",
      "targetLength": 250
    }
  ]
}
```

## 呼び出し例（バックエンド）

```bash
claude -p "/outline \
  --var STYLE=A \
  --var STRUCTURE_RULES_JSON=/tmp/struct.json \
  --var LENGTH_HINTS_JSON=/tmp/lengths.json \
  --input-file /tmp/sources.md"
```

## 失敗時の扱い（推奨）

- 生成JSONが整合NGのときはバックエンド側で `/repair` を自動実行（TARGET=OUTLINE_JSON）。
