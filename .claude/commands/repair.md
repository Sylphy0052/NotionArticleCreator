# /repair — 成果物修復コマンド
>
> **目的**：壊れたアウトラインJSONまたは本文Markdownを自動修復する。
> **出力**：修正済みの **JSON または Markdown本文のみ**（理由や説明を一切混在させない）。

## 使用サブエージェント

- **RepairAgent**（`.claude/agents/repair.md`）
  - このコマンドの主担当。入力成果物を検証し、ルール違反を自動修復した上で**有効なJSONまたはMarkdown**を返す。
  - 出力は純粋な成果物のみであり、説明・前置き・コードフェンスは禁止。

## 入力（引数仕様）

- `--var TARGET`: `"OUTLINE_JSON" | "MARKDOWN"`
- `--var ERRORS_FILE`: **必須**。検証エラーの要点をまとめた JSON ファイル（バックエンド側で作成）
- `--input-file`: **必須**。壊れた成果物ファイル（JSON または Markdown）

## バリデーション（実行前）

- `TARGET` が `OUTLINE_JSON` または `MARKDOWN` のいずれかであること。
- `ERRORS_FILE` がJSONとして整合していること。
- `--input-file` が非空であること。

## 実行手順（フロー）

1. **RepairAgent** を呼び出し、以下のルールで修復を指示する：
   - **JSON修復**
     - `outlineVersion="1.0"` を保証。
     - `nodes[].level ∈ {2,3}`、`id=UUID v4`、`order` 再採番、`parentId` 整合を担保。
     - 欠損フィールドは補完。孤立/循環は自動修正。
   - **Markdown修復**
     - H1削除、H2/H3 のみ許可。
     - 各見出し行末に `<!-- id: UUID -->` を必須付与。
     - 未閉包フェンスを修復。
     - 空行を1行に統一。
     - 曖昧見出しを具体的な表現に改善。
2. 修復済み成果物を標準出力に返す。

## 出力（契約例）

### TARGET=OUTLINE_JSON

```json
{
  "outlineVersion": "1.0",
  "nodes": [
    {
      "id": "UUIDv4",
      "level": 2,
      "title": "具体的な見出し",
      "order": 0,
      "parentId": null,
      "targetLength": 500
    }
  ]
}
```

### TARGET=MARKDOWN

```markdown
## セクション見出し <!-- id: 11111111-1111-4111-8111-111111111111 -->
本文…
```

## 呼び出し例（バックエンド）

```bash
claude -p "/repair \
  --var TARGET=MARKDOWN \
  --var ERRORS_FILE=/tmp/errors.json \
  --input-file /tmp/broken.md"
```

## 失敗時の扱い（推奨）

- 修復不能（循環除去不可・入力がほぼ空など）の場合は最小限の成果物に縮退して返す。
- 出力が検証で再びNGの場合はエラー通知（E400）を返し、ユーザーに再入力を促す。
