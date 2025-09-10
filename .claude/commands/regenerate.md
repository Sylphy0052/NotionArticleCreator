# /regenerate — 部分/全文再生成コマンド
>
> **目的**：既存本文Markdownから、全文または指定範囲（セクション/段落）を再生成する。
> **出力**：Markdown **本文のみ**（説明・前置き・コードフェンスや余計な文字禁止）。

## 使用サブエージェント

- **DraftingAgent**（`.claude/agents/drafting.md`）
  - このコマンドの主担当。対象スコープと追加指示に基づいて部分的な本文を再生成する。
  - 見出しやIDを維持しつつ、差し替え範囲だけを置換可能な形で返す。

## 入力（引数仕様）

- `--var STYLE`: `"A" | "B" | "C"`
- `--var OUTLINE_JSON_FILE`: **必須**。`{"outlineVersion":"1.0","nodes":[...]}` 形式のアウトラインJSON（ファイルパス）
- `--var REGEN_SCOPE`: `"document" | "section" | "paragraph"`
  - `document`: 全文再生成
  - `section`: 指定H2/H3セクションごと再生成
  - `paragraph`: 指定H2/H3配下の段落単位で再生成
- `--var REGEN_TARGET_ID`: 任意。対象ノードのUUID（`REGEN_SCOPE=section|paragraph` 時は必須）
- `--var EXTRA_INSTRUCTIONS`: 任意。追加指示（例：「箇条書きを増やす」）
- `--input-file`: **必須**。既存本文Markdown

## バリデーション（実行前）

- `OUTLINE_JSON_FILE` の存在とJSON整合（`outlineVersion=1.0` で階層/ID妥当）を確認。
- `REGEN_SCOPE` が `document|section|paragraph` のいずれかであること。
- `REGEN_TARGET_ID` が必要時に必ず指定されていること。
- 入力Markdownが非空であること。

## 実行手順（フロー）

1. **DraftingAgent** を呼び出し、以下を指示する：
   - `REGEN_SCOPE=document` の場合：全本文を生成し直す。
   - `REGEN_SCOPE=section` の場合：指定UUIDの見出し〜次の同階層直前までを置換対象として生成。
   - `REGEN_SCOPE=paragraph` の場合：指定UUID配下の段落のみを再生成。
   - 見出しと `<!-- id: UUID -->` は必ず維持。
   - `EXTRA_INSTRUCTIONS` があれば必ず反映。
   - 出力は **Markdown本文のみ**。余計な文字禁止。
2. 生成結果を標準出力に返す。

## 出力（契約例）

```markdown
## セクション見出し <!-- id: 11111111-1111-4111-8111-111111111111 -->
新しい本文…
```

- 見出し行＋idは維持。
- 差し替え範囲のみ更新された状態のMarkdown本文。

## 呼び出し例（バックエンド）

```bash
# セクション再生成の例
claude -p "/regenerate \
  --var STYLE=A \
  --var OUTLINE_JSON_FILE=/tmp/outline.json \
  --var REGEN_SCOPE=section \
  --var REGEN_TARGET_ID=11111111-1111-4111-8111-111111111111 \
  --var EXTRA_INSTRUCTIONS='事例を追加' \
  --input-file /tmp/current.md"
```

## 失敗時の扱い（推奨）

- Markdown検証（H1混入、id欠落/不一致、未閉包フェンス）でNGの場合、バックエンド側で `/repair` を自動実行（`TARGET=MARKDOWN`）。
- 修復不能時はエラー通知（E301）として扱い、ユーザーに再入力を促す。
