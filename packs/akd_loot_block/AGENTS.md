# プロジェクト構造

このリポジトリは、Minecraft Bedrock Edition（統合版）向けのカスタムブロック系アドオンです。ルート直下にビヘイビアーパックとリソースパックが分かれて配置されています。

```text
.
├── akd_loot_block_bp/                 # ビヘイビアーパック
│   ├── manifest.json                  # BPの基本情報、UUID、対応エンジンバージョン
│   ├── blocks/
│   │   └── loot_surface_block.json    # カスタムブロックの挙動、硬さ、ドロップ表の指定
│   ├── features/
│   │   └── loot_block_surface.json    # 地形生成で配置する単一ブロックFeature
│   ├── feature_rules/
│   │   └── loot_block_surface_rule.json
│   │                                      # Featureの生成条件、頻度、座標を定義
│   └── loot_tables/
│       └── blocks/
│           └── loot_surface_block.json # ブロック破壊時のドロップ内容
├── akd_loot_block_rp/                 # リソースパック
│   ├── manifest.json                  # RPの基本情報、UUID、対応エンジンバージョン
│   ├── blocks/
│   │   └── loot_surface_block.json    # ブロックのジオメトリ、材質、描画・効果音設定
│   └── textures/
│       ├── terrain_texture.json       # テクスチャ短縮名と画像パスの対応表
│       └── blocks/
│           └── loot_surface_block.png # ゲーム内ブロック用テクスチャ
├── textures/                          # 元素材・作業用画像
│   ├── lucky_block.png
│   └── maesuke.png
└── uuid.json                          # BP/RPのheader・module UUIDの控え
```

ビヘイビアーパックとリソースパックはMinecraftのフォルダからジャンクションでリンクされています。
この2つ以外のフォルダは素材など、Minecraftからは参照しないフォルダです。

## 各パックの役割

### `akd_loot_block_bp/`

ゲーム内の挙動を担当します。

- `blocks/loot_surface_block.json` はブロックの識別子、当たり判定、採掘時間、爆発耐性、使用するルートテーブルを定義します。
- `features/loot_block_surface.json` は地形上に置くブロックと、置換可能な地形ブロックを定義します。
- `feature_rules/loot_block_surface_rule.json` はオーバーワールドでの生成タイミング、試行回数、確率、座標を定義します。
- `loot_tables/blocks/loot_surface_block.json` は対象ブロックを壊した際のドロップを定義します。

### `akd_loot_block_rp/`

ゲーム内の見た目を担当します。

- `blocks/loot_surface_block.json` はフルブロック形状と `loot_surface_block` というテクスチャ短縮名を使用します。
- `textures/terrain_texture.json` はテクスチャ短縮名を実画像のパスへ割り当てます。
- `textures/blocks/loot_surface_block.png` が現在配置されているブロック画像です。

### ルート直下の管理・素材ファイル

- `uuid.json` は両パックの `manifest.json` で使う4個のUUIDをまとめています。マニフェストのUUIDを変更する場合は、このファイルとの対応も維持してください。
- `textures/` はパックに直接組み込まれていない元素材・作業用画像の置き場です。ゲームから読み込ませる完成画像は `akd_loot_block_rp/textures/` 以下へ配置します。

## ファイル間の主な参照関係

```text
BP blocks/loot_surface_block.json
└── BP loot_tables/blocks/loot_surface_block.json

BP feature_rules/loot_block_surface_rule.json
└── BP features/loot_block_surface.json

RP blocks/loot_surface_block.json
└── RP textures/terrain_texture.json
    └── RP textures/blocks/*.png

uuid.json
├── BP manifest.json
└── RP manifest.json
```

## 編集時の注意

- JSONファイルの `format_version` と `manifest.json` の `min_engine_version` は、対象とするBedrock Editionの仕様に合わせて揃えてください。
- ブロック、Feature、Feature Ruleの識別子は、参照元と参照先で名前空間を含めて一致させてください。
- 現在、BPのブロック識別子だけが `akd_look_block:loot_surface_block` で、他の定義にある `akd_loot_block:loot_surface_block` と表記が異なります。
- 現在、`terrain_texture.json` は `textures/blocks/loot_surface` を参照していますが、配置済み画像名は `loot_surface_block.png` です。テクスチャを利用する際は参照パスとファイル名を一致させてください。
- BPとRPは独立したパックです。新しいブロックを追加するときは、必要に応じて両方へ対応する定義を追加してください。

## Agent-Specific Instructions

Before modifying any file, present a diff and wait for explicit `OK`.

## Planning Instructions

When the user asks Codex to create a plan, write the plan to a Markdown file under the codex_plans directory.
The file name must follow this format:
YYMMDD_NN.md
Where:

YY is the last two digits of the year.
MM is the two-digit month, zero-padded.
DD is the two-digit day, zero-padded.
NN is a two-digit sequential number, zero-padded.
